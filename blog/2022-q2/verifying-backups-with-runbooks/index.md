---
title: Verifying backups with Runbooks
description: Learn how to automate the process of verifying your backups using a custom runbook
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

It is no longer a question these days of whether or not you should have backups. Regular backup procedures are simply a fact of life for any modern organization that recognizes the value of their data, with many organizations even going as far as transferring their backup media to another site to ensure a physical disaster in one location doesn't destroy their data.

Less common though is a robust backup verification process. The finer points of the backup restoration process are often only worked out when disaster strikes. It is also during a disaster that organizations find themselves wondering if they have backed up the right data in the right format. Needless to say these are not the questions DevOps personnel want to be faced with when trying to restore operations in the midst of a crisis.

Runbooks provide a convenient solution to these questions. By automating and regularly executing the process of restoring backups in a disposable environment, DevOps teams can gain confidence that their backups are valid and the process of restoring a system is well rehearsed.

Some years ago we released a [blog series documenting the process of building a CI/CD and operations pipeline for a Java application](https://octopus.com/blog/java-ci-cd-co/from-jar-to-docker). This series wrapped up with a runbook designed to backup a MySQL database. While valuable advice, this post series made the same mistake of assuming creating a backup and saving it offsite was the end of the story.

In this post you'll learn how to complete the backup cycle by verifying the backup against a real, if disposable, MySQL database in a custom runbook.

## Treating backups as deployable artifacts

The database describe in the [previous post](https://octopus.com/blog/java-ci-cd-co/from-cd-to-co) was deployed to a Kubernetes cluster. As was noted in the previous post, the process of performing the backup involves running the `mysql` client within the MySQL container, dumping the database tables, and uploading the resulting file to a more permanent location. That location was an S3 bucket.

However, verifying a backup file means treating it much like any other deployable artifact. The artifacts used to deploy your applications are versioned and saved in repositories that allow artifacts versions to be queried and compared. Conceptually, your database backup should be just another deployable and versioned artifact, ready to be queried and consumed by a runbook.

In practice this means the backup artifact needs to be uploaded to a repository rather than a simple file store. This is not to say that you can no longer use a service like S3, as you can quite easily [format S3 to function as a Maven repository](https://octopus.com/blog/hosting-maven-in-s3). However, for the purposes of this blog, you'll upload the backup file directly to the Octopus built-in feed.

Upload files to Octopus is most easily done with the Octopus CLI. The `Dockerfile` below installs the Octopus and AWS CLIs alongside the MySQL server:

```Dockerfile
FROM mysql
RUN apt-get update; apt-get install python python-pip -y
RUN pip install awscli
RUN apt update && apt install --no-install-recommends gnupg curl ca-certificates apt-transport-https -y && \
curl -sSfL https://apt.octopus.com/public.key | apt-key add - && \
sh -c "echo deb https://apt.octopus.com/ stable main > /etc/apt/sources.list.d/octopus.com.list" && \
apt update && apt install octopuscli -y
```

## Backing up the database

The following PowerShell script locates the first pod whose name starts with "mysql", executes `mysqldump` to perform a backup, packages the SQL file as a artifact with the Octopus CLI, and pushes the artifact to the Octopus server:

```PowerShell
# Get the list of pods in JSON format
kubectl get pods -o json |
# Convert the output to an object
ConvertFrom-Json |
# Get the items property
Select -ExpandProperty items |
# Limit the items to those with the name starting with "mysql"
? {$_.metadata.name -like "mysql*"} |
# We only expect to find 1 such pod
Select -First 1 |
# Execute mysqldump on the pod to perform a backup
% {
    Write-Host "Performing backup on pod $($_.metadata.name)"
    $backupVersion = Get-Date -Format "yyyy.MM.dd"
    kubectl exec $_.metadata.name -- /bin/sh -c 'cd /tmp; mysqldump -u root -p#{MySQL Password} petclinic > dump.sql 2> /dev/null'
    kubectl exec $_.metadata.name -- /bin/sh -c "cd /tmp; octo pack --overwrite --include dump.sql --id PetClinicDB --version $($backupVersion) --format zip"
    kubectl exec $_.metadata.name -- /bin/sh -c "cd /tmp; octo push --package PetClinicDB.$($backupVersion).zip --overwrite-mode OverwriteExisting --server https://tenpillars.octopus.app --apiKey #{Octopus API Key} --space #{Octopus.Space.Name}"
}
```

This is the equivalent in bash:

```bash
POD=$(kubectl get pods -o json | jq -r '[.items[]|select(.metadata.name | startswith("mysql"))][0].metadata.name')
VERSION=$(date +"%Y.%m.%d")
kubectl exec $POD -- /bin/sh -c 'cd /tmp; mysqldump -u root -p#{MySQL Password} petclinic > dump.sql 2> /dev/null'
kubectl exec $POD -- /bin/sh -c "cd /tmp; octo pack --overwrite --include dump.sql --id PetClinicDB --version ${VERSION} --format zip"
kubectl exec $POD -- /bin/sh -c "cd /tmp; octo push --package PetClinicDB.${VERSION}.zip --overwrite-mode OverwriteExisting --server https://tenpillars.octopus.app --apiKey #{Octopus API Key} --space #{Octopus.Space.Name}"
```
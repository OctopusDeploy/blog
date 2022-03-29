---
title: Automating support emails with Octopus Runbooks
description: You can use Octopus Runbooks to automate delivery of important information to your support teams when things go wrong. This post explains how. 
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-placeholder.png
bannerImage: blogimage-placeholder.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Runbooks Series
  - Runbooks
---

You can automate lots of useful tasks with Octopus Runbooks, especially around the management of infrastructure. One of the subtler but no less useful things, however, is  Octopus can let people know when there are problems.

There are a couple of ways to approach this, depending on the information needed and what's useful to your support teams.

Let's look at some examples.

## Setting an SMTP connection in Octopus

You must enter SMTP settings in Octopus before runbook steps can send emails to support teams. If you don't know your SMTP server or port details, speak to your operations team.

1. Click **Configuration** from the top menu.
1. Click **SMTP** from the left menu.
1. Complete the following fields and click **SAVE AND TEST**:
   - **SMTP Host**
   - **SMTP Port**
   - **Use SSL/TLS** - forces secure SSL and TLS protocols. If disabled, Octopus will still use this option if your server supports either protocol.
   - **From Address** - the address you'd like Octopus to send from.
   - **Credentials** - the username and password for the email account.

If the test fails, check your details and try again. Some email services, such as Gmail, may need settings changed to allow external apps to send emails.

## Simple support emails

If your step or script is simple enough that a problem should be easy to solve, a simple email saying 'hey, something's up!' might be enough. In this case, creating an automated support email step is very easy.

In our sample instance, we created a runbook that runs a doomed-to-fail 'Hello World!' script. On failure, the runbook triggers an email to a support address.

Adding a step like this is a great way to bring outdated and broken runbooks to the attention of those that can fix them.

Here's how we set up the **Send an Email** step:

1. Open the project you'd like to trigger a support email for.
1. Click **Operations**  
1. Click **Runbooks**
1. Click an existing runbook to edit it, or create a new one with the **ADD RUNBOOK** button. If creating a new runbook, enter a name and description, then click **SAVE**.
1. Click **ADD STEP**.
1. Search for `email`, hover over **Send an Email** from the results and click **ADD**.
1. Complete the following fields and click **SAVE**:
   - **Step Name** - give the step a descriptive name.
   - **To**, **CC**, and **BCC** - enter an email address or select a team from the dropdown. See our documentation for more information about [creating teams](https://octopus.com/docs/security/users-and-teams).
   - **Subject**
   - **Body** - use raw text or HTML emails
   - **Run Condition** - if alerting someone to a failed deployment or runbook step, select **Failure: only run when previous step fails**
   - **Start Trigger** - usually best to leave as **Wait for the previous step to complete, then start**.

When creating or editing a runbook, you must click **PUBLISH** to make it available to other teams and Octopus trigger events.

:::hint
New steps will always appear at the bottom of your runbook process. If you need to reorder your steps:

1. Open your runbook.
1. Go to the **Process** tab.
1. Click any step to open the process editor.
1. Click the menu button (3 vertical dots) next to the **Filter by name** search box and select **Reorder Steps**.
1. Drag your steps into the order you need and click **DONE**.
:::

## Advanced support emails

A simple support email is fine for things that are easy to fix, but what if your support teams need a little more info?

With [output variables](https://octopus.com/docs/projects/variables/output-variables) and a little effort, the **Send an Email** step can also include everything needed to start troubleshooting.

In this example, our sample runbook scrapes a Kubernetes cluster for information to send in an email to support, including:

- Deployment information
- Pod logs
- Description of the deployment targets

This type of runbook is perfect for

- Automatically pulling technical info from your deployment targets
- Helping less technical staff get information without knowing tricky terms or commands
- Speeding up system recovery

### Before you start

This sample uses:

- Kubernetes as a deployment target, though you can tailor the idea for other target types.
- PowerShell scripts to scrape data from Kubernetes. You may need to install PowerShell on your Linux distribution for these scripts to work. See [Microsoft's PowerShell install documentation](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.2) for how. 

### Create the project runbook

1. Open the project you'd like to trigger a support email for.
1. Click **Operations**.
1. Click **Runbooks**.
1. Click **ADD RUNBOOK**.
1. Enter a name and description and click **SAVE**.

### Create Kubernetes steps

Now we can create the steps needed to scrape information from your Kubernetes cluster. The steps use output variables so we can add the information to an email.

Use the **ADD STEP** button to create 3 **Kubectl CLI Script** steps with the following names and Powershell scripts.

:::hint  
To create the steps:

1. While in the new runbook, click the **Process** tab and then **ADD STEP**.
1. Click **Kubernetes**, then **Run a kubectl CLI Script** under **Installed Step Templates**.
1. Complete the following fields and click **SAVE**:
   - **Step Name** - get the step name from the step information below.
   - **On Behalf Of** - select your target role from the dropdown.
   - **Worker Pool** - check **Runs on a worker from a specific work pool** and select **Hosted Ubuntu** from the dropdown. (only needed if using Octopus workers)
   - **Container Image** - check **Runs inside a container, on a worker**, then click **Use latest Linux-based image** to automatically complete the fields. (only needed if using Octopus workers)
   - **Inline Source Code** - check **Powershell** and enter the code for each step below.
:::

#### Step 1: Get deployment

Call this step `Get deployment`. We reference this name when setting up the support email step.

Enter the following code in the **Inline Source Code** field. Swap **yournamespace** for your own cluster's namespace.

```
ps
$output = (kubectl describe deployment -n) -join "`n"
Set-OctopusVariable -name "Describe[$($_.metadata.name)].Content" -value $output

exit 0
```

#### Step 2: Get pod logs

Call this step `Get pod logs`. We reference this name when setting up the support email step.

Enter the following code in the **Inline Source Code** field and swap **deployment-name** your own Kubernete's deployment name.

```
ps
$pods = kubectl get pods -o json | ConvertFrom-Json
$items = if ($pods.items -ne $null) {$pods.items} else {@($pods)}

$items | 
	? {$_.metadata.name -like "deployment-name*"} |
    % {
    	$logs = (kubectl logs $_.metadata.name) -join "`n"
        Set-OctopusVariable -name "Logs[$($_.metadata.name)].Content" -value $logs
    }

exit 0
```

#### Step 3: Get description

Call this step `Get description`. We reference this name when setting up the support email step.

Enter the following code in the **Inline Source Code** field and swap **deployment-name** your own Kubernete's deployment name.

```
ps
$pods = kubectl get pods -o json | ConvertFrom-Json
$items = if ($pods.items -ne $null) {$pods.items} else {@($pods)}

$items | 
	? {$_.metadata.name -like "deployment-name*"} |
    % {
    	$logs = (kubectl describe pod $_.metadata.name) -join "`n"
        Set-OctopusVariable -name "Describe[$($_.metadata.name)].Content" -value $logs
    }

exit 0
```

### Create the email step

Now we can add the **Send an Email** step and configure it to include the scraped information.

1. Click **ADD STEP**.
1. Search for `email`, hover over **Send an Email** from the results and click **ADD**.
1. Complete the following fields and click **SAVE**:
   - **Step Name** - give the step a descriptive name.
   - **To**, **CC**, and **BCC** - enter an email address or select a team from the dropdown. See our documentation for more information about [creating teams](https://octopus.com/docs/security/users-and-teams).
   - **Subject**
   - **Body** - use the following raw text:

```
Issue context:

The description of the deployment is included below:

#{each deployment in Octopus.Action[Get deployment].Output.Describe}
Description of #{deployment}
#{deployment.Content}

-----------------------------------------------------------

#{/each}

The pod logs are included below:

#{each pod in Octopus.Action[Get pod logs].Output.Logs}
Logs for pod #{pod}
#{pod.Content}

-----------------------------------------------------------

#{/each}

The pod descriptions are included below:

#{each pod in Octopus.Action[Get description].Output.Describe}
Description of #{pod}
#{pod.Content}

-----------------------------------------------------------

#{/each}
 ```

### Reviewing the support email

Looking through the email triggered by our example here, we can spot a couple of problems with our Kubernetes cluster:

- The port number is not a number
- The pod has an undefined URL

This means the support team quickly have an idea as to the problem with the deployment target.

```
The description of the deployment is included below:


Description of
Name:                   random-quotes
Namespace:              randomquotes
CreationTimestamp:      Mon, 21 Mar 2022 03:34:15 +0000
Labels:                 Octopus.Action.Id=ca042dbb-ea71-4a77-b9ae-edf94acc441e
                        Octopus.Deployment.Id=deployments-21
                        Octopus.Deployment.Tenant.Id=untenanted
                        Octopus.Environment.Id=environments-1
                        Octopus.Kubernetes.DeploymentName=random-quotes
                        Octopus.Kubernetes.SelectionStrategyVersion=SelectionStrategyVersion2
                        Octopus.Project.Id=projects-1
                        Octopus.RunbookRun.Id=
                        Octopus.Step.Id=1968cd33-5ab9-4e47-8d9e-0f87f5c780c2
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               Octopus.Kubernetes.DeploymentName=random-quotes
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  Octopus.Action.Id=ca042dbb-ea71-4a77-b9ae-edf94acc441e
           Octopus.Deployment.Id=deployments-21
           Octopus.Deployment.Tenant.Id=untenanted
           Octopus.Environment.Id=environments-1
           Octopus.Kubernetes.DeploymentName=random-quotes
           Octopus.Kubernetes.SelectionStrategyVersion=SelectionStrategyVersion2
           Octopus.Project.Id=projects-1
           Octopus.RunbookRun.Id=
           Octopus.Step.Id=1968cd33-5ab9-4e47-8d9e-0f87f5c780c2
  Containers:
   octopussamples:
    Image:      index.docker.io/octopussamples/randomquotesnodejs:1.0.2
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      PORT:  notanumber
    Mounts:  <none>
  Volumes:   <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   random-quotes-758866bcd8 (1/1 replicas created)
Events:          <none>

-----------------------------------------------------------

The pod logs are included below:

Logs for pod random-quotes-758866bcd8-dk2x9
App listening at **http://undefined:undefined**

-----------------------------------------------------------

The pod descriptions are included below:

Description of random-quotes-758866bcd8-dk2x9
Name:         random-quotes-758866bcd8-dk2x9
Namespace:    randomquotes
Priority:     0
Node:         aks-agentpool-10984672-vmss000001/10.240.0.5
Start Time:   Mon, 21 Mar 2022 03:34:15 +0000
Labels:       Octopus.Action.Id=ca042dbb-ea71-4a77-b9ae-edf94acc441e
              Octopus.Deployment.Id=deployments-21
              Octopus.Deployment.Tenant.Id=untenanted
              Octopus.Environment.Id=environments-1
              Octopus.Kubernetes.DeploymentName=random-quotes
              Octopus.Kubernetes.SelectionStrategyVersion=SelectionStrategyVersion2
              Octopus.Project.Id=projects-1
              Octopus.RunbookRun.Id=
              Octopus.Step.Id=1968cd33-5ab9-4e47-8d9e-0f87f5c780c2
              pod-template-hash=758866bcd8
Annotations:  <none>
Status:       Running
IP:           10.244.0.10
IPs:
  IP:           10.244.0.10
Controlled By:  ReplicaSet/random-quotes-758866bcd8
Containers:
  octopussamples:
    Container ID:   containerd://04b1c9a199d52dd6e72c8677ecc5ab7eb57e85772d05c0733608c2d4e694bca0
    Image:          index.docker.io/octopussamples/randomquotesnodejs:1.0.2
    Image ID:       docker.io/octopussamples/randomquotesnodejs@sha256:b2104dd603ef648f92cdd605ecf814e4cb24a2e1827f036b004985d06a380644
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 21 Mar 2022 03:34:37 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      PORT:  **notanumber**
    Mounts: /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jss2f (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-jss2f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute for 300s
                             node.kubernetes.io/unreachable:NoExecute for 300s
Events:                      <none>
```

## Conclusion

These examples are just a small insight into how Octopus Runbooks can help you get key information to those that need it.

If you'd like to try Octopus Runbooks for yourself, [sign up for a free trial](https://octopus.com/start) and have fun experimenting!

Happy deployments!
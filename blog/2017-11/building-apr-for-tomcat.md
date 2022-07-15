---
title: Building the Apache Portable Runtime (APR)
description: Depending on your Linux distro, you may have to build the APR from scratch to take advantage of the higher performance of the OpenSSL library in Tomcat.
author: matthew.casperson@octopus.com
visibility: public
published: 2017-10-31
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

The [Apache Portable Runtime (APR)](https://tomcat.apache.org/tomcat-9.0-doc/apr.html) is used by Tomcat to provide a number of enhanced features and performance. For example, the APR needs to be present in order to get the increased performance provided by OpenSSL for HTTPS.

Tomcat provides a precompiled copy of the APR with for Windows users, but Linux users are often directed to install the APR from their OS distribution package repositories.

These are the APR and Tomcat Native packages available from the [EPEL repo](https://fedoraproject.org/wiki/EPEL) for Centos 7:

```
$ yum info apr
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: centos.mirror.ausnetservers.net.au
 * epel: fedora.melbourneitmirror.net
 * extras: mirror.nsw.coloau.com.au
 * updates: mirror.nsw.coloau.com.au
Installed Packages
Name        : apr
Arch        : x86_64
Version     : 1.4.8
Release     : 3.el7
Size        : 221 k
Repo        : installed
From repo   : anaconda
Summary     : Apache Portable Runtime library
URL         : http://apr.apache.org/
License     : ASL 2.0 and BSD with advertising and ISC and BSD
Description : The mission of the Apache Portable Runtime (APR) is to provide a
            : free library of C data structures and routines, forming a system
            : portability layer to as many operating systems as possible,
            : including Unices, MS Win32, BeOS and OS/2.

Available Packages
Name        : apr
Arch        : i686
Version     : 1.4.8
Release     : 3.el7
Size        : 107 k
Repo        : base/7/x86_64
Summary     : Apache Portable Runtime library
URL         : http://apr.apache.org/
License     : ASL 2.0 and BSD with advertising and ISC and BSD
Description : The mission of the Apache Portable Runtime (APR) is to provide a
            : free library of C data structures and routines, forming a system
            : portability layer to as many operating systems as possible,
            : including Unices, MS Win32, BeOS and OS/2.

$ yum info tomcat-native
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos.mirror.ausnetservers.net.au
 * epel: fedora.melbourneitmirror.net
 * extras: mirror.nsw.coloau.com.au
 * updates: mirror.nsw.coloau.com.au
Installed Packages
Name        : tomcat-native
Arch        : x86_64
Version     : 1.1.34
Release     : 1.el7
Size        : 172 k
Repo        : installed
From repo   : epel
Summary     : Tomcat native library
URL         : http://tomcat.apache.org/tomcat-8.0-doc/apr.html
License     : ASL 2.0
Description : Tomcat can use the Apache Portable Runtime to provide superior
            : scalability, performance, and better integration with native server
            : technologies.  The Apache Portable Runtime is a highly portable library
            : that is at the heart of Apache HTTP Server 2.x.  APR has many uses,
            : including access to advanced IO functionality (such as sendfile, epoll
            : and OpenSSL), OS level functionality (random number generation, system
            : status, etc), and native process handling (shared memory, NT pipes and
            : Unix sockets).  This package contains the Tomcat native library which
            : provides support for using APR in Tomcat.
```

So we have access to packages providing APR version 1.4.8 and Tomcat Native version 1.1.34. But with these packages installed, Tomcat 9.01 reports the following error:

```
org.apache.catalina.core.AprLifecycleListener.init An incompatible version [1.1.34] of the APR based Apache Tomcat Native library is installed, while Tomcat requires version [1.2.14]
```

This means the version we can install from the Centos package manager is not up to date enough for later versions of Tomcat.

The solution is to compile these libraries for yourself. This sounds complicated, but is actually quite easy.

First, you will need to install the development tools and the Java Development Kit. In Centos, this is done with the commands:

```
sudo yum groupinstall "Development Tools"
sudo yum install java-1.8.0-openjdk-devel
```

Next you run the following script script, which will download, extract, compile and install the various libraries required by Tomcat.

:::hint
The URLs in this script point to mirrors for the [https://tomcat.apache.org/download-native.cgi](Tomcat Native) library and the [APR](https://apr.apache.org/download.cgi) library. You can change these URLs to point to a closer server, or a different server if the ones listed here are down.
:::

```
#!/usr/bin/env bash
cd /tmp
wget http://apache.mirror.amaze.com.au//apr/apr-1.6.3.tar.gz
tar -zxvf apr-1.6.3.tar.gz
cd apr-1.6.3
./configure
make
make install

cd /tmp
wget http://apache.mirror.amaze.com.au//apr/apr-util-1.6.1.tar.gz
tar -zxvf apr-util-1.6.1.tar.gz
cd apr-util-1.6.1
./configure --with-apr=/usr/local/apr
make
make install

cd /tmp
wget http://apache.melbourneitmirror.net/tomcat/tomcat-connectors/native/1.2.14/source/tomcat-native-1.2.14-src.tar.gz
tar -zxvf tomcat-native-1.2.14-src.tar.gz
cd tomcat-native-1.2.14-src/native
./configure --with-apr=/usr/local/apr --with-java-home=/usr/lib/jvm/java-1.8.0-openjdk
make
make install
```

Finally, create a file called `bin/setenv.sh` under the Tomcat directory with the following content. This allows Tomcat to find the libraries that were compiled by the script above.

```
CATALINA_OPTS="-Djava.library.path=/usr/local/apr/lib"
```

With these changes in place, you should see the following message in the `logs/catalina.out` file. This indicates that Tomcat successfully loaded the APR library.

```
31-Oct-2017 18:21:08.930 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded APR based Apache Tomcat Native library [1.2.14] using APR version [1.6.3].
```

If you are interested in automating the deployment of your Java applications to Tomcat, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).

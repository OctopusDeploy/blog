---
title: Checking Kubernetes pod CPU and memory
description: Learn how to check a pod's resource usage in Kubernetes
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: blogimage-gettingstartedwithdockeralpine2-2022.png
bannerImage: blogimage-gettingstartedwithdockeralpine2-2022.png
bannerImageAlt: Man standing with a laptop in front of a large blue container
tags:
 - Octopus
---

Tracking the resource usage of local processes is relatively easy in Linux with the `top` or `htop` command. But how do you track resource usage of pods spread across a Kubernetes cluster?

In this post I'll show you how to view the CPU and memory usage of pods in Kubernetes.

## The metrics server

The [metrics server](https://github.com/kubernetes-sigs/metrics-server) provides Kubernetes clusters with a lightweight and highly scalable solution for collecting CPU and memory resources. Although the metrics server is not built into Kubernetes, most clusters either bundle it or provide an easy solution for enabling it.

Once the metrics service is installed, pod resources are displayed with the command:

```
kubectl top pod
```

The node resource usage is available with the command:

```
kubectl top node
```

The following error indicates that the metrics server is not installed:

```
error: Metrics API not available
```

In this case, you can install the metrics server with the instructions [here](https://github.com/kubernetes-sigs/metrics-server).

## cgroup resource usage

If the metrics service is not available, it is possible to determine the memory usage of a single pod by entering an interactive session and printing the contents of cgroup interface files.

Enter an interactive session with the following command, replacing `podname` with the name of the pod you wish to inspect:

```
kubectl exec -it podname -- sh
```

Print the current memory usage with the command:

```
cat /sys/fs/cgroup/memory/memory.usage_in_bytes
```

Print the current cpu usage with the command:

```
cat /sys/fs/cgroup/cpu/cpuacct.usage
```

The value returned by `cpuacct.usage` is not immediately useful as [it returns](https://www.kernel.org/doc/Documentation/cgroup-v1/cpuacct.txt):

> the CPU time (in nanoseconds) obtained by this group

Converting this value into a more usable measurement like CPU usage percentage requires some calculation. This [post on Stack Exchange](https://unix.stackexchange.com/a/451005) provides more details.

You can find more information on these files [here](https://www.kernel.org/doc/Documentation/cgroup-v1/00-INDEX).

## cgroup2 resource usage

If the directories `/sys/fs/cgroup/memory` or `/sys/fs/cgroup/cpu` do not exist, you are likely working on a system with cgroups v2.

Print the current memory usage with the command:

```
cat /sys/fs/cgroup/memory.current
```

Print the current cpu usage with the command:

```
cat /sys/fs/cgroup/cpu.stat
```

This will return a file with a value called `usage_usec`. As with value returned by the cgroup v1 `cpuacct.usage` file, this value must be converted into a CPU usage percentage to be useful. Note the `usage_usec` value is measured in milliseconds, unlike the value returned by the `cpuacct.usage` file, which is in nanoseconds. 

You can find more information on these files [here](https://www.kernel.org/doc/Documentation/cgroup-v2.txt).

## Conclusion

The metrics server provides a convenient method for inspecting the CPU and memory resources of your Kubernetes pods and nodes. It is also possible to find these values manually by inspecting the cgroup interface files, although some manual calculations are required to determine CPU usage as a percentage.

Happy deployments!
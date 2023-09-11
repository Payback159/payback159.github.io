---
title: 'Resource Management in Kubernetes'
date: 2023-11-11T23:36:00+00:00
draft: false
tags:
  - kubernetes
  - resources
  - management
---

| ![Photo by Growtika on UnsplashForeword](https://cdn-images-1.medium.com/max/1600/0*mye2tGZdVkKTFr1c) |
|:--:|
| *Photo by Growtika on Unsplash* |

## Foreword

In the following chapter we will discuss the topics CPU requests, CPU limits, memory requests and memory limits. We will try to go into details here only so far as to build an understanding of the foundation of Kubernetes Resources Management. For those who are interested in the topic and want to learn more, I recommend the official documentation <https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/>.

## General

In general, the 4 values of CPU Requests, CPU Limits, Memory Requests and Memory Limits can be viewed in 2 phases.

### 1. Phase: Scheduling

When you want to roll out a pod, whether managed by a manager (e.g.: Deployment, StatefulSet) or not on Kubernetes, the installation goes through a scheduling phase. During scheduling, Kubernetes takes into account the values of CPU requests and memory requests you specify.
Specifically, if you have a pod with CPU request of 1 ( or 1000m), Kubernetes will only consider nodes that have at least one CPU free. Mathematically, these are all nodes where

```
sum(cpu-requests of all scheduled pods on this node)
 - (Total CPU count of this node) >= 1000m (1 Core)
```

Of course, a similar calculation applies to memory requests, if one assumes that the deployment has a memory request of 1Gi.

```
sum(memory-requests of all scheduled pods on this node)
 - (Total Memory count of this node) >= 1Gi
```

For the sake of completeness, it should be mentioned here that the values CPU Limits and Memory Limits are not taken into account in this phase.

### 2. Phase: Runtime

We come to the 2nd phase. Kubernetes was able to successfully deploy your pod on a node. From now on, the CPU limits and memory limits come into play. Depending on which of these limits your pod reaches, Kubernetes reacts differently hard. In the case of CPU limits, the requested CPU (time) will be withdrawn from your pod and a throttling occurs. However, this may or may not have a negative impact on your application from an end user perspective.

In the case of memory limits, Kubernetes uses the so-called Out of Memory Manager. The [Out of Memory Manager](https://www.kernel.org/doc/gorman/html/understand/understand016.html) is a well-maintained component from the Linux area and is intended to protect the system from processes that try to use more memory than is available / allowed. Should your pod exceed the memory limits, the process and with it the container will be automatically killed in order not to consume even more memory.

**Summary**: If you reach the CPU limits, your pod will be throttled but will continue to run. If you reach the memory limits, your pod will be killed and restarted.

### CPU-Requests

In the past, I had very good experience when I set the CPU requests in my deployments to the average CPU consumption.

### CPU-Limits

As already described, your application is throttled on the CPU side if the actual consumption exceeds the CPU limits. If possible, I set my CPU limits to the highest CPU consumption I know of. Of course it has to be taken into account that I could exceed the limits of the node below.

### Memory-Requests

At this time, I personally would not recommend over-provisioning memory in a production environment unless you are quite sure that your application(s) can handle unexpected out of memory kills and or failures. That is why I am using Kubernetes [LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/) to enforce that the ratio between memory requests and memory limits is always 1.
This way I can ensure that the requests and limits always get the same value and thus no over-provisioning takes place. You can achieve this by only defining memory limits in your deployment or by equating these values.

### Memory-Limits

Although I already talked about setting the limits to the same value in the Memory Requests section, I don't want to withhold this information. With the relation between memory limits and memory requests I can define the over-provisioning behavior of your application regarding the memory. Again, any additional bytes above the memory requests will not receive any guarantees from Kubernetes and thus this memory can be "reclaimed" at any time. In the past, I had painful experiences with applications where the operating system could not recognize the actual memory consumption. (Yes! I'm looking at you Java Virtual Machine 😒)

Nonetheless, you should make your own experiences with these values and experiment around with them. If you have another method to set or find your values, feel free to contact me. I am always interested in how others make their experience.

## Summary

In summary, it means that with the request values of CPU and memory we can on the one hand influence the scheduling behavior of Kubernetes and on the other hand also get a certain form of guarantee that we will get these resources provided. In the case of limits it is always a bit of a game, as long as the resources are not needed by other applications on the node they can be used but they will never be 100% owned by your applications.
Keep this in mind, especially in Kubernetes clusters working with multiple clients this can also affect the stability of other deployments. In future articles I will go into more detail on how Kubernetes and the Linux kernel behaves when CPU and/or memory starvation actually occurs on node level.
Until then, have fun experimenting and stay curious.
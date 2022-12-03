---
title: 'The journey to a self-healing platform'
date: 2022-11-08T21:47:41+00:00
draft: false
tags:
  - kubernetes
  - self-healing
  - cloud native
  - openfero
---

![](https://cdn-images-1.medium.com/max/1600/0*BttpClgrW2Yrq-Bv)
Photo by Austin Kehmeier on Unsplash

Hi guys, my first story is about my hobby project "OpenFero".

>Open Fero is a little play on words from the Latin "opem fero", which means "to help" and the term "OpenSource". Hence the name "openfero". The scope of OpenFero is a framework for self-healing in a cloud-native environment.

## Beginning of the journey

In my ongoing time as a Kubernetes Platform Engineer, I've always wondered, "Isn't there a solution to take the self-healing principle of Kubernetes further?" Kubernetes itself already offers some self-healing features but often only for the application and services running on top of it.

The term self-healing becomes more difficult when it comes to self-healing functions for the cluster itself and even though this topic is getting more and more attention in the community and solutions are being developed, in my view it still depends heavily on the technology stack in the company whether these can be easily integrated.

Apart from that, many of the services in the cluster depend on metrics coming from the service itself or its dependencies. (Databases, Kafka, Redis)

The more I dealt with it, the more I wanted to find a solution that would not only allow engineers and developers to provide self-healing jobs, but would also be flexible enough to allow the user to take the implementation of the self-healing process in the language they usual use.

After some time and from my point of view unsuccessful search for exactly such a tool, I made the decision to write exactly this solution myself. (Hints for known tools are always welcome)

## The idea of OpenFero was born

… The rough requirements for the solution are.

* To follow Cloud Native principle
* Easy installation variant, ideally on the cluster itself or at least use the already existing features of Kubernetes.
* The tool should provide a framework but not restrict the user in the choice of tooling for the implementation.

Of course there are still a few questions open:

* Where does OpenFero actually get the information that there is something broken?
* How to create an environment that can handle this heterogeneous landscape of jobs that can potentially arise?

At the beginning of our journey into the Kubernetes and Cloud Native world, we quickly realized that without a monitoring solution adapted to this world, we won't get far. And, what goes better with a Kubernetes cluster than a well-maintained Prometheus stack. 😉

With that, I quickly had the answer to my first question. OpenFero itself does not take on the task of checking the health and metrics of the services or the cluster, but provides a webhook endpoint as an interface that should meet the specifications for the Prometheus webhook receiver.

With this I could not only use in this specific case the accumulated know-how of our teams, which has been collected over the years in the form of Prometheus Alert Rules, but I also claim to have created a certain form of compatibility for others in this world.

That left only the question: **How to create an environment that can handle this heterogeneous landscape of jobs that can potentially arise?** and the quick answer "Kubernetes Jobs and ConfigMaps".

As of today, jobs are simply stored in ConfigMaps in Kubernetes, as shown in a small example below, and interpreted by OpenFero in the event of an alert and executed via Kubernetes. (As I write here, I'm wondering if an operator pattern might even make sense for this, but I'll save that for another story.)

```yaml
apiVersion: v1
data:
  KubeQuotaFullyUsed: |-
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: openfero-kubequotafullyused-firing
      labels:
        app: openfero
    spec:
      parallelism: 1
      completions: 1
      template:
          spec:
            containers:
            - name: python-job
              image: ubuntu:latest
              args:
              - bash
              - -c
              - |-
                echo "Hallo Welt"
              imagePullPolicy: Always
            restartPolicy: Never
kind: ConfigMap
metadata:
  labels:
    app: openfero
  name: openfero-kubequotafullyused-firing
  namespace: openfero
```

## So how is OpenFero structured?

OpenFero consists logically of 2 components. The first is a Golang implemented endpoint for Alertmanager webhooks.

If an alert is now sent from the alert manager to this endpoint, OpenFero checks whether a job in the form of a ConfigMap exists for this alert and tries to execute the Kubernetes job defined therein.

In the world of OpenFero, these jobs are not simply called jobs, otherwise that would be too simple 😁. We speak of "Operarios", which also comes from Latin and translates as "worker"💪.

Since the actual hard work for the self-healing process is realized in this part, I thought the name was appropriate.

But what is such an operarios and how does all this enable me to make OpenFero a code-agnostic solution?
To put it simply, the only requirements that OpenFero currently places on an Operarios is that it provides a runtime environment for the desired language in the form of a container and that it can receive and process the information from the alert injected by OpenFero via environment variables.

## It's not the end of the journey

The OpenFero journey has only just begun and many questions known and unknown will hopefully be answered on this journey to the full-featured solution for deploying a self-healing platform.

It's currently just a project I'm working on, but if you're interested you're more than welcome to be a part of this journey.

I'm grateful for every contribution, be it code, discussion, feedback, documentation or a great logo to give the project more personality. If you just like the project, I would be happy about a star too! 😊

If I have sparked your interest, you are welcome to contact me or simply leave a comment in the [project on github](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2FPayback159%2Fopenfero).
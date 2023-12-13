---
title: 'When a self-service portal is not enough'
date: 2023-12-13T21:15:00+00:00
draft: false
Tags:
  - kubernetes
  - self-service
  - cloud native
  - tenama
---

| ![White clock on a white wall. The clock contains the word temporary](https://cdn-images-1.medium.com/max/1600/0*nxgW6jrjr0bQBMMu) |
|:--:|
| *Photo by Emanuel Ekström on Unsplash* |

## temporary namespace manager (tenama)

Today's article is about another service that I was able to develop and open-source during my work as a platform engineer. With the following anecdote, I would also like to lay the foundation for my current "story" of why we developed tenama.

## What happenend so far

In the environment I work in, we build our on-premise Kubernetes clusters very heavily based on open source products. In recent years, it has been important to us to source our components as directly as possible from the open source community and thus have the opportunity to give direct feedback to the community and also to be able to deliver the occasional bug fix.

Our Kubernetes landscape works in a relatively centralized way. We, as the Platform Engineering team, are responsible for the installation, management and further development of Kubernetes clusters for our DevOps and development teams. In addition, we are also strong advocates of separating workloads using Kubernetes namespaces rather than splitting them by installing many small Kubernetes clusters.

As a result, we have to operate fewer clusters, but in my view the consideration of who is allowed to do what (classic RBAC questions) occurs more frequently, as we cannot allow individual teams to simply create their namespaces via `kubectl create namespace`. (Hello good old friend of the separation of concerns)

We had already started to establish a self-service portal with existing solutions early on in our Kubernetes journey so that teams could still request their namespaces with as little manual effort as possible. This was easily possible for structured approaches such as projects where it was usually already clear that the workload would exist over a longer period of time.

But how can such a self-service portal with partially manual review processes in the background (despite GitOps) help when we need an environment for our development & automated tests at short notice?

## The idea of ​​Tenama was born

The question to be solved was "How can we enable our teams to create temporary environments in a shared Kubernetes environment in the course of their development pipelines in order to install and test their microservices and dependencies and still somehow ensure control over the creation of namespaces and, as a result, ensure compliance?

The short version can be found in the [README.md](https://github.com/Payback159/tenama) from the Tenama repository.

> Tenama provides a simple REST API that allows non-cluster administrators in a shared Kubernetes environment to create temporary namespaces. tenama handles the creation, management, and cleanup of the temporary namespaces.

So the idea of Tenama was born. Together with feedback from our DevOps people, we developed a REST service that allows our DevOps guys to request namespaces with an expiration date.

With Tenama, we have created the possibility to put the provisioning of namespaces even faster and, above all, in the hands of our DevOps and development teams without giving them more extensive namespace administration rights at cluster level.

## So what exactly makes tenama different?

As the name suggests, tenama not only creates namespaces, but also gives them an expiration date. Tenama allows cluster administrators to configure the maximum lifetime and resource quotas of namespaces and DevOps and development teams can create namespaces within this framework via REST call.

But that's not all, tenama not only creates the namespace itself, but also generates a ServiceAccount whose token is transmitted to the client in the response in order to have the possibility to connect to the namespace via kubectl or other kubernetes client in unexpected cases and to execute manual steps which can in turn be checked in via git and tested in the next run.

For a better understanding, the following 2 sequence diagrams show the two core functionalities of Tenama.

## Namespace creation

DevOps teams can now call the Tenama API via REST. Tenama takes care of setting up the necessary structures such as namespace, quotas, service accounts and the assignment of rights and returns a valid kubeconfig to the user as a response.

![](https://raw.githubusercontent.com/Payback159/tenama/main/docs/diagramms/createNamespaceSeq.png)

## Namespace Cleanup Loop

At an interval that can be configured by the cluster administrator, the cleanup loop can check whether the namespaces created by tenama have reached their expiration date and then takes care of the cleanup of the expired namespaces.

Apart from this housekeeping logic, DevOps teams always have the option of cleaning up the namespaces they have created in advance via the DELETE api.

![](https://raw.githubusercontent.com/Payback159/tenama/main/docs/diagramms/cleanupNamespaces.png)

## What's next?

Tenama is only at the beginning of its journey and there are still many new features to implement to improve the user experience of Tenama.

Very specifically, we want to introduce a global resource quota in the near future. Since individual namespaces are currently quoted but there is no logic to prevent the sum of cluster resources from being "silently" used up by creating many namespaces.

Furthermore, Tenama only offers the management of local users and basic authentication for quick and simple implementation. I am convinced that it makes sense to offer further authentication flows in the future, e.g. via OIDC, to be able to obtain an existing user base from an identity provider and thus better support integration into a larger landscape.

If I have sparked your interest, you are welcome to contact me or simply leave a comment in the [project on github](https://github.com/Payback159/tenama).

I'm grateful for every contribution, be it code, discussion, feedback, documentation or a great logo to give the project more personality. If you just like the project, I would be happy about a star too! 😊

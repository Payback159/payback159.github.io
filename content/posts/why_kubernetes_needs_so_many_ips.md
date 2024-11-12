---
title: 'Why Kubernetes needs so many IPs?'
date: 2024-11-12T19:50:00+00:00
draft: false
Tags:
  - kubernetes
  - resources
  - management
  - google kubernetes engine
  - google
---

| ![Photo by Shubham Dhage on UnsplashForeword](https://cdn-images-1.medium.com/max/1600/0*TwHlcUDbkuVGrGhu) |
|:--:|
| *Photo by Shubham Dhage on UnsplashForeword* |

## Foreword

Why can the networks of a GKE/Kubernetes cluster become so "big"? This article answers this very question. As the name suggests, the Google Kubernetes Engine is based on the open source project Kubernetes. Kubernetes requires 4–5 networks in each installation variant as a basis for further calculations, which Kubernetes or, more precisely, the container network interface provider (CNI) can/may use.

The 4(5) networks are generally named as follows:

* Node-Network
* Cluster-Network
* Pod network
* Service-Network
* (ControlPlane-Network)

Depending on the specific Kubernetes installation, especially with regard to the control plane nodes of a Kubernetes cluster, an additional network may be required. In the case of private Google Kubernetes clusters, for example, a network is also required for the control plane nodes. As these nodes are managed by Google and are not directly visible to the customer, Google still requires a subnet. This enables the control plane to communicate with the rest of the network, such as the network of the nodes on which the Kubernetes agents (kubelets) are running. This allows Google Kubernetes Engine to regularly query the status of the agents.

We therefore have a total of 5 networks that we need to provide for Kubernetes. Now let's move on to the individual networks, what we need them for and a rough rule of thumb for determining the size of the respective networks.

Let's start with the node network.

## Node network

As the name suggests, the node network is the network in which Kubernetes connects the so-called Kubernetes nodes, i.e. the (virtual machines) on which the workloads (the pods/containers) run. To be able to estimate the size of this network, it is important to know, among other things:

* How many pods/workloads are expected on this cluster?
* How many pods per node should be able to run? At this point, it should be mentioned that Kubernetes recommends running a maximum of 110 pods per node.

This gives rise to further questions such as the size per node and the workload density per node.

As these questions already indicate, it is often a question of the expected workload. In some situations, it is possible to increase the workload density per node in order to save costs and IPs, while other workloads deliberately require breadth and distribution across several nodes in order to exploit the full potential and avoid various limitations.

## Cluster network

The cluster network is the network in which Kubernetes provides the IPs for the pods, i.e. the actual workload of Kubernetes. The cluster network should therefore be large enough to provide sufficient IP addresses across the entire cluster, i.e. across all Kubernetes nodes or for all pods. A "very important partner" of the cluster network is the pod network. As the Cluster Network is used as the basis for the calculations of the Kubernetes Pod Network. This means that we can make the following mathematical considerations:

`size of cluster network > size of pod network`

This means that the following questions are interesting for the size of the cluster network:

* How many pods do I want to be able to operate in the cluster in total?

This brings us to the pod network.

## Pod Network

As already mentioned, Kubernetes uses the pod network to divide the cluster network into smaller, more "independent" networks. These networks are assigned to the Kubernetes nodes in addition to the node network. The pod network assigned by the control plane can be managed autonomously by the Kubernetes node in order to be able to supply the pods planned on it with IP addresses; in addition, Kubernetes recommends that each node, i.e. each pod network, should be twice as large as the maximum number of pods. In order to support a rotation of all pods on a Kubernetes node without getting into IP address starvation.

In the case of 110 pods per node, this would be 220 IPs and since there is no subnet with exactly 220 IPs, we have to choose a /24 subnet in this case, since with 254 (256) IPs it is large enough to provide 220 IPs and at the same time is the smallest possible subnet so as not to reserve IPs unnecessarily. Shown here once again as a calculation:

```bash
#Calculation for Pod Network
maxPods pro Node * 2 = neededIPs pro Node
# Example of a /24 network
ipcalc 192.168.0.0/24
Address: 192.168.0.0 11000000.10101000.00000000. 00000000
Netmask: 255.255.255.0 = 24 11111111.11111111.11111111. 00000000
Wildcard: 0.0.0.255 00000000.00000000.00000000. 11111111
=>
Network: 192.168.0.0/24 11000000.10101000.00000000. 00000000
HostMin: 192.168.0.1 11000000.10101000.00000000. 00000001
HostMax: 192.168.0.254 11000000.10101000.00000000. 11111110
Broadcast: 192.168.0.255 11000000.10101000.00000000. 11111111
Hosts/Net: 254 Class C, Private Internet
```

## Service Network

There are so-called services in Kubernetes. In simple terms, a service in Kubernetes can be thought of as a load balancer within the Kubernetes cluster. This means that in order to avoid having to remember the IP addresses of the pods and to distribute the load or increase the availability of a service by operating several pods for it, I can setup an additional service. Kubernetes also requires a network for this service. The separation into a separate network makes sense because, on the one hand, not all pods require a service (e.g. if the pod itself does not need to be addressed directly) and, conversely, there can also be scenarios in which a pod is served by several services.

This means that the following questions are interesting for the size of the service network:

* How many Kubernetes services do I want to be able to operate in the cluster in total?

## ControlPlane Network

As mentioned at the beginning, this is not necessary for every Kubernetes installation. Kubernetes installation, depending on whether I want to operate the ControlPlane in isolation from the rest of the Kubernetes cluster in order to increase the security and reliability of the ControlPlane. ControlPlane nodes in the same subnet, i.e. in the node network, as the rest of the Kubernetes nodes.
In the case of Google Kubernetes Engine, Google has opted for separation. In this case, we as Google customers have relatively little freedom of choice. In return, we get the ControlPlane managed and the security of the ControlPlane is increased from an IP perspective for a small /28 network.

## Epilogue

I should mention that today's CNI providers (e.g. Calico) have one or two tricks, e.g. to share / borrow IP addresses between nodes or there are also projects that allow multiple networks to be attached to a pod (e.g. multus).
In all these cases, these are peculiarities of the CNI provider and should be considered separately if the situation requires it. This article is intended as an introduction to the Kubernetes network and how Kubernetes "sees it".

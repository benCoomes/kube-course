# Inensive Kubernetes

## Cluster Intro

From https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

Kubernetes clusters are multiple computers or virtual machines running software that allows them to coordinate and act as a single unit.

The two important types of machine are:
1. Node: A machine that runs various services as containers. Each node runs `Kubelet`, which interacts with the Control Plane.
2. Control Plane: A machine that runs software for managing the nodes. It schedules updates, deploys new containers, and scales applications. Nodes communicate with the control plane using the Kubernets API.

**Pod**: A pod is a group of containers that make a logical application and are tied together as a deployment unit.

## What can Kubernetes do?

* Start 5 containers with `bestshop/api:1.0` image
* Put an internal load balancer in front of these container
* Start 10 containers with `bestshop/ui:1.0` image
* Put a public load balancer in front of the new containers
* Scale up number of containers to handle traffic spikes
* Release a new version of the API: `bestshop/api:2.1`
* Autoscale containers to based on CPU usage or other metrics
* Reserve CPU/RAM for containers, control where containers are placed
* Advaced rollout patterns (blue/green, canary)

## Kubernetes Architecture

https://2022-11-live.container.training/kube.yml.html#81


The k8s API proccess stores data in `etcd`, a key-value database. We can only interact with etcd through the k8s API.

The scheduler decides where to place containers.

The controller manager monitors container/machine state and tries to keep actual state the same as desired state.

## Types of K8s Environments

* Single machine (like `minikube`)
* Managed K8s: Cloud provider runs control plane and nodes
* Single node control plane + workers
* Stacked control plane + workers
* Advanced control plane + workers
* Control plane and apps running on same nodes
    * common from home clusters, where there aren't many nodes to use

## How many nodes to have?

* Zero is possible (but then no pods can be started)
* A single node is fine for testing and development
* For production, extra capacity is important
* K8s is tested up to 5000 nodes, but this requires lots of tuning

There is a balance to strike between number of clusters and clusters per container.
All nodes in one cluster can become unwieldy, and is problematic if there are cluster-wide problems.
A cluster per application is very expensive.

Kluster deployment should be automated as much as possible so you can have as many clusters as makes sense.
Being hard to deploy/manage is not a good reason to limit the number of clusters.

## When to put containers on the same pod?

Placing containers together means they:
* must be scaled together
* can communicate effeciently over `localhost`

Putting them in different pods means:
* can be scaled independently
* must communicate over remote IP

Typically, pods have multiple containers when a sidecar is used to add-on functionality to the main container.
# Inensive Kubernetes

## Cluster Intro

From https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/

Kubernetes clusters are multiple computers or virtual machines running software that allows them to coordinate and act as a single unit.

The two important types of machine are:
1. Node: A machine that runs various services as containers. Each node runs `Kubelet`, which interacts with the Control Plane.
2. Control Plane: A machine that runs software for managing the nodes. It schedules updates, deploys new containers, and scales applications. Nodes communicate with the control plane using the Kubernets API.

**Pod**: A pod is a group of containers that make a logical application and are tied together as a deployment unit.


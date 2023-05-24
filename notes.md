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


## kubectl

`kubectl` is a CLI wrapper around the k8s API. Anything that can be done with `kubectl` can be done with the API. It can be used to manage servers in a k8s cluster.s

### verbosity

`kubectl` commands can be run with verbosity levels:

```
bencoomes@Benjamins-MBP kube-course % kubectl get nodes -v6
I0522 09:23:14.195670   55786 loader.go:372] Config loaded from file:  /Users/bencoomes/.kube/config
I0522 09:23:14.196504   55786 cert_rotation.go:137] Starting client certificate rotation controller
I0522 09:23:14.210508   55786 round_trippers.go:553] GET https://127.0.0.1:50746/api/v1/nodes?limit=500 200 OK in 10 milliseconds
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   4d15h   v1.23.3
```

The output can be expanded with `-o wide`:
```
bencoomes@Benjamins-MBP kube-course % kubectl get nodes -o wide
NAME       STATUS   ROLES                  AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane,master   4d16h   v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.10.104-linuxkit   docker://20.10.12
```

Or formatted as json with `-o json`, or yaml with `-o yaml`:
```jsonc
// output of:
// kubectl get node minikube -o json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "annotations": {
            "kubeadm.alpha.kubernetes.io/cri-socket": "/var/run/dockershim.sock",
            "node.alpha.kubernetes.io/ttl": "0",
            "volumes.kubernetes.io/controller-managed-attach-detach": "true"
        },
        "creationTimestamp": "2023-05-17T21:34:07Z",
        "labels": {
            "beta.kubernetes.io/arch": "arm64",
            "beta.kubernetes.io/os": "linux",
            "kubernetes.io/arch": "arm64",
            "kubernetes.io/hostname": "minikube",
            "kubernetes.io/os": "linux",
            "minikube.k8s.io/commit": "362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7",
            "minikube.k8s.io/name": "minikube",
            "minikube.k8s.io/primary": "true",
            "minikube.k8s.io/updated_at": "2023_05_17T17_34_09_0700",
            "minikube.k8s.io/version": "v1.25.2",
            "node-role.kubernetes.io/control-plane": "",
            "node-role.kubernetes.io/master": "",
            "node.kubernetes.io/exclude-from-external-load-balancers": ""
        },
        "name": "minikube",
        "resourceVersion": "23030",
        "uid": "7e1c00bc-d96a-400a-bd30-62c704172e8f"
    },
    "spec": {
        "podCIDR": "10.244.0.0/24",
        "podCIDRs": [
            "10.244.0.0/24"
        ]
    },
    "status": {
        "addresses": [
            {
                "address": "192.168.49.2",
                "type": "InternalIP"
            },
            {
                "address": "minikube",
                "type": "Hostname"
            }
        ],
        "allocatable": {
            "cpu": "5",
            "ephemeral-storage": "61255492Ki",
            "hugepages-1Gi": "0",
            "hugepages-2Mi": "0",
            "hugepages-32Mi": "0",
            "hugepages-64Ki": "0",
            "memory": "8039792Ki",
            "pods": "110"
        },
        "capacity": {
            "cpu": "5",
            "ephemeral-storage": "61255492Ki",
            "hugepages-1Gi": "0",
            "hugepages-2Mi": "0",
            "hugepages-32Mi": "0",
            "hugepages-64Ki": "0",
            "memory": "8039792Ki",
            "pods": "110"
        },
        "conditions": [
            {
                "lastHeartbeatTime": "2023-05-22T13:38:19Z",
                "lastTransitionTime": "2023-05-17T21:34:05Z",
                "message": "kubelet has sufficient memory available",
                "reason": "KubeletHasSufficientMemory",
                "status": "False",
                "type": "MemoryPressure"
            },
            {
                "lastHeartbeatTime": "2023-05-22T13:38:19Z",
                "lastTransitionTime": "2023-05-17T21:34:05Z",
                "message": "kubelet has no disk pressure",
                "reason": "KubeletHasNoDiskPressure",
                "status": "False",
                "type": "DiskPressure"
            },
            {
                "lastHeartbeatTime": "2023-05-22T13:38:19Z",
                "lastTransitionTime": "2023-05-17T21:34:05Z",
                "message": "kubelet has sufficient PID available",
                "reason": "KubeletHasSufficientPID",
                "status": "False",
                "type": "PIDPressure"
            },
            {
                "lastHeartbeatTime": "2023-05-22T13:38:19Z",
                "lastTransitionTime": "2023-05-17T21:34:10Z",
                "message": "kubelet is posting ready status",
                "reason": "KubeletReady",
                "status": "True",
                "type": "Ready"
            }
        ],
        "daemonEndpoints": {
            "kubeletEndpoint": {
                "Port": 10250
            }
        },
        "images": [
            {
                "names": [
                    "kubernetesui/dashboard@sha256:ec27f462cf1946220f5a9ace416a84a57c18f98c777876a8054405d1428cc92e",
                    "kubernetesui/dashboard:v2.3.1"
                ],
                "sizeBytes": 216901146
            },
            {
                "names": [
                    "k8s.gcr.io/kube-apiserver@sha256:b8eba88862bab7d3d7cdddad669ff1ece006baa10d3a3df119683434497a0949",
                    "k8s.gcr.io/kube-apiserver:v1.23.3"
                ],
                "sizeBytes": 132450723
            },
            {
                "names": [
                    "k8s.gcr.io/etcd@sha256:64b9ea357325d5db9f8a723dcf503b5a449177b17ac87d69481e126bb724c263",
                    "k8s.gcr.io/etcd:3.5.1-0"
                ],
                "sizeBytes": 132115484
            },
            {
                "names": [
                    "registry.k8s.io/e2e-test-images/agnhost@sha256:7e8bdd271312fd25fc5ff5a8f04727be84044eb3d7d8d03611972a6752e2e11e",
                    "registry.k8s.io/e2e-test-images/agnhost:2.39"
                ],
                "sizeBytes": 127408057
            },
            {
                "names": [
                    "k8s.gcr.io/kube-controller-manager@sha256:b721871d9a9c55836cbcbb2bf375e02696260628f73620b267be9a9a50c97f5a",
                    "k8s.gcr.io/kube-controller-manager:v1.23.3"
                ],
                "sizeBytes": 122423990
            },
            {
                "names": [
                    "nginx@sha256:d20aa6d1cae56fd17cd458f4807e0de462caf2336f0b70b5eeb69fcaaf30dd9c",
                    "nginx:1.16.1"
                ],
                "sizeBytes": 120089353
            },
            {
                "names": [
                    "k8s.gcr.io/kube-proxy@sha256:def87f007b49d50693aed83d4703d0e56c69ae286154b1c7a20cd1b3a320cf7c",
                    "k8s.gcr.io/kube-proxy:v1.23.3"
                ],
                "sizeBytes": 109183127
            },
            {
                "names": [
                    "nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d",
                    "nginx:1.14.2"
                ],
                "sizeBytes": 102757429
            },
            {
                "names": [
                    "k8s.gcr.io/kube-scheduler@sha256:32308abe86f7415611ca86ee79dd0a73e74ebecb2f9e3eb85fc3a8e62f03d0e7",
                    "k8s.gcr.io/kube-scheduler:v1.23.3"
                ],
                "sizeBytes": 52955871
            },
            {
                "names": [
                    "k8s.gcr.io/coredns/coredns@sha256:5b6ec0d6de9baaf3e92d0f66cd96a25b9edbce8716f5f15dcd1a616b3abd590e",
                    "k8s.gcr.io/coredns/coredns:v1.8.6"
                ],
                "sizeBytes": 46808803
            },
            {
                "names": [
                    "kubernetesui/metrics-scraper@sha256:36d5b3f60e1a144cc5ada820910535074bdf5cf73fb70d1ff1681537eef4e172",
                    "kubernetesui/metrics-scraper:v1.0.7"
                ],
                "sizeBytes": 32519053
            },
            {
                "names": [
                    "gcr.io/k8s-minikube/storage-provisioner@sha256:18eb69d1418e854ad5a19e399310e52808a8321e4c441c1dddad8977a0d7a944",
                    "gcr.io/k8s-minikube/storage-provisioner:v5"
                ],
                "sizeBytes": 29032448
            },
            {
                "names": [
                    "k8s.gcr.io/pause@sha256:3d380ca8864549e74af4b29c10f9cb0956236dfb01c40ca076fb6c37253234db",
                    "k8s.gcr.io/pause:3.6"
                ],
                "sizeBytes": 483864
            }
        ],
        "nodeInfo": {
            "architecture": "arm64",
            "bootID": "e180665c-6e0e-46ea-a111-fbb17d7176c5",
            "containerRuntimeVersion": "docker://20.10.12",
            "kernelVersion": "5.10.104-linuxkit",
            "kubeProxyVersion": "v1.23.3",
            "kubeletVersion": "v1.23.3",
            "machineID": "7f42765e713c4b909dde4d5f15b8d18f",
            "operatingSystem": "linux",
            "osImage": "Ubuntu 20.04.2 LTS",
            "systemUUID": "7f42765e713c4b909dde4d5f15b8d18f"
        }
    }
}
```

### .kube/config

Note the reference to the `~/.kube/config` file in the command logs. This file contains information about configured clusters and access token locations. Multiple clusters can be configured in this file.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/bencoomes/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 22 May 2023 09:23:01 EDT
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: cluster_info
    server: https://127.0.0.1:50746
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 22 May 2023 09:23:01 EDT
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/bencoomes/.minikube/profiles/minikube/client.crt
    client-key: /Users/bencoomes/.minikube/profiles/minikube/client.key
```

### resource names

K8s has different resource types, each with shortnames:

```
nodes node no
deployments deployment deploy
services service svc
pods pod po
customresourcedefinitions customresourcedefinition crd
replicesets replicaset rs
statefulsets statefulset sts
```

### kubectl describe

`kubectl describe <resource> <resource_name>` is a helpful command for debugging resources.

```txt
Name:               minikube
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=arm64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=arm64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    minikube.k8s.io/commit=362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
                    minikube.k8s.io/name=minikube
                    minikube.k8s.io/primary=true
                    minikube.k8s.io/updated_at=2023_05_17T17_34_09_0700
                    minikube.k8s.io/version=v1.25.2
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 17 May 2023 17:34:07 -0400
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  minikube
  AcquireTime:     <unset>
  RenewTime:       Mon, 22 May 2023 09:48:44 -0400
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Mon, 22 May 2023 09:48:31 -0400   Wed, 17 May 2023 17:34:05 -0400   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Mon, 22 May 2023 09:48:31 -0400   Wed, 17 May 2023 17:34:05 -0400   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Mon, 22 May 2023 09:48:31 -0400   Wed, 17 May 2023 17:34:05 -0400   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Mon, 22 May 2023 09:48:31 -0400   Wed, 17 May 2023 17:34:10 -0400   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.49.2
  Hostname:    minikube
Capacity:
  cpu:                5
  ephemeral-storage:  61255492Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  hugepages-32Mi:     0
  hugepages-64Ki:     0
  memory:             8039792Ki
  pods:               110
Allocatable:
  cpu:                5
  ephemeral-storage:  61255492Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  hugepages-32Mi:     0
  hugepages-64Ki:     0
  memory:             8039792Ki
  pods:               110
System Info:
  Machine ID:                 7f42765e713c4b909dde4d5f15b8d18f
  System UUID:                7f42765e713c4b909dde4d5f15b8d18f
  Boot ID:                    e180665c-6e0e-46ea-a111-fbb17d7176c5
  Kernel Version:             5.10.104-linuxkit
  OS Image:                   Ubuntu 20.04.2 LTS
  Operating System:           linux
  Architecture:               arm64
  Container Runtime Version:  docker://20.10.12
  Kubelet Version:            v1.23.3
  Kube-Proxy Version:         v1.23.3
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (11 in total)
  Namespace                   Name                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                         ------------  ----------  ---------------  -------------  ---
  default                     nginx-deployment-9456bbbf9-5zmj4             0 (0%)        0 (0%)      0 (0%)           0 (0%)         12m
  default                     nginx-deployment-9456bbbf9-ccftw             0 (0%)        0 (0%)      0 (0%)           0 (0%)         12m
  kube-system                 coredns-64897985d-gjhqh                      100m (2%)     0 (0%)      70Mi (0%)        170Mi (2%)     4d16h
  kube-system                 etcd-minikube                                100m (2%)     0 (0%)      100Mi (1%)       0 (0%)         4d16h
  kube-system                 kube-apiserver-minikube                      250m (5%)     0 (0%)      0 (0%)           0 (0%)         4d16h
  kube-system                 kube-controller-manager-minikube             200m (4%)     0 (0%)      0 (0%)           0 (0%)         4d16h
  kube-system                 kube-proxy-cnwlw                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         4d16h
  kube-system                 kube-scheduler-minikube                      100m (2%)     0 (0%)      0 (0%)           0 (0%)         4d16h
  kube-system                 storage-provisioner                          0 (0%)        0 (0%)      0 (0%)           0 (0%)         4d16h
  kubernetes-dashboard        dashboard-metrics-scraper-58549894f-bqdvx    0 (0%)        0 (0%)      0 (0%)           0 (0%)         4d16h
  kubernetes-dashboard        kubernetes-dashboard-ccd587f44-ccrj9         0 (0%)        0 (0%)      0 (0%)           0 (0%)         4d16h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                750m (15%)  0 (0%)
  memory             170Mi (2%)  170Mi (2%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
  hugepages-32Mi     0 (0%)      0 (0%)
  hugepages-64Ki     0 (0%)      0 (0%)
Events:
  Type    Reason                   Age                From             Message
  ----    ------                   ----               ----             -------
  Normal  Starting                 25m                kube-proxy       
  Normal  Starting                 25m                kubelet          Starting kubelet.
  Normal  NodeHasSufficientMemory  25m (x8 over 25m)  kubelet          Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    25m (x8 over 25m)  kubelet          Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     25m (x7 over 25m)  kubelet          Node minikube status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  25m                kubelet          Updated Node Allocatable limit across pods
  Normal  RegisteredNode           25m                node-controller  Node minikube event: Registered Node minikube in Controller
```

## Namespaces

Namespaces are k8s objects.

```txt
bencoomes@Benjamins-MBP kube-course % kubectl get ns
NAME                   STATUS   AGE
default                Active   4d16h
kube-node-lease        Active   4d16h
kube-public            Active   4d16h
kube-system            Active   4d16h
kubernetes-dashboard   Active   4d16h
```

You can get resources for a specific namespace:

```
bencoomes@Benjamins-MBP kube-course % kubectl get pods --namespace=kube-system
NAME                               READY   STATUS    RESTARTS      AGE
coredns-64897985d-gjhqh            1/1     Running   3 (37m ago)   4d16h
etcd-minikube                      1/1     Running   3 (37m ago)   4d16h
kube-apiserver-minikube            1/1     Running   3 (37m ago)   4d16h
kube-controller-manager-minikube   1/1     Running   3 (37m ago)   4d16h
kube-proxy-cnwlw                   1/1     Running   3 (37m ago)   4d16h
kube-scheduler-minikube            1/1     Running   3 (37m ago)   4d16h
storage-provisioner                1/1     Running   6 (37m ago)   4d16h

bencoomes@Benjamins-MBP kube-course % kubectl get pods --namespace=kubernetes-dashboard 
NAME                                        READY   STATUS    RESTARTS      AGE
dashboard-metrics-scraper-58549894f-bqdvx   1/1     Running   3 (37m ago)   4d16h
kubernetes-dashboard-ccd587f44-ccrj9        1/1     Running   6 (36m ago)   4d16h
```

Or all namespaces:
```
bencoomes@Benjamins-MBP kube-course % kubectl get nodes --all-namespaces                
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   4d16h   v1.23.3
```

Namespaces organize resources, but they don't have any implications for the topology of the cluster (just a logical division, no hardware implications).

## Services

A service is a stable endpoint to connect to "something". Something can be inside or outside the cluster, identified by DNS or IP. Originally they were called 'portals'.

```
bencoomes@Benjamins-MBP kube-course % kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d16h
```

Services can be resolved by name _by processes inside of a pod_:
```
[0.0.0.0] (shpod:N/A) k8s@shpod ~
$ ping kubernetes.default.svc
PING kubernetes.default.svc.cluster.local (10.96.0.1) 56(84) bytes of data.
^C
--- kubernetes.default.svc.cluster.local ping statistics ---
8 packets transmitted, 0 received, 100% packet loss, time 7200ms
```


## Running containers

Containers can be started in k8s with `kubectl`:

```
bencoomes@Benjamins-MBP kube-course % kubectl run pingpong --image alpine ping localhost
pod/pingpong created
bencoomes@Benjamins-MBP kube-course % kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-5zmj4   1/1     Running   0          52m
nginx-deployment-9456bbbf9-ccftw   1/1     Running   0          52m
pingpong                           1/1     Running   0          6s
bencoomes@Benjamins-MBP kube-course % kubectl logs pingpong 
PING localhost (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.031 ms
64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.052 ms
64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.267 ms
64 bytes from 127.0.0.1: seq=3 ttl=64 time=0.138 ms
```

## Deployments

Create a deployment instead of a single pod: `kubectl create deployment pingpong --image=alpine -- ping 127.0.0.1`.
Note `--`: this is used to separate the options of `kubectl create` from the command to run in the container.

This creates a deployment, `pingpong`, with a replica set (ex: `pingpong-5bb8687c6d`), with a pod (ex: `pingpong-5bb8687c6d-5qchp`).

The deployment can be scaled:
```
bencoomes@Benjamins-MBP kube-course % kubectl scale deployment pingpong --replicas 3
deployment.apps/pingpong scaled
```

Why have deployments, replica sets, and pods? Why not just replica sets and pods?
Originally, K8s did not have deployments, but it was difficult to manage. 
For example, in an update, it is easy to create a replica set with the new nodes, then remove the old replica set.
This allows replica sets to always contain the identical nodes, but requires something to associate the replica sets (a deployment).

## Networking with Services

We can create services which act as a single connection point for deployments.

Create deployment and service:
```
kubectl create deployment green --image jpetazzo/color
kubectl expose deployment green --port 80 
kubectl expose deployment green 
```

View service:
```
bencoomes@Benjamins-MBP kube-course % kubectl get services                                
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
green        ClusterIP   10.96.38.17   <none>        80/TCP    3m12s
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   4d20h
```

Scale up:
```
bencoomes@Benjamins-MBP kube-course % kubectl scale deployment green --replicas 3
deployment.apps/green scaled
```

Observe different pods serve curl requests:
```
[0.0.0.0] (shpod:N/A) k8s@shpod ~
$ while sleep 1; do curl 10.96.38.17; done
🟢This is pod default/green-7548fcf59c-w2lw5 on linux/arm64, serving / for 172.17.0.1:50328.
🟢This is pod default/green-7548fcf59c-wq9d2 on linux/arm64, serving / for 172.17.0.1:52722.
🟢This is pod default/green-7548fcf59c-vt2kv on linux/arm64, serving / for 172.17.0.1:41539.
🟢This is pod default/green-7548fcf59c-wq9d2 on linux/arm64, serving / for 172.17.0.1:41253.
🟢This is pod default/green-7548fcf59c-vt2kv on linux/arm64, serving / for 172.17.0.1:62384.
🟢This is pod default/green-7548fcf59c-w2lw5 on linux/arm64, serving / for 172.17.0.1:20155.
🟢This is pod default/green-7548fcf59c-wq9d2 on linux/arm64, serving / for 172.17.0.1:10091.
🟢This is pod default/green-7548fcf59c-wq9d2 on linux/arm64, serving / for 172.17.0.1:1767.
🟢This is pod default/green-7548fcf59c-vt2kv on linux/arm64, serving / for 172.17.0.1:6359.
🟢This is pod default/green-7548fcf59c-w2lw5 on linux/arm64, serving / for 172.17.0.1:65459.
^Chtml>
```

`<service_name>.<namespace>.svc` can also be used to resolve the service:
```
[0.0.0.0] (shpod:N/A) k8s@shpod ~
$ curl green.default.svc
🟢This is pod default/green-7548fcf59c-wq9d2 on linux/arm64, serving / for 172.17.0.1:10990.
```

## Exercise: Dockercoins

See slides here for goal cluster: https://2022-11-live.container.training/kube.yml.html#241

Commands:

```
kubectl create deployment redis --image redis
kubectl expose deployment redis --port 6379

kubectl create deployment hasher --image dockercoins/hasher:v0.1
kubectl expose deployment hasher --port 80

kubectl create deployment rng --image dockercoins/rng:v0.1
kubectl expose deployment rng --port 80

kubectl create deployment worker --image dockercoins/worker:v0.1

kubectl create deployment webui --image dockercoins/webui:v0.1
kubectl expose deployment webui --port 80 --type LoadBalancer
```

If using minikube, create a tunnel for the external IP in another terminal (sudo required):
```
minikube tunnel
```

Then the UI should be avilable at http://localhost/index.html !

Scaling the worker deployment changes the 'mining' rate:
```
kubectl scale deployment worker --replicas 2
```
---
author: Dennis Kruyt
categories:
- kubernetes
- kubelet
- containerd
- deprecated
- docker
- dockershim
- "1.22"
- cri
- migrate
- "1.20"
date: "2021-03-16T21:17:54Z"
description: ""
draft: false
cover:
  image: /images/2021/03/mvii.jpeg
slug: migrate-docker-containerd-kubernetes
tags:
- kubernetes
- kubelet
- containerd
- deprecated
- docker
- dockershim
- "1.22"
- cri
- migrate
- "1.20"
title: Migrate from Docker to Containerd in Kubernetes
---


Kubernetes is [deprecating Docker](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#deprecation) as a container runtime after v1.20. Don't Panic 😱 Docker containers are still supported, but the dockershim/Docker, the layer between Kubernetes and containerd  is deprecated and will be removed from version 1.22+.

{{< bookmark url="https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/" title="Don’t Panic: Kubernetes and Docker" description="Authors: Jorge Castro, Duffie Cooley, Kat Cosgrove, Justin Garrison, Noah Kantrowitz, Bob Killen, Rey Lejano, Dan “POP” Papandrea, Jeffrey Sica, Davanum “Dims” Srinivas\nKubernetes is deprecating Docker as a container runtime after v1.20.\nYou do not need to panic. It’s not as dramatic as it sounds.\nT…" icon="https://kubernetes.io/favicons/apple-touch-icon-180x180.png" author="Wednesday, December 02, 2020" publisher="Kubernetes" thumbnail="https://kubernetes.io/images/favicon.png" caption="" >}}

So if you are running docker you need to change to a supported container runtime interface (CRI). containerd is a good choice, it is already running on your Kubernetes node if you are running Docker.

An extra advantage is there is less overhead, there is no docker-shim and Docker translation layers as  you can see is this diagram.

{{< figure src="/images/2021/03/docker_containerd.png" caption="change from docker shim to containerd CRI" >}}

## How to migrate

First we check what container runtime is currently running. We do this with `kubectl get nodes -o wide`

As we can see we are runnig Docker as runtime.

```
NAME       STATUS   ROLES                  AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                             CONTAINER-RUNTIME
k8s-cn01   Ready    control-plane,master   78m     v1.20.4   10.65.79.164   <none>        Ubuntu 20.04.2 LTS   5.4.0-67-generic                           docker://20.10.5
k8s-wn01   Ready    <none>                 64m     v1.20.4   10.65.79.131   <none>        Ubuntu 20.04.2 LTS   5.4.0-67-generic                           docker://20.10.5
k8s-wn02   Ready    <none>                 4m16s   v1.20.4   10.65.79.244   <none>        CentOS Linux 8       4.18.0-240.15.1.el8_3.centos.plus.x86_64   docker://20.10.5
```

Now we check if we have the containerd cli `/usr/bin/ctr` and the namespace _moby_ is there. _moby_ is the namespace from docker.

```
NAME LABELS
moby
```

And we can list the running containers in this namespace

```
CONTAINER                                                           IMAGE    RUNTIME
04f9500885c473c9cb2b4f8d09dc4feea5c24838519b9a01251011830bab16a2    -        io.containerd.runc.v2
57d4c75ab9947829228a087b857b203c48a9d1c83de0a1b49af3624fb08c9d33    -        io.containerd.runc.v2
934c007a259018a5cbda56dd8e066a66f2c9cfcb8003e7f8d25833fe462582fd    -        io.containerd.runc.v2
94315822d8f8a05e1be5adb7e5c18add33cbf2604057100c87572b5fc55169cd    -        io.containerd.runc.v2
dfa01906e845239c74a0b35d457e845382468dd9ad6e99dd0c16be30f8a23a2d    -        io.containerd.runc.v2
```

If everything looks fine we can change the cri, We change one node at a time and first the worker nodes then our control node. If you have only one control node you will lose access to the cluster, this will be temporally and it should recover it self.

### Cordon and Drain node

We need to cordon and drain the nodes, so that are workloads are rescheduled.

```
root@k8s-cn01:~# kubectl cordon k8s-wn01
node/k8s-wn01 cordoned

root@k8s-cn01:~# kubectl drain k8s-wn01 --ignore-daemonsets
node/k8s-wn01 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-9wnh4, kube-system/weave-net-pgptm
evicting pod default/nginx-6799fc88d8-r44x9
pod/nginx-6799fc88d8-r44x9 evicted
node/k8s-wn01 evicted

root@k8s-cn01:~# kubectl get nodes
NAME       STATUS                     ROLES                  AGE    VERSION
k8s-cn01   Ready                      control-plane,master   138m   v1.20.4
k8s-wn01   Ready,SchedulingDisabled   <none>                 124m   v1.20.4
k8s-wn02   Ready                      <none>                 64m    v1.20.4
```

### Stop services

```
  systemctl stop kubelet
  systemctl stop docker
```

### Remove docker (optional)

We remove Docker, this is not necessary but make things more clear, less prone for mistakes later and we will save some disk space...

```
apt purge docker-ce docker-ce-cli
OR
yum remove docker-ce docker-ce-cli
```

### Containerd config

Disable the line disabled_plugins in `/etc/containerd/config.toml` so the cri interface is loaded.

```
#disabled_plugins = ["cri"]
```

If there is **no config** file for containerd, you can generate a new default file.

```
containerd config default > /etc/containerd/config.toml
```

The restart containerd

```
systemctl restart containerd
```

### Change runtime

Edit the file `/var/lib/kubelet/kubeadm-flags.env` and add the containerd runtime to the flags. `--container-runtime=remote` and `--container-runtimeendpoint=unix:///run/containerd/containerd.sock"`

So the kubeadm-flags file would look something like this.

```
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
--resolv-conf=/run/systemd/resolve/resolv.conf --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
```

### Start kubelet

After changing the runtime we can start the kubelet service.

```
systemctl start kubelet
```

### Check

Now when we run `kubectl get nodes -o wide` and we see containerd a the runtime for the node we just changed.

```
NAME       STATUS  		 			ROLES                  AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                             CONTAINER-RUNTIME
k8s-cn01   Ready	 		    	control-plane,master   131m   v1.20.4   10.65.79.164   <none>        Ubuntu 20.04.2 LTS   5.4.0-67-generic                           docker://20.10.5
k8s-wn01   Ready,,SchedulingDisabled	    <none>                 117m   v1.20.4   10.65.79.131  		 <none>        Ubuntu 20.04.2 LTS   5.4.0-67-generic                           containerd://1.4.4
k8s-wn02   Ready   			 <none>                 57m    v1.20.4   10.65.79.244   <none>        CentOS Linux 8       4.18.0-240.15.1.el8_3.centos.plus.x86_64   docker://20.10.5
```

The node we just changed is still cordoned. So we can uncordon it now.

```
root@k8s-cn01:~# kubectl uncordon k8s-wn01
node/k8s-wn01 uncordoned

root@k8s-cn01:~# kubectl get nodes
NAME       STATUS   ROLES                  AGE    VERSION
k8s-cn01   Ready    control-plane,master   143m   v1.20.4
k8s-wn01   Ready    <none>                 129m   v1.20.4
k8s-wn02   Ready    <none>                 69m    v1.20.4
```

If we check the namespaces on the node now, we see a new namespace, _k8s.io_. The _moby_ namespace is now empty, no containers are running in this namespace all the containers are now running the the k8s.io namespace.

```
root@k8s-wn01:~# ctr namespaces list
NAME   LABELS
k8s.io
moby

root@k8s-wn01:~# ctr --namespace moby container list
CONTAINER    IMAGE    RUNTIME

root@k8s-wn01:~# ctr --namespace k8s.io container list
CONTAINER                                                           IMAGE                                    RUNTIME
08d4b5ca1f0ddd08fff7f64ea4eb12be66b8ec860d119565e553c84d16942d26    docker.io/weaveworks/weave-kube:2.8.1    io.containerd.runc.v2
1c9e3f61f542b12abb4b849075b084fb7fe3f69b89ce668d73022c2cd647bcd1    k8s.gcr.io/pause:3.2                     io.containerd.runc.v2
1f8e0c5a6f8219de1c8f25bbb28d5d5c71b74e9ccfb9620007701847d29f23a2    k8s.gcr.io/kube-proxy:v1.20.4            io.containerd.runc.v2
39296ebd6017a7c83cd58004c94708c927f10a996a4e6ba0bbf003c6713fe713    docker.io/weaveworks/weave-kube:2.8.1    io.containerd.runc.v2
67f812954f46fa5c1f6ab39e681e2481f868309f28bd1b8ba44cce53f5c0071c    docker.io/weaveworks/weave-npc:2.8.1     io.containerd.runc.v2
9caed1d57d40cedef736e45adf550eda6a0befd32712e5b6af5d711681ba71f0    k8s.gcr.io/pause:3.2                     io.containerd.runc.v2
```

We have changed successfully the cri, now we can move to the next node and repeat everything.


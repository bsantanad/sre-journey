# 🐋 k8s up and running


> ⚠️ These are notes taken from the book [_Kubernetes Up and Running by Brendan
> Burns, Joe Beda, and Kelsey Hightower O'Reilly_](https://learning.oreilly.com/library/view/kubernetes-up-and/9781098110192).
> **Please go read it and just use this as reference**.

# Table of Contents

<!-- toc -->

# Introduction

<aside>
💡 K8s is an open source orchestrator for deploying containerized applications
</aside>

It has become the standard API for building cloud native applications. It
provides the software needed for build and deploy, reliable, scalable
distributed systems.

**Distributed systems:** Most services nowadays are delivered via the network,
they have different moving parts in different machines talking to each other
over the network via APIs.

**Reliable:** Since we rely a lot on these distributed systems, they cannot
fail, even if some part of them crashes, they need to be kept up and running,
and be fault-tolerant

**Availability:** They must still be available during maintenance, updates,
software rollouts and so on.

**Scalable:** We live in a highly connected society, so these systems need to
be able to grow their capacity without radical redesign of the distributed
systems.

Most of the reasons people come to use containers and container orchestration
like k8s, can be boil down to 5 things:  **velocity, scaling, abstracting your
infra, efficiency, cloud native ecosystem**

## Development Velocity

Today software is delivered via de network more quickly than ever, but velocity
is not just a matter of raw speed, people are more interested in a highly
reliable service, all users expect constant uptime, even as the software is
being updated.

Velocity is measures in term of the number of things you can ship while
maintaining a highly available service.

k8s give you the tools to to do this, the core concepts to enable this are:

- **Immutability**

    Software used to be mutable (imperative), you installed `vim` using `dnf`
    and then one day you would do `dnf update`  and it would add modifications
    on top of the binary that you already have for `vim`. This is what we call
    *mutable* software.

    On the other hand, *immutable* software, you would not add on top of the
    previous release, you would build the image from zero. Which is how
    containers work. Basically when you want to update something in a
    container, you do not go inside it and do `dnf update vim` you create a new
    image with the version of vim you want, destroy the old one, and create a
    container with the new one.

    This makes rollbacks way easier, and it keeps tracks of what changed from
    version to version.

    <aside>
    💡 Imperatively change running containers (going into the container and
    update a package for example) is an anti-pattern, and should be avoided.
    </aside>

- **Declarative Configuration**

    This is an extension of immutability but for the configuration. It
    basically means that you have a file where you defined the ideal state of
    your system. The idea of storing this files in source control is often
    called “infrastructure as code”.

    The combination of declarative state stored in a version control system and
    the ability of k8s to make it  match the state makes rollback super easy.

- **Self-Healing Systems**

    K8s will continuously take action to make sure the state on the declarative
    configuration is met. Say it has declared 3 replicas, then if you go and
    create one extra, it will kill it, and if you were to destroy one, it would
    create it.

    This means that k8s will not only init your system but will guard it
    against any failures that might destabilize it.


## Scaling Your Service and Your Teams

K8s is great for scaling because of its decoupled architecture.

### Decoupling

In a decoupled architecture each components is separated from each other by
defined APIs and service loads

This basically means that we can isolate each service.
- The API provide a buffer between implementer and consumer.
- Load balancing provide a buffer between running instances of each service.

When we decouple the services, it makes it easier to scale, since each small
team can focus on a single _microservice_

### Scaling for Applications and Clusters

Since your app is deployed in containers, which are immutable, and th
configuration o  is also declarative, scaling simply becomes a matter of
changing a number in a configuration file. Or you can even tell k8s to do that
for you.

Of course k8s will be limited to the resources you have, it will not create a
physical computer on demand. But k8s makes it easier to scale the cluster
itself as well.

Since a set of machines in a cluster are identical and the apps are decoupled
from the specifics of the machine by containers, scaling a cluster is just a
matter of provisioning the system and joining it to a cluster.

K8s can also come in handy when trying to forecast costs on growth and scaling.
Basically, if you have different apps that need to scale from many teams, you
can aggregate them all in a single k8s, and try to forecast the usage of the 3
apps together.


### Scaling Development Teams

Teams are ideal when using the two-pizza team when (6 to 8 people), larger
teams tend to have issues of hierarchy, poor visibility, infighting and so on.

K8s can help with that, since you can have multiple small teams working on
different microservices and  aggregate them all into a single k8s cluster.

K8s has different abstractions that make it easier to build decoupled
microservices.

- __pods__: are groups of containers, can contain images developed by different
  teams into a single deployable unit.
- __k8s services__: provide load balancing, naming and discovery to isolate the
  microservices.
- __namespaces__: provide isolation and access control, so that each microservice
  can control the degree to which other services interact with it.
- __ingress__: easy to use front end that can combine microservices into a single
  API.

### Separation of Concerns for Consistency and Scaling

K8s leads to greater consistency for the lower levels of the infrastructure.
Basically the devs need to only worry about their app meeting the SLA (service
level agreements) while the orchestration admin only need to worry about
reaching the SLA from their side.

## Abstracting You Infrastructure

When you build your app in terms of containers and deploy it via k8s APIs, you
make moving your app between environments super easy. It is simply a question
of sending the declarative config to a new cluster.

K8s make building deploying and managing your app truly portable across a wide
variety of environments

## Efficiency

Efficiency is measured by the ratio of the useful work performed by a machine.

K8s enables efficiency in multiple ways. Since developers do not need to use a
whole machine for himself/herself, but rather one container, multiple users can
share the same baremetal. Meaning less idle CPUs.

## Cloud Native Ecosystem

K8s is open source and has a huge community building a thousand things on top
of it. You can check the maturity of different projects in Cloud Native
Computing Foundation. The maturities are sandbox incubating and graduated,
going from less mature to most mature.

## Summary

Fundamentally k8s was design to give developers more velocity efficiency and
agility. Now that we have seen why to use it, lets see how to use it.


# Deploying a k8s cluster

You can use a cloud provider, deploying on  bare-metal quite hard, other good
alternatives.

[http://kind.sigs.k8s.io](http://kind.sigs.k8s.io/docs/user/configuration/)
that has a really cool logo, and works on using containers for the nodes.

Go check the pages of cloud providers on how to deploy it there since it will
change depending on the provider.

One thing to bare in mind is that in order to use `kubectl` with your cluster
you need a `~/.kube/config` file, probably your cloud provider will give that
to you.

Also you can go to [k8s cheatsheet (describe
cluster)](/k8s_cheatsheet.html#describe-cluster) to see the basics commands on
describing a cluster.

I will  just cover some of the commands here, like `k describe nodes <node
name>`.

At the top you can see basic node info:
```
Name:               kind-control-plane
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=arm64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=arm64
```
We can see that it is linux on ARM

Next we can see info about the operation of the node
```
Conditions:
Type             Status  LastHeartbeatTime  LastTransitionTime Reason                       Message
----             ------  -----------------  ------------------ ------                       -------
MemoryPressure   False   29 Apr 2024        29 Apr 2024        KubeletHasSufficientMemory   kubelet has sufficient memory available
DiskPressure     False   29 Apr 2024        29 Apr 2024        KubeletHasNoDiskPressure     kubelet has no disk pressure
PIDPressure      False   29 Apr 2024        29 Apr 2024        KubeletHasSufficientPID      kubelet has sufficient PID available
Ready            True    29 Apr 2024        29 Apr 2024        KubeletReady                 kubelet is posting ready status
```

Here we can see that the node is Ready, that it has sufficient memory, it has
no disk pressure, and that is has sufficient PID.

Then we see info about the capacity of the system.
```
Capacity:
cpu:             1
memory:          1933908Ki
pods:            110
Allocatable:
cpu:             1
memory:          1933908Ki
pods:            110
```

Then info about software in the node like OS image, docker version all that.
```
System Info:
# more stuff
Kernel Version:             6.5.6-200.fc38.aarch64
OS Image:                   Debian GNU/Linux 12 (bookworm)
Operating System:           linux
Architecture:               arm64
Container Runtime Version:  containerd://1.7.13
Kubelet Version:            v1.29.2
Kube-Proxy Version:         v1.29.2
```

Then info about the pods running on the node:
```
Non-terminated Pods:          (9 in total)
Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
---------                   ----                                          ------------  ----------  ---------------  -------------  ---
kube-system                 coredns-76f75df574-92286                      100m (10%)    0 (0%)      70Mi (3%)        170Mi (9%)     23s
kube-system                 coredns-76f75df574-m2z2k                      100m (10%)    0 (0%)      70Mi (3%)        170Mi (9%)     23s
kube-system                 etcd-kind-control-plane                       100m (10%)    0 (0%)      100Mi (5%)       0 (0%)         39s
kube-system                 kindnet-qm29d                                 100m (10%)    100m (10%)  50Mi (2%)        50Mi (2%)      23s
kube-system                 kube-apiserver-kind-control-plane             250m (25%)    0 (0%)      0 (0%)           0 (0%)         39s
# more stuff
```

We can see that k8s tracks both `requests` and upper `limits` for resource for
each Pod. The difference is that resources requested by a pod are guaranteed to
be present in the node, while pod's limit is the max amount of given resource
that pod can consume

## Cluster Components

Many components that make up the k8s cluster are actually deployed using k8s
itself. All of them run in the kube-system namespace

- k8s proxy

    It routes network traffic to load-balanced services in the k8s cluster. It
    must be present on every node in the cluster

    ```bash
    % k get daemonSets --namespace=kube-system kube-proxy
    ```

- k8s dns

    provides naming and discover for the services that are defined in the
    cluster

    ```bash
    % k get deployments --namespace=kube-system coredns
    ```


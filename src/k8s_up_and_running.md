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

I will  just cover some of the commands here, like

```bash
k describe nodes <node name>
```

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

# Common `kubectl` commands

> I will repeat the commands here and in the page: [k8s
> cheatsheet](/k8s_cheatsheet.html) but I will go a bit more into detail here.
> I will also `alias k=kubectl`

`kubectl` is the way you talk with k8s API. Here we will go over basic
`kubectl` commands.

## Namespaces

Namespaces are like folders that hold a set of objects. They are used to
organize the cluster.

By default the `kubectl` command interacts with the `default` namespace, but
you can specify which you want to use by `-n` or `--namespace=<ns>`.

You can also use `--all-namespaces` flag to refer to all the ns.

## Contexts

If you want to change the namespaces in a more _permanent_ way, you can use
contexts. These get recorded in your `~/.kube/config` file. (That file also
stores how to find and auth to your cluster)

You can create a with a different default namespaces using

```bash
% kubectl config set-context my-context --namespace=mystuff
```

We need to tell `kubectl` to start using it.

```bash
% kubectl config use-context my-context
```

You can also use context to manage different clusters users for auth using
`--users` or `--cluster` flags with `set-context` read the man for more info

## Viewing k8s API Objects

Basically k8s is an API and kubectl is just an http client. To see the k8s
objects (everything represented by a RESTful resource) we use `k get <resource
name>`

By default it will throw to the STDOUT human readable stuff, but you can also
use `-o json` or `-o yaml` in case you want it in specific format.

Also when using `awk` you may want to use `--no-haders`.

You can also get multiple objects
```bash
k get pods,services
```

To get more info on a particular object
```bash
k describe <resource name> <obj name>
```
If you want to  see like a mini man page of an specific k8s object:
```bash
% k explain pods
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:

```

Sometimes you want to continually observe the state of a k8s resource, like
when waiting for an app to restart of something you can use the `--watch` flag:
```
k get pods -n kube-system --watch
```

## Creating, updating and destroying k8s objects

Objects in the k8s API are represented as JSON or YAML files. We can query the
server for those, or post them with an API request.

You use these JSON and YAML files to create, update or delete objects in your
k8s server.

**Create** an object from `obj.yaml` file.
```bash
% k apply -f obj.yaml
```
The yaml file will have the resource type of the object.

You can use the same command to **update** an object
```bash
% k apply -f obj.yaml
```

If nothing change it will not do anything, it will exit successfully, good for
`for` loops. You can do `--dry-run` to print objects to the terminal without
actually sending them.

To do interactive edits you can use
```bash
% k edit <resource-name> <obj-name>
# example k edit pod coredns-76f75df574-k97nj  -n kube-system
```
It will open the `yaml` in a text editor, wonce you save it will automatically
be uploaded back to the k8s cluster.

The `apply` command also has a history of the previous configurations
`edit-last-applied`, `set-last-applied` and `view-last-applied` for example:
```bash
% k apply -f myobj.yaml view-last-applied
```
To delete you can simply run
```bash
% k delete -f myobj.yaml
```
You can also delete an object without the file using
```bash
% k delete <resource-name> <obj-name>
```

## Labeling and Annotating Objects

You can update the labels and annotations using `label` and `annotate` (who
would have thought).

For example to add the `color=red` label to the pod named `bar` you can run:
```bash
k label pods bar color=red
```
You can not rewrite unless using `--overwrite` flag.

Finally you can remove a label by doing
```bash
k labels pods bar color-
```

## Debugging Commands

You can check logs from a pod by:

```bash
k logs <pod-name>
```

If you have multiple container in your pods you can choose which container to
view using the `-c` flag.

If you want to follow add `-f`.

You can also use the `exec` command to execute a command in a running
container.
```bash
k exec -it <pod-name> -- bash
```

If you do not have bash or a shell in your container, you can `attach` to a
running process

```bash
k attach -it <pod-name>
```

You can also copy files to and from a container using `cp`
```bash
k cp <pod-name>:/path/to/remote/file /path/to/local/file
```

This will copy a file from the container to your local machine.

You can also forward traffic from your local system to the pod
```bash
k port-forward <pod name> 8080:80
```
Forwards traffic from the local machine on port 8080 to the remote container on
port 80

If you want to see the last 10 events in a namespace
```
k get events
```
You can also stream events `--watch`

Finally,  to check cluster resources being used
```
k top nodes
```
or
```
k top pods
```
These commands will only work if metrics servers are present, which they likely
are.

## Cluster Management

Cordon and drain a particular node:

- _cordon_: prevent future pods form being scheduled onto that machine
- _drain_: remove any pods that are currently running on that machine.

Useful for removing physical machine for repairs or upgrades. You would do `k
cordon` and then `k drain` to safely remove the machine form the cluster.

Once the system is back online do `k uncordon` there is no `undrain` it will
naturally get back to normal.

# Pods

K8s groups multiple containers into a single atomic unit called a _pod_ (This
also goes with the docker theme, since a group of whales is a pod lol)

A pod is a collection of app containers and volumes running in the same
environment. Containers in a pod always land in the same machine.

Apps running in the same pod share, ip addresses and port, hostname, and can
communicate to each other using interprocess communication channels. Meaning
they share:

- network namespace
- UTS namespace
- IPC namespace

## Thinking in Pods

_What should go in a Pod?_ To answer this question you go look at your
different containers and ask _"Will these containers work correctly if they
land on different machines?"_ If no, then a pod is the correct way of grouping
them. If yes, you can use multiple pods.

An example, say you want to deploy a wordpress app with a mysql database. Would
you put them in the same pod? I already read the book so I know the answer is
no, but why?

The answer is no because both of those services do not need to scale together.
The wordpress app itself is stateless, so you can create more wordpress pods if
you are scaling. On the other hand a mysql database is much different to scale,
it can be trickier, and you are likely to increase the resources dedicated to a
single mysql pod.

## The Pod Manifest

Pods are describe in a _manifest_ which is a text-file representation of the
k8s API object.

As you might remember from the fist chapter, k8s uses a _declarative
configuration_, which basically means that you tell k8s your ideal state of the
world and it will take action to ensure the state becomes a reality.

When you send a manifest to k8s, it will accept it and process it before
storing it in persistent storage `(etcd)`. The _scheduler_ will find pods that
have not been assigned to a node, and will place the pods onto those nodes
depending on the resources and constrains you specified in your manifest. K8s
will try to ensure pods from the same app are in different machines for
reliability.

You can see the pods with
```bash
k get pods
```

You can delete a pod with
```bash
k delete pods/<name>
```

### Creating a manifest

Pod manifest should be treated in the same way a source code, adding comments
to help explain the pod to new team members and stuff like that.

The manifests include a couple of key fields and attributes,
- `metadata`:
    - `name`: a unique identifier for the deployment
    - `namespace`: the Kubernetes namespace where the deployment will live
    - `labels`: key-value pairs that can be used to filter and categorize objects
- `spec`:
    - `containers`:
        - `name`: a unique identifier for the container
        - `image`: the Docker image to use for this container
        - `command`: the default command to run in the container
        - `args`: any additional arguments to pass to the command
        - `volumeMounts`: mounts for persistent storage or other files
    - volumes:
        - `name`: a unique identifier for the volume
        - `persistentVolumeClaim`: claims a Persistent Volume resource
- `selectors`: used to filter which pods are included in this deployment

Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-04-29T20:02:16Z"
  generateName: coredns-76f75df574-
  labels:
    k8s-app: kube-dns
    pod-template-hash: 76f75df574
  name: coredns-76f75df574-k97nj
  namespace: kube-system
  # more stuff
  uid: 4dd80223-de4c-49d7-b407-31908ae96b9e
spec:
  # more stuff
  containers:
  - args:
    - -conf
    - /etc/coredns/Corefile
    image: registry.k8s.io/coredns/coredns:v1.11.1
    imagePullPolicy: IfNotPresent
    # more stuff

    volumeMounts:
    - mountPath: /etc/coredns
      name: config-volume
      readOnly: true
    # more stuff

  volumes:
  - configMap:
      defaultMode: 420
      items:
      - key: Corefile
        path: Corefile
      name: coredns
      # more stuff

status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-04-29T20:02:19Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
  # more stuff
```

You can create a pod from them by using `k apply -f manifest.yaml` as seen in
previous sections.

## Some Useful Commands

```bash
# get info
k get pods  # list pods
k describe pods <podname> # more info on the k8s object

# manipulate pods
k delete pods/<podname> # delete pod, all info associated to it will be deleted
k logs <podname> # check logs
k exec <podname> -- date # run date command inside pod
k exec -it <podname> -- bash # interactive tty
k cp # keep in mind that copying files into a container is an antipattern.
```

## Health Checks

When you are running an app in k8s, it is automatically kept alive by a process
health check. This ensure that the app is always running, if it is not it k8s
will restart it.

Sometimes, checking if the process is alive is not enough, maybe it was left
hanging or something like that. K8s has something to check application
_liveness_ and other stuff for your app.

There are different kinds of probes here are some:

- **Liveness Probe**
    This will check app specific stuff, for example checking if http is
    returning 200.

    While the default response to a failed liveness is to restart you can
    change what happened with `restartPolicy`

    Here is an example of a `livenessProbe`
    ```
    spec:
      containers:
      - name: my-container
        image: my-image:latest
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
            scheme: HTTP
            initialDelaySeconds: 30
            timeoutSeconds: 5
    ```

- **Readiness Probe**
    These probes check if an app is ready to serve user requests. Apps that
    fail are removed from the load balancers

- **Startup Probe**
    Enables you to poll slowly from a slow-starting container, while also
    enabling  responsive liveness checks once the container has initialized

## Resource Management

K8s allow you to increase the overall utilization of the machines that are in
the cluster.

Utilization is defined as the amount of resource actively being used divided by
the amount of a resource that has been purchased

Example, if you purchase a one-core machine, and your app uses one tenth of a
core, then your utilization is 10%

You can tell k8s the upper limit and lower limit of resources you want for an
specific container. Keep in mind that this is on a per-container basis, the
whole pod will the sum of the containers.

You can do this by setting up `resources` in the manifest. The lower limit is
`requests` and the upper limit is `limits`.

```yaml
spec:
    containers:
    - image: blabla:latest
      name: jose
      resouces:
        cpu: "500m"
        memory: "128Mi"
      limit:
        cpu: "1000m"
        memory: "256Mi"
    # more stuff
```

## Persisting Data with Volumes

When a container restarts, any data in it will also be deleted. Which is good
since we do not want to leave around cruft.

We can still add volumes to our pods though, that will persist this deletion


In order to do this, you need to two things to the manifest `spec.volumes`,
which is an array of all volumes that the pod will need.

And the other is `volumeMount` which specify inside the container definition
the file system to use.

So basically one is like, what volumes will the whole pod use, and the other is
specific to each container.

```yaml
spec:
  containers:
  - name: my-container
    image: busybox:latest
    volumeMounts:
    - name: my-filesystem
      mountPath: /mnt
      subPath: data
  volumes:
  - name: my-filesystem
    persistentVolumeClaim:
      claimName: my-pvc
```

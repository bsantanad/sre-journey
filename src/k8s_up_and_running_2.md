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

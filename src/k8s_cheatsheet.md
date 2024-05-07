# 📝 k8s cheatsheet

Some useful commands

> *using kind?*
>
> in you mac? with podman?
>
> ```bash
> podman machine start;
> export KIND_EXPERIMENTAL_PROVIDER=podman;
> kind create cluster
> ```

# Describe Cluster

- Check version

    ```bash
    % k version  # what do you think it does? :)
    *Client Version: v1.28.3
    Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
    Server Version: v1.29.2*
    ```

- Check cluster status

    ```bash
    % k get componentstatuses # will be deprectated
    Warning: v1 ComponentStatus is deprecated in v1.19+
    NAME                 STATUS    MESSAGE   ERROR
    controller-manager   Healthy   ok
    scheduler            Healthy   ok
    etcd-0               Healthy   ok
    ```

    or talk directly to the endpoint

    ```bash
    % k get --raw='/readyz?verbose'
    [+]ping ok
    [+]log ok
    [+]etcd ok
    [+]etcd-readiness ok
    [+]informer-sync ok
     # more messages
     readyz check passed
    ```

- List k8s nodes

    ```bash
    % k get nodes
    NAME                 STATUS     ROLES           AGE   VERSION
    kind-control-plane   Ready   control-plane   13s   v1.29.2
    node01               Ready   <none>          13s   v1.29.2
    ```

    In k8s nodes are separated into `control-panel` nodes, like the API server,
    scheduler, etc. which manage the cluster, and `worker` nodes where your
    containers will run.

- Get more information about a specific node (prints bunch of info)

    ```bash
    % k describe nodes kind-control-plane
    ```
    more info on the ouput
    [here](/k8s_up_and_running.html#deploying-a-k8s-cluster)

- See the proxies if they are under the API object named DaemonSet

    ```bash
    % k get daemonSets --namespace=kube-system kube-proxy
    ```

- See DNS k8s deployment

    ```bash
    % k get deployments --namespace=kube-system coredns
    ```

- Wait for nodes to be ready

    ```bash
    % k wait --for=condition=Ready nodes --all
    node/node1 condition met
    node/node2 condition met
    ...
    ```

# Manage your context

- Change namespace for current context

    ```bash
    % kubectl config set-context --current --namespace=<some ns>
    ```

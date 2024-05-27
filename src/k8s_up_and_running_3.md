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

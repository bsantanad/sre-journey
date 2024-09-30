# Authentication and Authorization

Access control for the k8s api.

![auth-k8s.png](img/auth-k8s.png)

This process has 4 steps.

1. Authentication: checks that the identity of the client is valid.
2. Authorization: okay, your identity is good, but do you have access to what
   are trying to do? (Here it comes into place RBAC, read below for more info)
3. Admission Control: you have access alright, did you send the request in a
   well-formed manner?
4. Validation: checks that the resource included in the request is valid.

<!-- toc -->

## Authentication via credentials in kubeconfig


Some useful commands:
```
k config view # renders the contents of the kubeconfig file
k config current-context # shows the currently-selected context
k config user-context johndoe # switch context
```

Inside `.kube/config` we can see a yaml
- `clusters.cluster.server`: api server endpoint for connecting with a cluster.
- `contexts.context`: groups access params
  - `contexts.context.cluster`: name of the cluster
  - `contexts.context.user`: user name
- `contexts.name`: name of the context

Here is an example of a `kind` cluster.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <token>
    server: https://127.0.0.1:51224
  name: kind-kind-cluster
contexts:
- context:
    cluster: kind-kind-cluster
    user: kind-kind-user
  name: kind-kind
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: kind-kind-user
  user:
    client-certificate-data: <cert>
    client-key-data: <cert>
```


## Role-based Access Control (RBAC)

RBAC works after the authentication process. Now that we know the
credentials are valid we have to ask. Does this user have permission to do
this?

RBAC defines policies for users, groups and processes, that allow or
disallow access to k8s objects. For exmaple there can be a group that can
only list and create pods, but not touch `secrets` or `configmaps`.

Enabling RBAC is a most for any org.

High level breakdown.

1. Subject
  - groups
  - users
  - service accounts
2. API resources
  - configmap
  - pod
  - deployment
3. Operations
  - create
  - list
  - watch
  - delete

There are 2 k8s resources/objects we care about.
- `roles`: define the role
- `rolebining`: attach the role to a user.

This are namespace scoped. If you want to do this but in all the cluster
then we have `cluster` and `clusterolebinding`.

In this case, creating the resources with the imperative command is easy:
```yaml
k create role read-only --verb=list,get,watch --resource=pods,deployment,services
```
Then to bind it:
```yaml
k create rolebinding read-only-binding --role=read-only --user=johndoe
```

Of course we can describe, get, and any other operation you can do to any
other k8s resource/obj.

## Using a ServiceAccount with RBAC

A service account is a non-human account that interacts with the kubernetes
cluster.

Entities outside or inside the cluster (pods, system components, etc) can
use specific credentials to identify as ServiceAccount's.

How to use the service accounts.

- Create a `serviceaccount` object
- Grant permissions to the ServiceAccount object using RBAC
- Assign the ServiceAccount object to a pod during its creation or retrieve
  the SerivceAccount token and use if from an external service.

### How do you assign it to a pod?

Under `spec.serviceAccountName` you add the name of the service account name.

### How do you retrieve it for an external service?

```
k describe pod list-pods -n k97
```
```
k exec -it list-pods -n k97 -- /bin/sh
# cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJShAthetoken...
```

> Note. To get the API server endpoint from within the cluster you can do
> `k get service kuberentes`.

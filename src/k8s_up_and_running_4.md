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


# Pods

<!-- toc -->

Go read more on pods [here](./k8s_up_and_running_4.md)

## Running and config applications in containers with pods


Creating a pod with imperative command (probably we will never do this?)

```bash
$ k run hazelcast \
    --image=hazelcast/hazelcast \
    --restart=Never \
    --port=5701 \
    --env="DNS_DOMAIN=cluster" \
    --labels="app=hazelcast,env=prod" \

```

In `yaml` would look something like this.

```yaml
apiVersion: v1
kind: Pod
metadata: v1
    name: hazelcast
spec:
    containers:
    - image: hazelcast/hazelcast
    - name: hazelcast
      ports
      - containerPort: 5701
    restartPolicy: Never
```

The container runtime engine will download the image from the registry and
store it in that pod.

Pod Life Cycle Phases

```
Pending -> Running -> Succeeded
                  |
                   -> Failed
```

Create a temporary pod, for experimentation, for example here we create a pod
to see if it can communicate with the ip provided.

```bash
k run busybox \
    --image=busybox \
    --rm \ # this will  remove the pod after creation
    -it
    --restart=Never
    -- wget 10.1.0.41
```

We can overwrite the entry point of a container, in the manifest
```yaml
# more stuff
spec:
    containers:
    - image: some/image
      name: spring-boot
      command: ["/bin/sh"]
      args: ["-c", "while true; do date; sleep 10; done"]
```

Here we use `command` and `args` to overwrite the entrypoint.

If you want to delete a pod without graceful deletion, you can do:
```bash
k delete -f pod.yaml --now
```


## Namespace

Groups resources for ease of organize.

Namespaces starting with `kube-` are not consider end user-namespaces, when
you are a developer you wont need to interact with those.

If you delete a namespace, everything that it contains will be deleted as well

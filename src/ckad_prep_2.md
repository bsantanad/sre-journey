# Object Management

Hybrid approach for managing objects.

First you create the yaml automatically with the `run` command
```bash
k run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

Then you can edit it with `vi` or smth,

```bash
vi nginx-pod.yaml
```

And then you actually create the object out of the yaml file:

```bash
k create -f ngnix-pod.yaml
```


# Metrics Server

The metric server is component you have to install in a k8s cluster to retrieve
metrics.

It works by the `kubelet`s sending information (`cpu`, `memory consuption`) to the
centralized server.

To install the metrics server, you can `k apply` the manifest:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Once it is installed you can do
```
$ k top nodes # show metrics on nodes
$ k top pods # show metrics on nodes
```

For the Horizontal Pod Autoscaler (HPA) you need to have this installed in your
cluster. Pod scheduling on clusters also sometimes need the metrics server
installed. For example, when assigning cpu, memory limits.

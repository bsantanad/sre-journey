# Services

It is providing a stable network endpoint to one or many pods.

When a pod restart the IP changes, so we cannot rely on those. There is a k8s
primitive that provides discoverable names and load balancing to a set of Pods.
It's called Service

## Exposing a Network Endpoint to Pods

Request routing, the service uses label selection to see which pods can it use.

We also need to do port mapping. `contianerPort` is the port in the container,
`targetPort` is the port in the service. They need to be the same.

Imperative approach.

```bash
k create service clusterip echoserver --tcp=80:8080
```

Or we can create a pod and a service from one shot.

```bash
k run echoserver --image=some-image --port=8080 --expose
```

This is the manifest approach.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

## Service Types (How can we export the service)

- `ClusterIP`: exposes the service on a cluster-interal ip. Meaning it will
  only be reachable from pods within the cluster (this is the default).
- `NodePort`: Exposes the service on each node's IP at a static port.
- `LoadBalancer`: Exposes the service externally using a cloud provider's load
  balancer.

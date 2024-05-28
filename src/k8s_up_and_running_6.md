# Service Discovery

K8s is a dynamic system. It has to manage placing pods on nodes, make sure they
are running and more stuff. Due to this nature k8s makes it easy to run a
lot of things, but it can be a bit hard to _find_ these things.

## What is Service Discovery?

A service is a method for exposing a network application that is running in one
or more Pods in your cluster

Actually you use one type of service discover everyday, DNS, the thing that
helps you resolve hostnames to IPs. It works great for the internet, but falls
short in the k8s world.

Why? DNS caches a bunch of stuff, and in a k8s cluster pods are ephemeral, they
can be killed and come back up in seconds, so due to this cache we could be
pointing to something that is already dead.

While these pods that compose the backend may change, the frontend clients
should not need to be aware of that, nor should they need to keep track of the
of the backend themselves.

K8s has an object/resource specific to this issue.

## The Service Object

A Service is an object, it has a manifest and all that. Say you have a set of
pods that each listen on port 5000 and are labeled as `name=flask-app`. The
manifest would look something like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
```


K8s will assign the service a Cluster IP , and will be continuously scanning
for Pods that match the selector (the `flask-app` part). Then it will make the
updates to the set of `EndpointSlices` for the Service.

## Using the type `NodePorts`

By using this type, in addition of a Cluster IP, the system picks a port on
every node in the cluster to forward traffic to the service.

Meaning you can connect to any node in the cluster, and reach an specific
service. You can use `NodePort` without knowing where any of the pods for that
service are running.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort <--- here
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```

Now each request that is send to the service will be randomly directed to one
of the pods that implements the service.

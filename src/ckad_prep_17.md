# Resource Management

Resource requests and limits, resource quotas and limit range.

Here we will cover:

- Define min and max resources needed to run a container. We just need to
  modify the definition of a pod.

- `ResourceQouta` constrain resources on a namespace-level.

- `LimitRange` constrains resource alloc for a single object

## Resource Units in K8s

K8s measures: CPU in millicores and memory resource in bytes.

1 CPU unit is equivalent to 1 physical CPU core, or 1 virtual core, depending
on whether the node is a physical host or a virtual machine running inside a
physical machine.

## Container Resource Request (min)

You can define the min amount of resources needed to run an app via
`spec.containers[].resources.requests`

These resources include: cpu, memory, huge page, ephemeral storage

This limits influence if the pod can be scheduled or not, you might see these
messages: `PodExceedsFreeCPU` or `PodExceedsFreeMemory`.

The options available are:

- `spec.containers[].resources.requests`
  - `cpu`: eg 500m
  - `memory`: eg 64Mi
  - `hugepages-<size>`: eg 60Mi
  - `ephemeral-storage`: eg 4Gi

## Container Resource Limits (max)

Can be defined in `spec.containers[].resources.limits`

These resources include: cpu, memory, huge page, ephemeral storage

Container runtime decides how to handle situation where app exceeds alloc
capacity. (Maybe kill the container)

- `spec.containers[].resources.limits`
  - `cpu`: eg 500m
  - `memory`: eg 64Mi
  - `hugepages-<size>`: eg 60Mi
  - `ephemeral-storage`: eg 4Gi

## ResourceQuota

This is a k8s primitive, it defines resources constrains for an specific
namespace. This is usually done by the admin.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
  namespace: this-ns
spec:
  hard:
    pods: "2"
    requests.cpu: "2"
    requests.memory: 1024Mi
    limits.cpu: 4
    limits.memory: 4096Mi
```

If the there is a resource quota, then all the resources it covers must define
`limits` and `requests` so the resource quota can do the math. It wont allow us
to create the objs otherwise.

## LimitRange

So basically `resourcequota` is for namespaces, but you can micro manage it
more with limit ranges you can specify for each object within a namespaces the
resource allocation.

You can apply default values that will be set if no explicit declaration is set
in the specific object manifest. As well as global min and max.

For example this will
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

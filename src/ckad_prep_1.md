# k8s in a nutshell

<aside>
💡 For more in depth explanations, please go to
 [k8s up and running](./k8s_up_and_running.md)
</aside>

## High level k8s arch

A k8s cluster has two kind of nodes

- **control plane node**: exposes the k8s API, so if you want to interact with
  the k8s cluster you need to go thru here.
- **worker node**: nodes that execute the workload. Needs to have a container
  runtime engine installed (containerd) for executing the containers.

### Control Plane Node Components

- API server: expose k8s api
- Scheduler: Watches for new k8s pods and assign to nodes
- Controller Manager: watches the state of cluster, and makes changes
- etcd: a key-value that captures the state of the objects we create in the
  cluster

### Common Node Components

Any node has these:

- Kubelet: agent that makes sure the containers are running in a k8s pod,
  usually this one runs in the workers.

- kube proxy: maintain network rules and allow communication

- Container runtime: software responsible for running containers

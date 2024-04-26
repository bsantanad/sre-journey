# ⛵️ k8s personal notes

- First thing you need to install kubectl in your system
- Then you need to have a `~/.kube/config` file, pointing to the cluster you
  want to interact with
- You can alias `echo 'alias k=kubectl' >>~/.bashrc`

# Namespaces

> The ***Namespace*** resource allows you to divide a Kubernetes cluster into
> several smaller virtual clusters.

```bash
k get ns
```

# Pods

> Pods are the smallest building blocks in Kubernetes …. A Pod is made up
of one or more containers that share network and storage resource

```bash
k get pod -n <some namespace>
```


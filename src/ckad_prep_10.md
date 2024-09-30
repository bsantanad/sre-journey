# Helm

Helm is a package manager (similar to dnf or brew). The artifact produce by
helm is a chart. The chart contains the info needed to create the cluster.

They are stored in a repository. At runtime, it replaces placeholders in `yaml`
templates files with the actual end-user defined values.

## Discovering and Installing a Public Helm Chart

You can go to [artifacthub.io/packages](artifacthub.io/packages)

Search for a chart:
```bash
helm seach hub jenkins
```

We can add repos for helm to look into them:
```bash
helm repo add bitnami https://charts.bitname.com/bitnami
helm repo update # this will give us the latest versions of the charts
```

Once we add the repo we can install the chart with
```bash
helm install jenkins bitnami/jenkins
```

## Listing installed charts

```bash
helm list
```
Then we can actually see the k8s objects/resources that were created by it.
At any point we can choose to uninstall the charts.


We can uninstall this at any time
```bash
helm uninstall jenkins
```
## Building and Installing a Custom Helm Chart

This is not part of the exam, but good to know. We have a `Chart.yaml` and it
has the meta info of the chart. Then we will have the `values.yaml` were we
can define specific values that need to be change at runtime.

We have another dir `templates` there we have all the manifests for the k8s
objects/resources.

You can just render the template `helm temlpate /path/to/the/chart/`.

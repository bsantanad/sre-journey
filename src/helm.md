# ⎈ helm

> ⚠️ These are notes taken from the book [_Learning Helm by Matt Butcher, Matt
> Farina and Josh Dolitsky on
> O'Reilly_](https://learning.oreilly.com/library/view/learning-helm/9781492083641/ch02.html#idm46126000033640)
> **Please go read it and just use this as reference**.
<!-- toc -->


# Introduction

Helm is the package manager for k8s. Let us do a super quick dive into what k8s
is an all that just to give the context necessary to understand helm.

## Context for Helm (containers & k8s)

Sometime ago someone thought of a new way of making apps, instead of them being
a big monolithic service, we could split everything into small discrete
standalone services. Then we could join those services using API over the
network and build the application that way. That is how microservices were
born.

### Containers

To help with this the concept of containers was introduced, it is different of
a VM because the VM runs an entire OS on on a host machine. On the other hand,
a container has its own file system, but it uses the same OS kernel as the
host.

A container is a program together with its dependencies and environment, they
are packaged together into a _container image_ which can be build based on a
file where you specify the packages you want to run, and how to set the
environments. The cool part is that all those instructions are not compiled
into a binary or anything, but they are packaged into discrete layers.

So if a host has an image with five layers, and there is another host that
needs the same image, it will just fetch the layers it does not already have.

Say you have three images using `fedora-minimal:37`, well it would reuse that
layer on all those three.

Those images are stored in a _registry_, the registry tells hosts which layers
compose an image. So the host can only download the layers it need from the
registry.

A registry identifies an image by three things:
- name: basically a string `fedora` or `fedora-minimal`
- tag: usually the version `latest` or `v1`
- digest: a hash of the image, since the tags are mutable.

They look like `name:tag@digest`.

### Kubernetes (or k8s like the cool kids call it)

So with all these container stuff, some questions begin to arise:
- How do we best execute lots of containers?
- How can they work together?
- How do we manage memory, cpu, network and storage?

Well a bunch of companies tried to create this orchestration of containers
technology to answer those questions, at the end as per 2024 seems like Google
won, and we now all use ⛵️ K8s.(which is greek for ship's capitan or something
like that)

K8s won because it introduced two concepts that people really liked.
- **Declarative infrastructure**
    Basically you tell k8s what your desired state of the cluster is, and it
    will work to make that happen.

- **Reconciliation loop**
    How does k8s work behinds the scenes to reach the declarative configuration
    we set? Using a reconciliation loop.

    In a reconciliation loop, the scheduler says: "Here is what the user wrote
    as his/her desired state. Here is the current state. They are not the same,
    lets reconcile them."

    Say you specified storage, and the scheduler sees that we do not have
    storage yet, it will create units of storage and attach them to your
    containers.

### Pods

We do not deal directly with containers when setting up our k8s cluster.
We use a pods. A pod which is basically a group of containers. These are
defined in a manifest (`yaml` or `json`, but most people use `yaml`)

```yaml
apiVersion: v1 # we can see that this will be a v1 Pod
kind: Pod
metadata:
    name: example-pod
spec:
    containers:
    - image: "fedora-minimal:latest"
    - name: example-fedora
```

A Pod can have 1 or more containers. The containers that help preconfigurate
stuff for the main one, are called _init containers_.

The ones that run alongside the main container are called _sidecar containers_

### `ConfigMaps`

Basically in a pod you describe the configuration the containers need, like
network ports files system mount points. You can store configuration
information for k8s in `ConfigMaps` and password and stuff like that in
`Secret`.

Here is an example:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: configuration-data
data: # inside here we will declare aribitrary key/value stuff
    backgroundColor: blue
    title: Learning Helm
```

For `Secret` is pretty much the same as for `ConfigMaps` but the values in data
must be Base64 encoded.

Pods then can be linked to the ConfigMaps like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: example-pod
spec:
    volumes:
    # note that this is not the metadata name you used in the ConfigMap
    - name: my-configuration
      configMap:
        name: configuration-data # same name as in metadata of ConfigMap
    containers:
    - image: "fedora-minimal:latest"
      name: example-fedora
      env:
        - name: TITLE # the env variable will be title
          valueFrom:
            configMapKeyRef:
                name: configuration-data # name for the volume
                key: title # key in the actual ConfigMap
```


### Deployments

Pretty cool stuff, but we might not just one to run one instance of our
container, therefore we can use something called `Deployments`.

A `Deployment` describe an application as a collection of identical pods,we can
tell k8s to create our app with a single pod and scale it up to five pods.

```yaml
apiVersion: apps/v1 # apps/v1 Deployment
kind: Deployment
metadata:
    name: example-deployment
    labels:
        app: my-deployment
spec:
    replicas: 3 # we want three replicas of the following template
    selector:
        matchLabels:
            app: my-deployment
    template:
        metadata:
            labels:
                app: my-deployment
        spec:
           containers :
            -image: "fedora-minimal"
             name: "example-fedora"
```


### Service

A `Service` is a persistent network resource, that persists even if the pod or
pods attached to it go away.

```yaml
apiVersion: v1
kind: Service
metadata:
    name: example-service
spec:
    selector:
      app: my-deployment
    ports:
      -  protocol: TCP
         port: 80
         targetPort: 8080
```

Here we define a service for the pods with the `app: my-deployment`, telling
that traffic port 80 of this `Service` will be routed to 8080.

## Helm's Goal

So we have seen the building block of K8s, say you want to deploy a WordPress
app, you would need a `Deployment` for the containers, then a `ConfigMap` and
`Secret` for the password, maybe some `Service` objects and so on. That sounds
like a lot of `yaml`.

The core idea of Helm  is that all those objects can be packaged to be
`installed`, `updated` and `deleted` together.

Basically helm is the package manger of kuberentes.  Helm will allow you to
spin up all the k8s objects necessary for an app by just using a few commands.

Here are some of the things that helm can do:
- Provide package repos (similar to `dnf` or `apt`).
- Familiar install, upgrade, delete commands.
- Helm has a method for configuring the packages before installing them.
- Helm can also let you know what already is installed.

You can also limit the installation of packages to a specific namespace, so you
can install the same package in different namespaces in the same k8s cluster.
([What is a namespace?](/k8s_up_and_running.html#namespaces))


Helm also provides **reusability**, it uses charts for this. A chart provides a
pattern for producing the same k8s manifests. But the cool part is that you can
also add more configuration to those charts.

Helm provides patterns for storing the initial configuration plus the changes
you did. Helm encourages k8s users to package their `yaml` into charts so that
these descriptors can be reused.

One last thing, keep in mind that helm is not a configuration tool, it helps
but it there are software specialized on that like ansible, puppet or chef.

## Helm Architecture

Here are the main components that helm uses.

### K8s Resource/Object

These are the `Pods`, `ConfigMap`, `Deployment`, we have seen throughout the
chapter. K8s has a lot of these objects, you can even define custom ones using
custom resource definition (CRD).

All the resources share some elements
```yaml
apiVersion: apps/v1 # api and version of the resource
kind: Deployment # resource type
metadata: # top-level data on the resource/object
    name: example-deployment # req for all objects
    labels: # used for creating query-able handles
        some-name: some-value
    annotations: # authors to attach their own keys and values
        some-name: some-value
```


### Charts

A package is called a chart. The idea es that k8s meaning captian, helm being
the steering mechanism of the ship. The chart plots the way k8s apps should be
installed.

A Chart is a set of files and directories that describe how to install the
different k8s resource/objects

A chart contains
- `Chart.yaml`: describes the chart (name, description, authors)
- `templates` directory: Inside all the k8s manifests  potentially annotated
  with templating directives
- `values.yaml`  file that provides the default configuration. You can override
  during installation.

These are the basic stuff for an unpacked chart, a packed chart is just a tar
ball with all this inside.

### Resources, Installations and Releases

When a helm chart is installed:
1. helm reads the charts (will download if necessary)
2. It sends the values into the templates generating the k8s manifests
3. The manifests are sent to k8s
4. K8s creates the requested resources inside the cluster

One last concept _release_. A release is created each time we use helm to
modify the installation


# Using Helm

Helm is a cli, you can install it with your favorite package manger or  build
it from source it is written in golang.

You can check the version by
```
helm version
```

Helm will use the same KUBECONFIG file you have configured for `kubectl`, it
will look in the same places, though you can specify a path for one.

The most common workflow is:
1. Add a chart repo
2. Find a chart to install
3. Install a Helm  chart
4. See the list of what is installed
5. Upgrade your installation
6. Delete your installation

## Adding a Chart Repo

A Helm Chart is an individual package that can be installed into your k8s
cluster. You can find then  at chart repositories.

You can find popular repositories in [the artifact hub](https://artifacthub.io)

By default `helm` does not have any repo added, so you need to look for one
there.

Bitnamis's official Helm charts are one of the best well-curated charts repos.
(Some Bitnami devs are among the core contributors who design the helm repo
system)

To add a repo you do `helm repo add` so:
```bash
% helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

Now if you do
```bash
% helm repo list
NAME    URL
bitnami https://charts.bitnami.com/bitnami
```

You will see it there. After that we can look for specific charts.

```bash
% helm search repo drupal
NAME            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/drupal  18.0.2          10.2.5          Drupal is one of the most versatile open source...
```

You can also search label and descriptions
```bash
% helm search repo content
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/drupal          18.0.2          10.2.5          Drupal is one of the most versatile open source...
bitnami/harbor          21.1.2          2.10.2          Harbor is an open source trusted cloud-native r...
bitnami/nginx           16.0.6          1.25.5          NGINX Open Source is a web server that can be a...
bitnami/owncloud        12.2.11         10.11.0         DEPRECATED ownCloud is an open source content c...
bitnami/wordpress       22.2.2          6.5.2           WordPress is the world's most popular blogging ...
```

The chart version is the version of well, the chart. On the other hand the app
version is the version of the software it would install.

## Installing a package


```bash
% helm install mysite bitnami/drupal
NAME: mysite
LAST DEPLOYED: Wed May  1 16:40:16 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
# more text
```

Typical stuff, just a few things to keep in mind.

- Difference between an installation and a chart.

    An installation of a chart is a specific instance of the chart
    One chart may have many installations

- You can repeat instance names (`mysite`), but it must be on different
  namespaces.


You can set values specific to your installation, you can set them directly
from the command line with `--set`  for example. (this works with both set
`install` and `upgrade`)

```bash
% helm install mysite bitnami/drupal --set drupalUsername=admin
```

You can also have them in a `yaml` file, which is the recommended approach.

```bash
% helm upgrade mysite bitnami/drupal --values values.yaml
```
For example `values.yaml` would look like this:
```yaml
drupalUsername: admin
drupalEmail: admin@example.com
mariadb:
    db:
        name: "my-database"
```

## Listing your Installations

```bash
% helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysite  default         1               2024-05-01 16:40:16.50636 -0600 CST     deployed        drupal-18.0.2   10.2.5
```

Not much to say here, does what expected.

## Upgrading an Installation

So there are two types of changes:
- upgrade version of the chart
- upgrade configuration of the chart

Every time we perform an upgrade we are doing a new release of the same
installation.

Helm will attempt to to alter only the bare minimum, so if you only changes one
simple configuration variable, it will not like restart everything and all
that.

To restart stuff just use `kubectl`.

To update your chart with a new version you can
```bash
% helm repo update
% helm upgrade mysite bitnami/drupal
```

## Configuration Values

If you do this:
```bash
% helm install mysite bitnami/drupal --values values.yaml
% helm upgrade mysite bitnami/drupal
```
Chances are you lost your values, so it is good to always send the `yaml`

```bash
% helm install mysite bitnami/drupal --values values.yaml
% helm upgrade mysite bitnami/drupal --values values.yaml
```

You can use `helm get values mysite` to see the values sent on the last install
or upgrade.

You could also use:
```bash
% helm upgrade mysite bitnami/drupal --reuse-values
```
But it is not recommended

## Uninstalling charts

Not much to say here
```bash
% helm uninstall mysite
```
Works as you would expect, if you would do
```
% helm list
```
You will not see it there.

Lastly, you can see a special record that contain release information.
```bash
% k get secret
```
Helm stores there the info.

If you uninstall you loose the history, so be careful. You could `helm
uninstall --keep-history`. Good if you plan on doing `helm rollback`.

That is pretty much the basics on helm.

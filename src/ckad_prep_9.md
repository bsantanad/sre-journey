# Deployments

Managing a set of pods with the same config (called replicas) we want to make
sure we have the same config for these replicas.

We can scale up or down the number of replicas to fulfill our reqs. Updates to
a replica config can be easily rolled out automatically.

Under the hood, we say
- `deployment`, "create 3 replicas"
- `ReplicaSet`, "maintain stable set of 3 pods"
- `pod`, "3 pods with the same definition"

How do we create them imperatively:
```bash
k create deployment my-deploy --image=nginx --replicas=3
```
The default `--replicas` is 1

The yaml would look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Everything under the `spec.template` is the same as the attributes for the pod
spec.

The deployment internally uses label selector `spec.selector.matchLabels`, it
matches against `spec.template.metadata.labels`.

## Rolling out changes.

If you change the live object of the deployment, it will roll out the changes
of the `replicasets` by themselves.

It will keep a track of the changes, you can check it with:

```bash
$ k rollout history deployment my-deploy
```

If we make a change to the deployment, for example assign a new image.
```bas
k set image deployment my-deploy nginx=nginx:1.19.2
```

If you run the `k rollout history` command again, you will see the new
deployment.

If we want to come back to other versions, we can do
```bash
k rollout undo deployment my-deploy --to-revision=1
```

You can annotate what changed on each revision
```
k annotate deployment my-deploy kubernetes.io/change-cause="image updated to 1.17.1"
```
This will be displayed when listing the revisions.

## Manually Scaling a Deployment

```bash
k scale deployment my-deploy --replicas=5
```
Or we can edit the live object of the deployment.


## Autoscaling a Deployment

The deployments scale automatically based on the metrics k8s is generating.

There are a couple of types of autoscalers (which are k8s objects):

- Horizontal Pod Autoscaler (HPA): standard feature of k8s that scales the
  number of pod replicas based on cpu and memory thresholds.
  (Relevant for the exam)
- Vertical Pod Autoscaler (VPA): scales cpu and memory alloc for existing pods
  based on historic metrics. Supported by a cloud provider as an add-on.

Both of them use the metrics server. You need to install the component, and set
the resource requests and limits.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef: # the scaling target
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 3
  maxReplicas: 5
  metrics:
  - type: Resource # the thresholds
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

You can see them with `k get hpa`

## Deployment Strategies

- ramped: we will touch one replica at a time and update the config needed.
  - pros: leads to no downtime. We roll out the new version to all the replicas
    overtime.
  - cons: make sure we do not introduce breaking changes

- recreate: terminate the old version and then create a new one with the new
  config
  - pros: good for dev envs, everything shutdowns at once and renewed at once
  - cons: can cause downtime

- blue/green: creates a new deployment with the new version, they run along
  side each other, and once we are happy with the new one, we switch traffic.
  (you need to set up two different deployments)
  - pros: no downtime, traffic can be routed when ready
  - cons: resource duplication, config and network routing.

> Special note on this one. When you have the two deployments, you'll need a
> svc that is the one that actually exposes the app, then you can just change
> the label selector there from blue to green (or viceversa) and k8s will do
> the rest for you

- canary: release a new version to a subset of users, then proceed to a full
  roll out (useful for testing) (you need to set up two different deployments)
  - pros: new version released to subset of users.
  - cons: may require a load balancer for fine-grained dist


### How do we implement this?

`under spec.strategy.type` we define the type we want. We can configure it
there


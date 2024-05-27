# Labels and Annotations

K8s apps can grow in size and complexity real quick. We can use _labels_
and _annotations_ to help organize our k8s objects/resources as sets of
things that map how we think about the app. Making it a bit more human
accessible. Meaning we can group resources that make the most sense of the
application.

We have two tools that help with this.
- **Labels**: key/value pairs, to attach identifying information  to the k8s
  obj/resource. You will be able to query based on this.
- **Annotations**: set non-identifying information that libraries can leverage.
  These are not meant for querying

## Labels

Labels give identifying metadata for k8s objects. They are key/value pairs of
strings.

Keys are broken down into two parts
- Optional prefix name, separated by a slash.
- Key name, it must start and end with alphanumeric chars.

Some examples:
* `acme.com/app-version` - `1.0.0`
* `app.version` - `1.0.0`

### Applying Labels

You can do it when doing the `k run` command. (Doubt that is the way you are
using it but anyway):
```bash
k run alpaca-test \
  --image=gcr.io/kuar-demo/kuard-amd64:green \
  --labels="ver=1,app=alpaca,env=test"
```

You can also add it to the manifest of the resource you are using:
```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:
    ver: "1"
    env: "test"
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

After the objects have been created, you can do `--show-labels`

```
k get deployments --show-labels
```

### Modifying Labels

You can update the labels on objects after creating them:
```
k label deployemnts alpaca-test "canary=true"
```
Keep in mind that this will only update the label on the deployment itself, it
wont affect any objects created by the deployment.

In case you feel lazy you can replace `--show-labels` with `-L`.

```
k get deployemnts -L canary
```

To remove a label you add a dash-suffix
```
k label deployments alpaca-test "canary-"
```

### Label Selectors

Okay so, how do we query this stuff? Using boolean expressions

This will list only the pods that have ver label set to 2.
```
k get pods --selector="ver=2"
```

You can get a bit more creative:

This is the AND operator:
```
k get pods --selector="app=bandicoot,ver=2"
```

You can kind of do the OR by:
```
k get pods --selector="app in (alpaca,bandicoot)"
```

You can also check if a label has been set by just using the key name:
```
k get deployments --selector="canary"
```

Check if  key is not in `value1` or `value2`:
```
k get pods --selector="key notin (value1, value2)"
```

You can combine them
```
k get pods --selector="ver=2,!canary"
```

### Labels Selector in Manifests

Newer versions support doing something like this:
```
selector:
  matchLabels:
    app: alpaca
  matchExpressions:
    - {key: ver, operator: In, values: [1, 2]}
```

## Annotations

This are only meant to assist tools and libraries, do not use them for queries.

If you are wondering if something should be a label or an annotations, add
information to an object as an annotation and promote it to a label if you find
yourself wanting to use it in a selector.

Usually annotations primary use is rolling deployments, they are used to track
rollout status and provide the information for a roll back if needed.

Annotations are defined in the `metadata` part of the  manifest:
```
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

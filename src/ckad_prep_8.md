# Labels and Annotations

<!-- toc -->

Labels:
- Key value pairs that can be assigned to an object
- They need to follow a naming convention, we can use them to filter objects

Annotations:
- Represent human-readable metadata
- Not meant for query

## Labels

We can assign it with the imperative `label` command, or with
`metadata.labels[]`.

Not meant for elaborate. They have a max of 63 characters.

We can add a label imperatively with:
```bash
k label pod nginx tier=backend env=prod app=viraccle
```
See pods with labels
```bash
k get pods --show-labels
```
To remove a label you put the key and a minus sign
```bash
k label pod nginx tier-
```

From a manifest.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
```

We can use them to query stuff.
```bash
$ k get pods -l tier=frontend,env=dev --show-labels
$ k get pods -l version --show-labels # only the key
$ k get pods -l 'tier in (frontend,backend),env=dev' # create a query
```

There is other k8s objects that enforce policies or smth, and that can also use
the labels to select to which pod you want to enforce them.  For example a
`NetworkPolicy` can have `spec.podSelector.matchLabels.mylabel: frontend`.

## Annotation

Metadata without the ability to be queryable. Make sure to put the annotation
in quotes, for handling spaces and stuff.

A example could be the author, a commit hash, or a branch.
```bash
$ k annotate pod nginx  commit='866a8dc' branch='users/jloca/bug'
```

If we `describe` the object we will see our annotations there.

There are some well-known annotations, `pod-security.kubernetes.io/warn: baseline`
but that goes out of the scope of the course.

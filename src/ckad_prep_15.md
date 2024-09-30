# Custom Resource Definition (CRDs)

These are custom primitives we can introduce by extending the k8s api.

We can interact with an object that we can define, create and persist, and we
the k8s api will expose it.

You need to create a `controller` to make them useful. This implements the
logic   between your custom object and the k8s api. This combination is called
_Operator pattern_.

The actual controller are written in Go or Python.

## Example CRD

- Requirement: a web app stack deployed via a Deployment with one or more
  replicas. There is a Service object that routes network traffic to the pods.

- Desired Functionality: After the deployment happened, we want to run a quick
  smoke test against Service's DNS names, to see everything is okay. The result
  of the test will be send to an external service for rendering a UI.

- Goal: Implement the Operator pattern (CRD and controller)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```

## Useful tips

Once you create it, you can see it if you list the api-resources
```
k api-resources
```
Then when you create an object you can list it with the `spec.names.singular`
or `spec.names.plural` you defined in the CRD's manifest. So if the name was
`my-custom-obj` you could do:

```
k get my-custom-obj
```

One last thing, to know what is the version you should put in the object manifest you can do.
```
k api-resources | grep my-custom-obj
```
And it will list it there in the second column.

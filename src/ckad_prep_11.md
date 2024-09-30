# API Deprecations

3 times a year k8s has a new release, this means that some of the APIs we are
using might be deprecated. If this is the case, we will see a warning when
doing the `kubectl` commands.

If you need to migrate to new APIs, you can search for the deprecated API
migration guide.

You can see the available versions in your cluster using:
```bash
k api-versions
```
Be sure to check what other changes you need to do to adapt the k8s
resources, sometimes you need to change something inside the manifest other
than the version.

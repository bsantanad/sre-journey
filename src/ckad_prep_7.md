# Multi-Container Pods

In general we want to define 1 container per pod. That is the case when we want
to operate a microservice on that container.

But there are reasons to run multiple ones. For example, helper containers, run
setup scripts, stuff like that.

## Design Patterns

Emerged from pod requirements

- Init Container: initialization logic, that need to be run before the main app
  starts.
  - Example: Downloading config files required by the application.

- Sidecar: containers not part of the main traffic the app receives, but will
  run along side the main app
  - Example: Watcher capabilities.

- Adapter: Transform output produced by app into another format or smth that
  makes it more usable for another program.
  - Example: Massaging log data

- Ambassador: Provides a proxy for communicating with external services. To
  abstract complexity.
  - Example: providing credentials for auth.

### Init Container

To define an `initContainer` you need to add this to your `yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  initContainers:
  - name: app-init
    image: busybox:1.28
    command: ['sh', '-c', "wget something.com/res.png"]
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

When we start the pod, under `STATUS` you can see `Init:0/1` meaning it is
creating the containers needed.

We can talk with any of the containers using `--contianer` or `-c`.
```bash
k --container=init
k --container=app
```

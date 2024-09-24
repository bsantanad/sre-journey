# Container Storage

<!-- toc -->

## Volumes

The containers have a temporary file system, if you restart the container the
file system will be deleted.

- Ephemeral Volumes: exist for the lifespan of a pod. Useful for sharing data
  between multiple containers running in a pod
- Persistent Volumes: preserve data beyond the lifespan of a pod.

Define a volume using `spec.volumes[]` and then reference it in a container
with `spec.containers[].volume.volumeMounts`

### Volume Types

There are a lot, some are only offered by certain cloud providers.

Some of the common ones:
- `emptyDir`: Empty dir in a pod with read/write access. Ephemeral.
- `hostPath`: Point to a file in the host.
- `configMap`: Mount config data
- `nfs`: provide a network file system. Persistent.

### Ephemeral Volume

We need to define the volume first in `spec.volumes` and then reference it
inside `spec.containers.volumeMounts`

Here is an example:

```yaml
apiVersion: v1
kind: Pod
metadata: Pod
    name: my-container
spec:
    volumes:
    - name: logs-volume
      emptyDir: {}
    containers:
    - image: nginx
      name: my-container
      volumeMounts:
      - mounthPath: /var/logs
        name: logs-volume
```

### Persistent Volume

We need to create two more objects to create this type of volume.

- `PersistentVolume` (PV): piece of storage in the cluster provisioned by an admin
- `PersistentVolumeClaim (PVC)`: consumes PV, similar to how a pod consumes
  node resources (cpu, memory) here the `pvc` consumes storage from a `pv`

There are different ways on how we can provision this type of storage.

- static provisioning: we create the `PersistentVolume` object by ourselves.
- dynamic provisioning: automatically creates the `PersistentVolume` object
- storage class: obj/s that already exist in the k8s cluster. (setup by admin)

Defining a `PersistentVolume`

We need to set _capacity_, _access mode_, and _host path_

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/db
```

#### Access Modes:

- `ReadWriteOnce`: rw by single node
- `ReadOnlyMany`: r by many nodes
- `ReadWriteMany`: rw by many nodes
- `ReadWriteOncePod`: rw mounted by a single pod

#### Reclaim Policy:

- `Retain`: default. When PVC (persisted volume claim) is deleted the pv is
  released and can be reclaimed.
- `Delete`:  Deletion removes PV and associated storage.

### How to use it.

So once we have our `pv`, we need to create a `pvc` to claim storage from that
`pv`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: "" # empty means we will use a statically-created pv
```

When we create it we need to check that the state is `Bound`.

In the pod, we create the volume as regular and assign it to the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: db-pvc # reference the volume by pvc name
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/mnt/data"
        name: app-storage
```

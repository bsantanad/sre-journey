# ConfigMaps and Secrets

A good practice is to make your container images reusable, making a general
purpose one that can be used across applications. But we might also need to add
some configuration to the image at runtime. Here is where ConfigMaps and
Secrets come into play.

<!-- toc -->

## ConfigMaps

In a ConfigMap you can define env variables, command line arguments or even
small file systems for your containers. The `configmap` is combined with the Pod
right before it is run.

### Creating ConfigMaps

We can create a `configmap` out of a file. Say for example we have a file
called `config.txt`:
```
parameter1 = value1
parameter2 = value2
```

You can create a `configmap` by doing:
```bash
% k create configmap config --from-file=config.txt
```

Or you can create literally from the command line
```bash
% k create configmap other-config --from-literal=some-param=some-value
```

Now you can list them
```bash
% k get cm
NAME               DATA   AGE
config             1      35s
some-config        1      2s
```
And get them as `yaml`s
```
% k get cm config -o yaml
apiVersion: v1
data:
  config.txt: |
    param1 = value1
    param2 = value2
kind: ConfigMap
metadata:
  creationTimestamp: "2024-06-11T00:58:10Z"
  name: config
  namespace: default
  resourceVersion: "2420"
  uid: 7df23db6-7d2c-4db4-9b22-5fecd4bb456e
```


At the end of the day, the `configmaps` are just some key/value pairs stores in
an object.


### Using a ConfigMap

There are 3 ways of using them:
- File system:
  - A ConfigMap can be mounted in a Pod. A file is created for each key, and
    the content is their respective values
- Command-line Argument:
  - Creating command lines dynamically
- Environment Variable:
  - It can set the value of an environment variable

Let's see each as an example

* File System Volumes
```yaml
apiVersion: v1
kind: Pod
metadata
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kaurd-amd64:blue
      imagePullPolicy: Always
      command:
        - "/kuard"
    volumeMounts:
    # Mounting the ConfigMap as a set of files
    - name: config-volume # <- HERE
      mountPath: /config
  volumes:
    - name: config-volume # <- HERE
      configMap:
        name: config # <- name of the cm we just created
  restartPolicy: Never
```

* Command-line Arguments
```yaml
apiVersion: v1
kind: Pod
metadata
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kaurd-amd64:blue
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)" <- NOTE HERE
      env:
        - name: EXTRA_PARAM
            valueFrom:
            configMapKeyRef
            name: config # <- name of the cm we just created
            key: param1
```
> **Note** that k8s will perform the correct substitution with a special
> `$(<env-var-name> )` syntax

* Environment Variable:
```yaml
apiVersion: v1
kind: Pod
metadata
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kaurd-amd64:blue
      imagePullPolicy: Always
      command:
        - "/kuard"
      env:
        - name: ENV_PARAM
            valueFrom:
            configMapKeyRef
            name: config # <- name of the cm we just created
            key: param2
```

## Secrets

This are pretty similar to ConfigMaps but they are intended for sensitive
information, for example, passwords, tokens, private keys, TLS certs.

This secrets are exposed to Pods via explicit declaration in the Pod manifest
and the k8s API.

> By default k8s stores secrets in plain text on `etcd` storage. If you have
> super sensitive information there, and a lot of people with admin access,
> might be worth encrypting it a bit more.

### Creating Secrets

We can create a secret like any other resource

```
% k create secret generic my-secret --from-file=some.key
```
Instead of `generic` -- depending on the secret -- we can use different types

From the manpage
```bash
Available Commands:
  docker-registry   Create a secret for use with a Docker registry
  generic           Create a secret from a local file, directory, or literal value
  tls               Create a TLS secret
```

If we describe it

```
% k describe secret my-secret
Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
some.key:  1984 bytes
```

The Pods consume the secrets.


### Consuming Secrets

We can call the API directly, but more often we will access secrets from a
_Secrets Volume_. These volumes are created at pod creation time, and are
stored in `tmpfs` (meaning memory not disk).

Each secret is in a different file under the target mount point. So if we mount
the `my-secret` secret in the `/keys` directory, it would looks something like
`/keys/some.key`.

The Pod manifest would end up looking something like this:
```yaml
apiVersion: v1
kind: Pod
metadata
  name: kuard-secret
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kaurd-amd64:blue
      imagePullPolicy: Always
    volumeMounts:
    - name: keys
      mountPath: /keys
      readOnly: true
  volumes:
    - name: keys
      secret:
        secretName: my-secret # <- name of the secret we just created
```

#### Private Container Registries

You can also create a secret for for use with a Docker registry. Instead of
`generic` you would use `docker-registry`, with `--docker-username`,
`--docker-password` and `--docker-email` and the manifest would look something
like this:

```yaml
apiVersion: v1
kind: Pod
metadata
  name: kuard-secret
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kaurd-amd64:blue
      imagePullPolicy: Always
    volumeMounts:
    - name: keys
      mountPath: /keys
      readOnly: true
  imagePullSecrets:
  - name: my-image-pull-secrets-from-registry # <- name of that secret
  volumes:
    - name: keys
      secret:
        secretName: my-secret
```

### Naming Constrains

Key names need to be valid variable names. Just use _normal_ names and you will
be fine.

- valid
  - `.token`
  - `secret.token`
  - `config_file`

- not valid
  - `token..key`
  - `auth token.key`
  - `_token`

Just not use spaces, double dots and stuff like that.


### Managing ConfigMaps and Secrets

The usual stuff, `get`, `delete`, `create`. Just a note on creating:
- `--from-file=<filename>`: keys in file will be keys in secret
- `--from-file=<key>=<filename>`: keys in file will be keys in secret
  - Instead of:
      ```
        Data
        ====
        <filename>:
        ----
        param1 = value1
        param2 = value2
      ```
      You would get
      ```
        Data
        ====
        <key>:
        ----
        param1 = value1
        param2 = value2
      ```
- `--from-file=<directory>`: loads all files in a directory

Also you can recreate and update like:
```
kubectl create secret generic kuard-tls \
  --from-file=kuard.crt --from-file=kuard.key \
  --dry-run -o yaml | kubectl replace -f -
```

Neat trick from the book [_Kubernetes Up and Running by Brendan Burns, Joe
Beda, and Kelsey Hightower
O'Reilly_](https://learning.oreilly.com/library/view/kubernetes-up-and/9781098110192).

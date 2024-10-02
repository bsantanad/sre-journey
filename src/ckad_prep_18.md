# Configmaps and Secrets

K8s primitives for configuration data that we can inject into a pod.

They have some characteristics:

- stores as key pair values.
- stored decoupled from consuming pod
- configmap: plain text values for config apps, flags, URLs, etc.
- secret: Base64-enconded values, for API keys or SSL certificates. The values
  are not encrypted.
- they are stored in `etcd` unencrypted, but you can encrypt them if you want.

Once we create the object we can mount is as a volume or as env variables.

## Imperative Approach

```bash
k create configmap db-config --from-literal=db=staging
```
you have multiple options:
```bash
--from-env-file=config.env
--from-file=app-config.json
--from-file=some-dir # a dir
```

Once you create it, you can  mount it in a pod like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
```

You can also get specific keys from a configmap using `configMapKeyRef`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      env:
      - name: DATABASE_URL
        valueFrom:
            configMapKeyRef:
              name: backend-config
              key: database_url
```

In this example we are using the key `database_url` from the config map
`backend-config` and setting it in the env as `DATABASE_URL` in caps.

## Consuming Changed ConfigMap Data

Containers will not refresh data upon a change. You would have to develop that
on your app, like refresh the env variables periodically or on-demand.

## Secrets

There are these secrets:
- `generic`, creates a secret from a file, directory or literal value
- `docker-registry`, creates a secret for a docker registry
- `tls`, creates a tls secret

Imperative commands
```
k create secret generic db-creds --from-literal=pwd=s3cre
```

Also we have specialized secret types, they are set using the `--type`
attribute in manifest.

- `kubernets.io/basic-auth`, credentials for basic auth
- `kubernets.io/ssh-auth`, credentials for ssh
- `kubernets.io/service-account-token`, serviceaccount token
- `kubernets.io/token`, node bootstrap token data

If you create a secret yaml by hand you need to encode the value yourself.
```bash
echo -n 's3cre!' | base64
```
If you don not want to do that you can also provide the data `stringData` in
the `yaml`, and k will encrypt it once you `k apply` it.




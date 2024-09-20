# Container Images

<!-- toc -->

## Image and Container Management

Glossary.
- Container: package and app into a single unit
- Container Runtime Engine: sw that executes your container (docker engine)
- Container Orchestrator: automates and mages workload  (k8s)
- ContainerFile: Instructions to build the container
- Container Image: packages app into a single unit of sw including its runtime,
  env and config
- Container Registry: where to share your container

Render local images you have in your machine.
```
docker images
```

Before pushing a docker container image to a registry we need to conform to the
conventions of the registry

For example, for docker registry you need to prepend your container image with
your username:
```
docker tag python-hello-world:1.0.0 jose/python-hello-world:1.0.0
```

If you want to push your container image, you need to auth first,
```
docker login --username=jose
```
And then push it
```
docker push jose/python-hello-world:1.0.0
```

You can create a backup of container image into a `.tar` file
```bash
docker save -o python-hello-world.tar jose/python-hello-world:1.0.0
```
Then list it
```bash
$ ls
python-hello-world.tar
```

We can do the inverse from a file:
```bash
docker load --input python-hello-world.tar
```

# Learnings from the Lab

Create the container

`cd` into the dir with the Dockerfile, then do:
```bash
docker build -t some-tag:1.0.0 .
```
The dot is important

You can double check if you did it right with
```bash
$ docker images
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
nodejs-hello-world   1.0.0     3af6d05d3c35   8 minutes ago   180MB
```

Then to actually run the thing,
```bash
docker run -d -p 80:3000 nodejs-hello-world:1.0.0
```
Keep in mind that all the flags go before the image tag, if not it will trip

To list the running containers you can do
```
$ docker container ls
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                                   NAMES
c02c36ecbc85   nodejs-hello-world:1.0.0   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:80->3000/tcp, :::80->3000/tcp   angry_mestorf
```
To stop it you can do
```
docker container stop  angry_mestorf
```
Saving to a tar file:
```
docker save -o nodejs-hello-world-1.0.0.tar nodejs-hello-world:1.0.0
```

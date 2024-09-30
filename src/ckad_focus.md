# What to focus on each topic.

<!-- toc -->

## Containers
- Practice with docker engine
- Understand the most important instructions used in a dockerfile
- Know hot to build a container image from a dockerfile
- Learn how to save a container image to a file and how to load a container
  from a file

## Pods and Namespaces

- Practice command for creating, edit, inspecting and interacting with a Pod
- `k run` allows for fast creation of pod
- Understand diff life cycles to be able to quickly diagnose error conditions


## Cronjob and jobs

- The actual job is executed in pods.
- Understand the different operational types (parallel, completion) for jobs
  can be tricky.
- Force yourself into setting up all possible scenarios and inspect their
  runtime behaviour.
- Know how to configure and inspect the retained job history.

## Volumes

- Understand different use cases for wanting to use ephemeral or persistent.
- Practice the most common volume types. (`emptyDir`, `hostPath`).
- Go to the process of dynamic and static binding.
  - static, you need to create the pv
  - dynamic, automatically created the pv


## Multi-Container Pods

- The attributes that can be assigned to an `initContainers` section are the
  same as the containers section
- Design patterns (Sidecar, Ambassador, etc). When to use and how to implement.

## Labels and Annotations

- Some primitives, Deployment, Service and Network Policy, use label selection
  heavily.
- Annotations are not made for querying, some reserved annotations may
  influence runtime behaviour.

## Deployments

- When creating a deployment make sure label selection match with the pod
  template.
- Practice how to scale the number of replicas manually `--replicas` or via
  `hpa` for thresholds.
- be aware of the `k rollout undo`.
- know how to apply the deployment strategies.

## Helm

- Just know that helm is an open source tool for installing a set of `yaml`
  manifests.
- Practice discovering and installing existing charts, using the helm
  executable.

## API Deprecations

- `kubectl` will show a message if an api will be deprecated
- Check the migration guide

## Probes

- Understand the purpose of readiness, startup and liveness probes.
- Easy to just copy-paste the k8s docs
- Try to induce failure conditions to see the runtime effects

## Metrics Server

- You wont have to install the metric server
- Understand the purpose of the metric server, and which other objects use it.

## Troubleshooting

- practice relevant `k` command to diagnose issues.
- proactively expose yourself to failing pods

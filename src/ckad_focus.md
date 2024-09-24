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

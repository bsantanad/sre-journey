# Jobs and CronJobs

<!-- toc -->

One-time operations and scheduled operations.

Here is the difference between these k8s primitives.
- Pod: we use this for __continuous__ operation, for example a web app that has
  an API on it.
- CronJob: a job we run periodically, for example: running a database backup
- Job: runs only one time, for example, import/export data processes.

## Job

We define a number of completions of a job and the actual work wont be consider
to be comleted until we reach that number of completions.

The work is managed by a `Job` but it will still run inside a Pod

K8s will not delete these objects once they finish, this helps for debugging
purposes.

To create a job imperative:
```bash
$ k create job \
    counter \
    --image=ngnix:1.24
    -- /bin/sh -c 'counter=0; while [ $counter -lt 3 ]; do \
       counter=$((counter+1)); echo "$counter"; sleep 3; done'
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
    name: counter
spec:
    completions: 1
    parallelism: 1 # do we want this to be exectued in parallel
    backoffLimit: 6 # if fails n times, mark it as failed
    template:
        spec:
            restartPolicy: OnFailure # restart pod when fail or start a new pod
            contianers:
            - args:
                - /bin/sh
                - -c
                - ...
                image: nginx:1.24.0
                name: counter

```

There are different operation types for a job. The default behaviour is to run
in a single pod and expect one successful operation. But we can:

- `spec.completions` change the number of times we want to execute it
- `spec.parallelism`  executing a workload by multiple pods in parallel
- `spec.backoffLimit` number of retries until the job is marked as successful
- `spec.template.spec.restartPolicy` need to be declared explicitly, can only
  be `OnFailure` or `Never`
- `spec.activeDeadlineSeconds` if the job is not completed in this amount of
  time, terminate it. This takes precedence over `backoffLimit`

You can check the events of a job either from the description or with this:

```bash
k events --for job/printer -n batch
```

## CronJob

It is a primitive for executing workloads periodically. It uses the same
notation as unix for repetition.

The cronjob will create `Jobs`, so we can do `k get jobs` and see the ones that
the `cronjob` has created

We can retain history for the jobs. The default for the successful ones is
(`spec.successfullJobsHistoryLimit`) set to 3.

The default for the failed ones (`spec.failedJobsHistoryLimit`) is 1


Imperative command to create a cronjob.
```bash
k create cronjob current-date \
    --schedule="* * * * *"
    --image=nginx:1.24.0.0
    -- /bin/sh -c 'echo "Current date: $(date)"'
```


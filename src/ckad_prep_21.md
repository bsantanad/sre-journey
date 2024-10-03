# Troubleshooting Services

We will see how to root cause failures for services.

Try to list the service, and see the service type. Then make a call to the
service (using an ephemeral pod).

There is a command `get endpoints`. It will render all the virtual ip of pods a
service should be able to route.

If we see `<none>` we have issues with connecting the pod.

2 sources of misconfiguration:
* label selector,
* `targetport` does not match with the container port.

Check the network policies. `k get networkpolicies`, make sure it says none

Make sure the pods are running.


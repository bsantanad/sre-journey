# Network Policies

By default all pods can communicate with each other. Network policies give us a
declarative approach to configure which pods can communicate to each other.

It can be as broad as, this namespace can talk to this other namespace, or as
specific as only the traffic on this port should enforce this policy.

## Anatomy of a NetworkPolicy

The manifest basically has two parts:

1. Target pods: Which pods should have the policy enforced, selected by labels.
2. Rules: Which pods can connect to the target pods.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: bookstore
      role: api
  ingress:
  - from:
      - podSelector:
          matchLabels:
            app: bookstore
  - from:
      - podSelector:
          matchLabels:
            app: inventory
```

## Gotcha's

- Empty selector will match everything, eg `spec.podSelector: {}` will apply
  the policy to all pods in the current namespace.
- Selector can only select pods that are in the same namespace.
- All traffic is allowed until a policy is applied.
- There are no deny rules in NetworkPolicies. NetworkPolicies are deny by
  default. Meaning "If you are not on the list you can not get in."
- If a NetworkPolicy matches a pod but has a null rule, all traffic is blocked.

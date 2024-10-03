# Ingresses

> Basically it routes to which service it should go based on the endpoint.

It simply routes traffic to the svc we have in our cluster, we expose them with
http. It is NOT a service type and should not be confused with `LoadBalancer`.

Defines rules for mapping URL context path to one or many service objects.

Not TLS by default.

The ingress cannot work without an Ingress Controller. We can have multiple
ingress controllers in a cluster.

## Imperative approach

```bash
k create ingress corellian --rule="start-alliance.com/corellian/api=my-service-corellian:8080"
```
It looks better on `yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mydomain
spec:
  rules:
  - host: "mydomain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/end/point"
        backend:
          service:
            name: my-service
            port:
              number: 80
```

See the definition of:
- host name (optional),
- list of paths, two types:
  - exact: `/end/point`, will match `/end/point` but no `/end/endpoint/`
  - prefix: `/end/point`, will match `/end/point` and `/end/endpoint/`
- backend.

## Configuring DNS for an Ingress

To resolve the Ingress, you'll need to config DNS entries to the external
address. There is an add-on that help you mange those DNS records called
ExternalDNS.

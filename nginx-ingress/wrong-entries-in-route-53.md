# Wrong entries in route 53

With the help of external-dns route 53 entries are created for assigned load balancers to Ingress. The created route 53 must create an A record with alias and value of assigned load balancer to work. If it is assigning ips or creating some other record the nginx-ingress will not work. We need to provie these args for making sure it creates an A record with alias.

```
    --publish-service = <namespace>/<service_name>
```

Here namespace is the namespace in which our nginx-ingress is deployed, and service_name is the actual name of nginx-ingress's created service.
Given a pod has correct RBAC then following can be used to curl the api and access stuff:

```
$ CA_CERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
$ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
$ NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
$ curl --cacert $CA_CERT -H "Authorization: Bearer $TOKEN" "curl -k https://100.244.0.1:443/api/v1/persistentvolumes"
```

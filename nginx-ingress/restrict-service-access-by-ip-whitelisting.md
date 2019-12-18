# Restrict access to services by whitelisting IP(s)

At times there are specific endpoints or even sites that we want to hide from the public eye. In that scenario ip whitelisting
can help us deny any public request and allows only a few **whitelisted** ips to access the service.
This functionality is achieved using `allow` and `deny` directives in nginx. In `nginx.conf` the following block allow us to 
implement this, restrict it to the hostname/baseurl: 

# For Nginx:

```
location / {
    allow 192.168.1.0;
    deny  all;
}
```

we can even apply this on specific sub urls or patterns, in the following examples this will block all external access to urls
that `contain admin` or `super-admin` for example, www.example.com/admin/test-url?params=doesntmatter

```
location ~ ^/(admin|super-admin)/ {
    allow 192.168.1.0;
    deny  all;
}
```


# For nginx-ingress controller: 

## Pre-requisite:

The value of `externalTrafficPolicy` should be set to `Local` instead of the default value of `Cluster`. With value `Cluster`
the ingress controller will recieve an interal IP(cluster ip) against client requests while setting it to `Local` ensures
that the actual client ip is being passed on along with the request.

Either deploy it using:
`helm install stable/nginx-ingress --set controller.service.externalTrafficPolicy=Local`

Or use a manifest(recommended) for this:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: control
spec:
  releaseName: nginx-ingress
  chart:
    repository: https://kubernetes-charts.storage.googleapis.com 
    name: nginx-ingress
    version: 1.6.11
  values:
      service:
        #Set external traffic policy to: "Local" to preserve source IP on providers supporting it
        externalTrafficPolicy: "Local"
      config:
        use-proxy-protocol: "false"
        enable-opentracing: "false"
```

## For all the traffic going through nginx ingress controller:

We can apply whitelisting on the controller directly using `http-snippet`. Modify the manifest like this:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: control
spec:
  releaseName: nginx-ingress
  chart:
    repository: https://kubernetes-charts.storage.googleapis.com 
    name: nginx-ingress
    version: 1.6.11
  values:
      service:
        #Set external traffic policy to: "Local" to preserve source IP on providers supporting it
        externalTrafficPolicy: "Local"
      config:
        use-proxy-protocol: "false"
        enable-opentracing: "false"
      extraArgs:
        configmap: control/nginx-configuration
        annotations-prefix: ingress.kubernetes.io
```

and use this configmap:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: control
data:
  http-snippet: "geo $the_real_ip $deny_access {default 1; 192.168.1.0 0;}"
  server-snippet: "if ($deny_access) { return 403; }"
```

## For a specific ingress(using ingress):

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: whitelist-login-page
  annotations:
    kubernetes.io/ingress.class: nginx-ingress
    ingress.kubernetes.io/whitelist-source-range: 192.168.1.0
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: service-name
              servicePort: 80
```

You can update value of `path` to apply policies on specific sub urls. 
**Con: You'll have to create multiple redudant ingresses for diffrent sub URLs**
 

## For an ingress using configuration snippet(didn't work properly):

Use the following annotation on your ingress for adding a location block in server configuration block.

`ingress.kubernetes.io/configuration-snippet: location ~ ^/(admin|super-admin)/ { allow 192.168.1.0; deny all;}`

This actually is the right way of doing this kind of whitelisting but it didn't work properly. It returns 404 in our case 
and that is because the way nginx ingress controller automatically generates the configuration. It dervies the value of 
`proxy_pass` at run time which causes conflict in routing and we get a 404 in case of a valid IP.



## Useful links:

https://github.com/kubernetes/ingress-nginx/blob/87d1b8bbf28265386b339e65d8943d8c3f8582ed/docs/user-guide/nginx-configuration/annotations.md#whitelist-source-range

https://github.com/kubernetes/ingress-nginx/issues/2257





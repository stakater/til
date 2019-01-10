# Dashboard issues with nginx ingress

Kubernetes dashboard exposes 2 ports by default

* 9090 (Insecure port)
* 8443 (Secure port)

For using secure port we need to explicitely provide certificates, otherwise the dashboad will not start, and TLS handshake will fail. In our cluster we are using Insecure port for dashboard pod, and our ingress gets secured as we have added an AWS certificate for the domain.
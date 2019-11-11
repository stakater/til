# Exposing service using istio

Istio by definition is `An open platform to connect, manage, and secure microservices`. It comprises of the following 
components:

- **Envoy:** Sidecar proxies to manage traffic from and to the service
- **Pilot:** Responsible for service discovery and configuring envoy(s)
- **Mixer:** Centralized component responsible for providing/managing Istio-policy(policy control) and 
         istio-telemetry(telemetry collection) for usage policies and gathering telemetry data, it enforces policies on the 
         envoys
- **Gateway:** Serves as an ingress port for external traffic
- **Citadel:** Responsible for key and certificate management
- **Galley:** Central component for validating, ingesting, aggregating, transforming and distributing config within Istio.

Now that we have the basic knowledge regarding istio, lets move forward with deploying istio. We'll use helm charts for the
deployment:

## 1. Create namespace: 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  namespace: istio-system
  labels:
    name: istio-system
```
## 2. Install CRDs:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: istio-init
  namespace: istio-system
spec:
  releaseName: istio-init
  chart:
      repository: https://storage.googleapis.com/istio-release/releases/1.3.4/charts/
    name: istio-init
    version: 1.3.4
```
## 3. Install Istio:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: istio
  namespace: istio-system
spec:
  releaseName: istio
  chart:
    repository: https://storage.googleapis.com/istio-release/releases/1.3.4/charts/
    name: istio
    version: 1.3.4
  values:
    global:
      enableTracing: true
    mixer:
      telemetry:
        resources:
          requests:
            cpu: 100m
            memory: 500m
    sidecarInjectorWebhook:
      enabled: true
    pilot:
      traceSampling: 100
    prometheus:
      enabled: true
    tracing:
      enabled: true
```

# 4. Once, istio is done setting up it's required ecosystem. Lets deploy a sample application(nodemart front-end) and expose
# it to external traffic using istio:

Deploy the following manifest or any app of your choice, the key being annotating it with `sidecar.istio.io/inject: "true"`.
Istio will recognize the service and deploy a side-car container, envoy proxy side-car, with our application container.
This side-car will be responsible for managing incoming and outgoing traffic through the application container:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: nordmart-dev-web
  namespace: nordmart-dev-apps
  annotations:
    sidecar.istio.io/inject: "true"
spec:
  releaseName: nordmart-dev-web
  chart:
    repository: https://stakater.github.io/stakater-charts/
    name: application
    version: 0.0.10
  values:
    applicationName: "nodemart-web"
    deployment:
      podLabels:
        app: web
      image:
        repository: docker-delivery.workshop.stakater.com:443/stakater-lab/web
        tag: v0.0.1
      imagePullSecrets: "<concealed>"
      probes:
        readinessProbe:
          failureThreshold: 3
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
          initialDelaySeconds: 10
          httpGet:
            path: /health
            port: 4200
        livenessProbe:
          failureThreshold: 3
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
          initialDelaySeconds: 10
          httpGet:
            path: /health
            port: 4200
      volumes: {}
      env:
      - name: PORT
        value: "4200"
    service:
      ports:
      - port: 4200
        name: web
        protocol: TCP
        targetPort: 4200
    rbac:
      create: false
      serviceAccount:
        name: default
    configMap:
      enabled: false
```
By going through the manifest we can see that it opens it's port 4200 for internal(within the cluster) communication via a
service. For external traffic influx to this service `nodemart-web` at port `4200` we need two things; a **gateway** for
allowing incoming traffic to the cluster based on port, protocol and certificates, a **virutalservice** that specifies
the routing information to find the correct service.

## 1. Install certificates:
Generate required certificate and place the appropriate values using the following manifest: 

```yaml
apiVersion: v1
data:
  key: "<concealed>"
  cert: "<concealed>"
kind: Secret
metadata:
  name: istio-ingressgateway-certs
  namespace: istio-system
type: tls
``` 

## 2. Create Gateway:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-gateway
  namespace: istio-shared
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      tls:
        mode: SIMPLE
        serverCertificate: /etc/istio/ingressgateway-certs/tls.crt
        privateKey: /etc/istio/ingressgateway-certs/tls.key
      hosts:
        - "web.hostname.com"
    - port:
        number: 80
        name: http
        protocol: HTTP
      tls:
        httpsRedirect: true
      hosts:
        - "web.hostname.com"
```
Let's break this manifest down, we created an Object of type `Gateway` in `istio-shared` namespace, why we used a 
seperate/shared will be explained later. To use `istio ingressgateway service` we specify `selector.istio: ingressgateway`.
This gateway will accept traffic for host `web.hostname.com` at port 443 and redirect all incoming traffic on port `80`(http)
to 443, implementing ssl redirect. In `tls` blocking we are using the certificates we installed in the previous step.

## 3. Create virtualservice:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-virtualservice
  namespace: istio-shared
spec:
  hosts:
    - "web.hostname.com"
  gateways:
    - istio-gateway
  http:
    - route:
        - destination:
            host: nodemart-web.nordmart-dev-apps.svc.cluster.local
```

`hosts` specifies the hostname, `gateways` specifies the gateway that is to be used for incoming traffic, and 
`http.route.destination.host` points to the corresponding service.

*That's it, your service is exposed to the external world using istio* :)

## 4. Setting up external-dns:
You can use external-dns to crete DNS entry using the following manifest:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: external-dns
  namespace: tools
spec:
  releaseName: external-dns-release
  chart:
    repository: https://kubernetes-charts.storage.googleapis.com
    name: external-dns
    version: 2.10.2
  values:
    sources:
      - istio-gateway
    istioIngressGateways:
      - istio-system/istio-ingressgateway
    domainFilters:
      - web.hostname.com
    txtOwnerId: rancher
    registry: txt
    provider: aws
    policy: sync
    rbac:
      create: true
      apiVersion: v1beta1
    tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "app"
        effect: "NoSchedule"
    extraEnv:
      - name: AWS_ACCESS_KEY_ID
        valueFrom:
          secretKeyRef:
            name: external-dns-secrets
            key: aws_access_key_id
      - name: AWS_SECRET_ACCESS_KEY
        valueFrom:
          secretKeyRef:
            name: external-dns-secrets
            key: aws_secret_access_key
```

## Issues Faced:
1. Istio doesn't allow multiple gateways against the same TLS certficate, so we had to go with a seperate namespace 
`istio-shared` and keep our gateways and virtualServices in that particular namespace so that they are accessible to
each other. You can specify multiple hosts(it's an array) or use wildcard "*.hostname.com" in gateway to allow multiple
hostnames against the same gateway.
Reference: https://istio.io/docs/ops/troubleshooting/network-issues/#404-errors-occur-when-multiple-gateways-configured-with-same-tls-certificate


## Relevant links:
https://istio.io/docs/concepts/
https://www.infoworld.com/article/3328817/what-is-istio-the-kubernetes-service-mesh-explained.html
https://blog.jayway.com/2018/10/22/understanding-istio-ingress-gateway-in-kubernetes/
https://rinormaloku.com/istio-practice-gateways/

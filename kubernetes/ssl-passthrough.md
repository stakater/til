### **TLS Passthrough**

There are occasions when you don't want the TLS to be terminated on the ingress and prefer to terminate in the pod instead. Note, by terminating in the pod the ingress will no longer be able to perform any L7 actions, so all of the feature set is lost (effectively it's become a TLS proxy using SNI to route the traffic)

#### **Example steps**

First create a kubernetes secret containing the certificate you wish to use.

```shell
$ kubectl create secret tls tls --cert=cert.pem --key=cert-key.pem
```

Create the deployment and service.

```YAML
---
apiVersion: v1
kind: Service
metadata:
  labels:
    name: tls-passthrough
  name: tls-passthrough
spec:
  type: ClusterIP
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: 10443
  selector:
    name: tls-passthrough
---
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tls-passthrough
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        name: tls-passthrough
    spec:
      volumes:
      - name: certs
        secret:
          secretName: tls
      containers:
      - name: proxy
        image: quay.io/ukhomeofficedigital/nginx-proxy:v3.2.0
        ports:
        - name: https
          containerPort: 10443
          protocol: TCP
        env:
        - name: PROXY_SERVICE_HOST
          value: "127.0.0.1"
        - name: PROXY_SERVICE_PORT
          value: "8080"
        - name: SERVER_CERT
          value: /certs/tls.crt
        - name: SERVER_KEY
          value: /certs/tls.key
        - name: ENABLE_UUID_PARAM
          value: "FALSE"
        - name: NAXSI_USE_DEFAULT_RULES
          value: "FALSE"
        - name: PORT_IN_HOST_HEADER
          value: "FALSE"
        volumeMounts:
        - name: certs
          mountPath: /certs
          readOnly: true
      - name: fake-application
        image: kennethreitz/httpbin:latest
```

Push out the ingress resource indicating you want ssl-passthrough enabled.

```YAML
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    ingress.kubernetes.io/ssl-passthrough: "true"
    kubernetes.io/ingress.class: "nginx-external"
  name: tls-passthrough
spec:
  rules:
  - host: tls-passthrough.notprod.homeoffice.gov.uk
    http:
      paths:
      - backend:
          serviceName: tls-passthrough
          servicePort: 443
        path: /
```

## Reference

- https://github.com/UKHomeOffice/application-container-platform/blob/master/how-to-docs/ssl-passthrough.md

# Multiple nginx-ingress with one default backend service

Nginx-ingress requires default backend service to route all calls which it doesn't understand. We can have multiple nginx-ingress share the same default backend via the following config: 

```
  defaultBackend:
    enabled: false
  controller:
    defaultBackendService: global/stakater-global-external-ingress-default-backend
```

Here `defaultBackendService` contains the <namespace>/<service-name> of an already running default backend service. In this way one default backend service will be shared.
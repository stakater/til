# Mount a file from configmap

This shows how to mount a file from a configmap into a pod as volume in a particular directory

```
          volumeMounts:
          - name: lab-users
            mountPath: /opt/jboss/keycloak/standalone/configuration/import/lab-users-0.json
            subPath: lab-users-0.json
      volumes:
      - name: lab-users
        configMap:
          name: keycloak
          items:
            - key: lab-users-0.json
              path: lab-users-0.json
```

Source: https://github.com/kubernetes/kubernetes/issues/44815#issuecomment-297077509

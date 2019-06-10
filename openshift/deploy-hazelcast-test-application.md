# Deploying test hazelcast application in Openshift

## App

Stakater provides a [sample hazelcast app](https://github.com/stakater-lab/hazelcast-app) that can be used to connect with Hazelcast.
The app exposes three endpoints:
- To add data just send request to `/hazelcast/write-data` which expects two request parameters named `key` and `value`.
- To fetch data just send request to `/hazelcast/read-data` which expects one request parameters named `key`.
- To fetch all data just send request to `/hazelcast/read-all-data` without any additional parameters.

All the data is handled inside a map named `my-map`.

## Deployment

The following Template uses an image of the above repository to deploy a sample hazelcast app that communicates with Hazelcast.

```yaml
apiVersion: v1
kind: Template
labels:
  template: hazelcast-test-app
  name: hazelcast-test-app
metadata:
  name: hazelcast-test-app
  annotations:
    description: hazelcast-test-app
    tags: java, springboot

objects:
- apiVersion: v1
  kind: DeploymentConfig
  metadata:
    labels:
      app: ${NAME}
    name: ${SERVICE_NAME}
  spec:
    replicas: 1
    strategy:
      resources: {}
      rollingParams:
        intervalSeconds: 1
        maxSurge: 25%
        maxUnavailable: 25%
        timeoutSeconds: 600
        updatePeriodSeconds: 1
      type: Rolling
    template:
      metadata:
        labels:
          app: ${NAME}
          deploymentconfig: ${SERVICE_NAME}
          zone: application
      spec:
        containers:
        - name: ${SERVICE_NAME}
          image: "stakater/hazelcast:1.0.0"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /actuator/health
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /actuator/health
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources:
            limits:
              cpu: ${SERVICE_CPU}
              memory: ${SERVICE_MEMORY}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
    test: false
    triggers:
    - type: ConfigChange


- apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: ${NAME}
    name: ${SERVICE_NAME}
  spec:
    ports:
    - name: 8080-tcp
      port: 8080
      protocol: TCP
      targetPort: 8080
    selector:
      app: ${NAME}
      deploymentconfig: ${SERVICE_NAME}
    sessionAffinity: None
    type: ClusterIP

parameters:

# -----------------------
# EXP parameters
# -----------------------
- name: NAME
  description: Name of application
  required: true
  value: hazelcast-test-app

# ---------------------
# Sample service parameters
# ---------------------
- name: SERVICE_NAME
  description: Name of service
  required: true
  value: hazelcast-test-app
- name: SERVICE_MEMORY
  description: Amount of memory for service
  required: true
  value: "450Mi"
- name: SERVICE_CPU
  description: Amount of cpu for service
  required: true
  value: "200m"
```

## Configuration

The default image expects hazelcast to be accessible at `hazelcast-service`.

The above template adds the label `zone: application` which can be modified as needed.

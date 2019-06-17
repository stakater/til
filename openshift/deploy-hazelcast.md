# Hazelcast in Openshift

## Deployment

To deploy hazelcast in openshift use the following template

```yaml
apiVersion: v1
kind: Template
metadata:
  name: hazelcast
  annotations:
    description: "Openshift deployment template for Hazelcast"
    tags: "hazelcast, imdg, datagrid, inmemory, kvstore, nosql, java"
    iconClass: "icon-java"
labels:
    template: hazelcast-openshift-template

parameters:
- name: HAZELCAST_IMAGE
  description: "Defines the location of Hazelcast image"
  value: hazelcast/hazelcast:3.11.1
  required: true
- name: SERVICE_NAME
  description: "Defines the service name of the POD to lookup of Kubernetes"
  value: hazelcast-service
  required: true
- name: NAMESPACE
  description: "Defines the namespace (project name) of the application POD of Kubernetes"
  required: true
- name: HAZELCAST_REPLICAS
  description: "Number of Hazelcast members"
  value : "3"
  required": true

objects:
- apiVersion: v1
  kind: ConfigMap
  metadata:
    name: hazelcast-configuration
  data:
    hazelcast.xml: |-
      <?xml version="1.0" encoding="UTF-8"?>
      <hazelcast xsi:schemaLocation="http://www.hazelcast.com/schema/config hazelcast-config-3.10.xsd"
                     xmlns="http://www.hazelcast.com/schema/config"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <properties>
          <property name="hazelcast.discovery.enabled">true</property>
        </properties>
        <network>
          <join>
            <multicast enabled="false"/>
            <tcp-ip enabled="false" />
            <!-- kubernetes enabled="true"/ -->
            <discovery-strategies>
              <discovery-strategy enabled="true" class="com.hazelcast.kubernetes.HazelcastKubernetesDiscoveryStrategy">
              </discovery-strategy>
            </discovery-strategies>
          </join>
        </network>
      </hazelcast>

- apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: hazelcast
    labels:
      app: hazelcast
  spec:
    replicas: ${HAZELCAST_REPLICAS}
    selector:
      matchLabels:
        app: hazelcast
    template:
      metadata:
        labels:
          app: hazelcast
          zone: database
      spec:
        containers:
        - name: hazelcast-openshift
          image: ${HAZELCAST_IMAGE}
          ports:
          - name: hazelcast
            containerPort: 5701
          livenessProbe:
            httpGet:
              path: /hazelcast/health/node-state
              port: 5701
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /hazelcast/health/node-state
              port: 5701
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 1
          volumeMounts:
          - name: hazelcast-storage
            mountPath: /data/hazelcast
          env:
          - name: HAZELCAST_KUBERNETES_SERVICE_DNS
            value: ${SERVICE_NAME}.${NAMESPACE}.svc.cluster.local
          - name: JAVA_OPTS
            value: "-Dhazelcast.rest.enabled=true -Dhazelcast.config=/data/hazelcast/hazelcast.xml"
          resources:
            limits:
              memory: 400Mi
              cpu: 250m
        volumes:
        - name: hazelcast-storage
          configMap:
            name: hazelcast-configuration

- apiVersion: v1
  kind: Service
  metadata:
    name: ${SERVICE_NAME}
    labels:
      app: hazelcast
  spec:
    type: ClusterIP
    clusterIP: None
    selector:
      app: hazelcast
    ports:
    - protocol: TCP
      port: 5701
```

### Note

This template will add a label `zone: database` to the deployment which can be modified as needed e.g for pod policies etc.

## Deploying test hazelcast application in Openshift

### App

Stakater provides a [sample hazelcast app](https://github.com/stakater-lab/hazelcast-app) that can be used to connect with Hazelcast.
The app exposes three endpoints:
- To add data just send request to `/hazelcast/write-data` which expects two request parameters named `key` and `value`.
- To fetch data just send request to `/hazelcast/read-data` which expects one request parameters named `key`.
- To fetch all data just send request to `/hazelcast/read-all-data` without any additional parameters.

All the data is handled inside a map named `my-map`.

### Deployment

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
          image: "stakater/hazelcast-test-app:1.0.0"
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
```

### Configuration

The default image expects hazelcast to be accessible at `hazelcast-service`.

The above template adds the label `zone: application` which can be modified as needed.

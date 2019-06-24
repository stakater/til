# Mongodb Deployment on Kubernetes using statefulsets

## Overview

This document will provide guidelines on how to deploy mongodb database on kubernetes cluster(deployed on AWS) using k8s stateful sets with helm chart.

## Deployment Steps

In this section I will deploy mongdb stateful set using minimum configuration

* Create a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) using the manifest given below, this manifest is specific to AWS.

```yaml
apiVersion: storage.k8s.io/v1beta1  
kind: StorageClass  
metadata:  
  name: fast
provisioner: kubernetes.io/aws-ebs  
parameters:  
  type: gp2
```

Storage class will be create at kubernetes cluster level and will be used by statefulsets to create pvc for each replica. To create storage class use the command given below:

```bash
$ sudo kubectl apply -f storage class
```

To check whether storage class is created or not use the command given below:
```bash
$ sudo kubectl get storageclasses
```


* Now we will deploy mongodb statefulset using helm chart. Helm chart manifest is given below:

```yaml
apiVersion: flux.weave.works/v1beta1
kind: HelmRelease
metadata:
  name: mongodb-deployment
  namespace: aaa
spec:
  releaseName: mongodb-deployment
  chart:
    repository: https://kubernetes-charts.storage.googleapis.com
    name: mongodb
    version: 5.17.0
  values:

    usePassword: false
    service:
      clusterIP: None
    port: 27017
    
    replicaSet:
      replicas:
        secondary: 1
      pdb:
        minAvailable:
          primary: 1
      enabled: true
      name: mongo
    
    podLabels:
      role: mongo
      environment: test
    
    persistence:
      mountPath: /bitnami/mongodb
      enabled: true
      storageClass: "fast"
      accessMode: 
        - "ReadWriteOnce"
      size: 2Gi
      
      
```


Use the command given below to create the statefulset
```bash
$ sudo kubectl apply -f mongoHelmChart.yaml -n <namespace-name>
```

To check statefulset is created or not use the command given below:
```bash
$ sudo kubectl get statefulsets -n <namespace-name>
```



## Notes

### Scaling

* Scaling statefulsets will increase the number of pvc but scaling down will not delete the pvc because it requires the users to copy the data and delete the pvc manually.

* To check whether configuration have been initialized successfully use the command given below on each pod:

```bash
rs.conf()
```
It will output something like this:

```json
{
	"_id" : "mongo",
	"version" : 3,
	"protocolVersion" : NumberLong(1),
	"writeConcernMajorityJournalDefault" : true,
	"members" : [
		{
			"_id" : 0,
			"host" : "mongodb-deployment-primary-0.mongodb-deployment-headless.aaa.svc.cluster.local:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 5,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 1,
			"host" : "mongodb-deployment-secondary-0.mongodb-deployment-headless.aaa.svc.cluster.local:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 2,
			"host" : "mongodb-deployment-arbiter-0.mongodb-deployment-headless.aaa.svc.cluster.local:27017",
			"arbiterOnly" : true,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 0,
			"tags" : {
				
			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	],
	"settings" : {
		"chainingAllowed" : true,
		"heartbeatIntervalMillis" : 2000,
		"heartbeatTimeoutSecs" : 10,
		"electionTimeoutMillis" : 10000,
		"catchUpTimeoutMillis" : -1,
		"catchUpTakeoverDelayMillis" : 30000,
		"getLastErrorModes" : {
			
		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("5cd10dbdb6f72bf0a19ed7f7")
	}
}

```


* An issue was faced by me and it was causing me to not access mongodb because I was not using the master node of statefulset to access the database. This issue was resolved by running the command given below on non-master nodes:

```bash
rs.slaveOk()
```


### Refrences

| Description  | Links  |
|---|---|
| Mongodb Helm Chart  | https://github.com/helm/charts/tree/master/stable/mongodb |
| Mongodb Replication Documentation  | https://docs.mongodb.com/manual/replication/ |


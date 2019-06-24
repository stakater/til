# Mongodb Deployment on Kubernetes using statefulsets

## Overview

This document will provide guidelines on how to deploy mongodb database on kubernetes cluster(deployed on AWS) using k8s stateful sets.

## Deployment Steps

In this section we will deploy mongodb on kubernetes by following the steps given below:

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

* Create a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) using the manifest given below:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    aaa-service: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    role: mongo
```

To create the above service use the command given below:

```bash
$ sudo kubectl apply -f headless-service.yaml -n <namespace-name>
```
To check service is created use the command given below:

```
sudo kubectl get services -n <namespace-name>
```

* Create a [statefulset](https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset) using the manifest given below:

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  template:
    metadata:
      labels:
        role: mongo
        environment: test
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mongo
          image: mongo:3.4
          command:
            - mongod
            - "--replSet"
            - rs0
            - "--smallfiles"
            - "--noprealloc"
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db
        - name: mongo-sidecar
          image: cvallance/mongo-k8s-sidecar
          env:
            - name: MONGO_SIDECAR_POD_LABELS
              value: "role=mongo,environment=test"
  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "fast"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 2Gi
```

Use the command given below to create the statefulset
```bash
$ sudo kubectl apply -f stateful-sets.yaml -n <namespace-name>
```

To check statefulset is created or not use the command given below:
```bash
$ sudo kubectl get sfs -n <namespace-name>
```



## Notes

### Scaling

* Scaling statefulsets will increase the number of pvc but scaling down will not delete the pvc because it requires the users to copy the data and delete the pvc manually.

* Until now the statefulsets doesn't share data. Found a [question](https://stackoverflow.com/questions/43827185/what-should-be-used-for-sharing-the-volume-of-statefulset-between-its-pods-nfs) regarding this issue. 

`Solution`: This issue was resolved by intiating a mongodb replication configurations by using the command given below:
```json
rs.initiate({ 
  _id : "rs0", 
  members: [
    { _id: 0, host: "mongo-0.mongo:27017" }, 
    { _id: 1, host: "mongo-1.mongo:27017" }, 
    { _id: 2, host: "mongo-2.mongo:27017"}
  ]
})
```

In the above configuration we just need to assign each host an id, one of them will be act as `Primary` while other act as `Secondaries`. 

To check whether configuration have been initialized successfully use the command given below:

```bash
rs.conf()
```
It will output something like this:

```json
{
	"_id" : "rs0",
	"version" : 1,
	"protocolVersion" : NumberLong(1),
	"members" : [
		{
			"_id" : 0,
			"host" : "mongo-0.mongo:27017",
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
			"_id" : 1,
			"host" : "mongo-1.mongo:27017",
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
			"host" : "mongo-2.mongo:27017",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
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
		"catchUpTimeoutMillis" : 60000,
		"getLastErrorModes" : {
			
		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("5ccfd17a79b0a49ce7ee22fa")
	}
}
```
One issue still exists, which is to automate the mongodb replication configuration initialization, [issue link](https://github.com/cvallance/mongo-k8s-sidecar/issues/57)


* An issue was faced by me and it was causing me to not access mongodb because I was not using the master node of statefulset to access the database. This issue was resolved by running the command given below:

`Solution`: This issue can be resolved by using the solution given above.



### Refrences

| Description  | Links  |
|---|---|
| Google guidelines on mongodb deployment  |  https://codelabs.developers.google.com/codelabs/cloud-mongodb-statefulset/index.html?index=..%2F..index#0 |
| Blog post explaing issue in google guidelines  | http://pauldone.blogspot.com/2017/06/deploying-mongodb-on-kubernetes-gke25.html  |
| k8s Mongodb  | http://k8smongodb.net/  |
| Mongodb Replication Documentation  | https://docs.mongodb.com/manual/replication/  |


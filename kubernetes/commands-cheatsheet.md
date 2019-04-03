# Basic Commands


* It returns the version of the client and master
```bash
kubectl version
```

* It will return cluster inoformation and link to the UI
```bash
kubectl cluster-info
```

* It will return all the nodes that can be used to host applications
```bash
kubectl get nodes
```

* It will list the current deployments
```bash
kubectl get deployments
```

* It will create a proxy that will forward communications into the cluster-wide, private network. The proxy can be terminated by pressing control-C and won't show any output while its running

```bash
kubectl proxy
```

* Show detailed informaton about a resource
```bash
kubectl describe <resource-type> <resource-name>
```

* Show logs of a container
```bash
kubectl logs <container-name>
```

* It will return running pods list
```bash
kubectl get pods 
```

* It will start an interactive shell inside a container
```bash
kubectl exec -ti $POD_NAME /bin/bash
```

* To get namespaces
```bash
Kubectl get namespaces.
```

* To create a namespace

There are two ways to create a namespace using json or yaml file. Create a json file and insert the content given below in that file 

 
```json
{ 
  "kind": "Namespace", 
  "apiVersion": "v1", 
  "metadata": { 
    "name": "qa", 
    "labels": { 
      "name": "qa" 
    } 
  } 
} 
```

Run the command given below 
```bash
kubectl create -f <filename> 
```

* To assign a context to a namespace
```bash
kubectl config set-context dev --namespace=dev --cluster=minikube --user=minikube
``` 

* To change the context
```bash
kubectl config use-context <context-name>
```
 
* To get the current context 
```bash
kubectl config current-context 
```
 

* To delete a namespace
```bash
kubectl delete namespaces dev 
```

# Basic Commands

This file contains collection of basic commands for k8s:

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

* It will return detailed informaton about a resource
```bash
kubectl describe <resource-type, like pods or service> <resource-name>
```

* It will return logs of a container
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

* It will return namespaces
```bash
Kubectl get namespaces.
```

* To create a namespace follow the instructions given below

There are two ways to create a namespace either by using json or yaml file. Create a json file and insert the content given below in that file:

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

Run the command given below to create namespace by using the file given above
```bash
kubectl create -f <filename> 
```

* To assign a context to a namespace
```bash
kubectl config set-context <context-name, like dev> --namespace=<namespace name, like dev> --cluster=<cluster name, like minikube> --user=<name of the user, like minikube>
``` 

* It will change current context
```bash
kubectl config use-context <context-name>
```
 
* It will return current context 
```bash
kubectl config current-context 
```
 
* It will delete a namespace
```bash
kubectl delete namespaces dev 
```

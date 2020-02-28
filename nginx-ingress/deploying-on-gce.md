# Deploying Nginx Ingress Controller on GCE

GKE itself gives its Ingress COntroller but it doesn't provide force https redirection. And for that we deploy Nginx Ingress Controller on GCE. SO now there will be 2 ingress controllers.

## Working

Nginx Ingress Controller deploys a Load Balancer and it will have a public IP. You will have to change your DNS entries and point it to Nginx Load Balancer IP. Then in your ingress, when you will mention a host e.g. `qa.DOMAIN` and will have different paths. So the routing will be

Browser -> Domain -> Load Balancer IP -> Google Load Balancer -> Nginx Pod -> Backend Pod

## Issue regarding Paths

Nginx ingress controller after version 0.22 doesn't support rewriting targets of ingresses of different paths. So you would have to create different Ingresses for different paths

one with  /*
Other with /static/*

Basically, the rewrites are different for different paths so one cannot define them using one ingress. I asked the maintainer of nginx ingress controller, he said

```
If you need different rewrites, you need to split the ingress definition. You can only have one rewrite-target annotation per ingress and that applies to all the paths
```

## Issue with deploying with GCE Controller

Nginx Ingress Controller updating Ingresses with no Ingress Class specified.

When deploying Nginx Ingress Controller, and if you dont specify class, the default class is `nginx`. But in our case when the Ingresses were using GCE Ingress Controller, so there was no class specified in the INgresses, so Nginx Ingress Controller was also updating them so solution is to specify a class when deploying Nginx Ingress Controller. 

Check https://github.com/kubernetes/ingress-nginx/issues/5189 for details

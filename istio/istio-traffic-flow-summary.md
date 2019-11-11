# Istio traffic flow summary

1. Client makes a request against a certain hostname(pointing to your loadbalancer IP) at a certain port
2. Loadbalancer forwards the request to the relevant node
3. The request is intercepted by `istio ingressgateway service`
4. Istio ingressgateway service forwards the request to a `istio ingressgateway pod`, The pod is configured by a `gateway` 
and `virtualservice`
5. The pod routes the call to the `service` configured against the virtualservice. 
6. The service redirects the call to the corresponding pods. 

Taken from:
https://blog.jayway.com/2018/10/22/understanding-istio-ingress-gateway-in-kubernetes/

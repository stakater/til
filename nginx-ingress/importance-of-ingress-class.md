# Importance of Ingress class

Ingress class tell nginx-ingress which controller to route through. Its default value is 'nginx', and we specify it via annotation while creating an Ingress. In case we want to create multiple nginx-ingress in our cluster, we must create them with different ingress class, and then pass the correct class to Ingress.
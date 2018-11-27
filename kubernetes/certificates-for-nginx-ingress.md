# Certificates for nginx-ingress

While creating nginx-ingress, we can specify certificate arn for our load balancers. We need to make sure the certificate arn are present in the same region load balancers are being created otherwise nginx-ingress's created service will be stuck in Pending state, and no load balancer will be created. 

# Using static website hosted on s3 as maintenance page

Create an S3 bucket with the name of the service against which you want to use a maintenance page and use that for static 
website hosting. **Ensure that bucket name is based on this pattern `namespacename-servicename-port`**. This is the 
`proxy_upstream_name` that nginx ingress controller will autogenerate and if there is a mismatch in name you won't be 
redirected to the maintenance page. 

Once everything is setup just annotate your ingress like this:
`nginx.ingress.kubernetes.io/server-snippet: proxy_ssl_server_name on; error_page 502 503 504 @maintenance; location @maintenance { proxy_pass http://namespacename-servicename-port.s3-website-ap-southeast-1.amazonaws.com;}`

Where `http://namespacename-servicename-port.s3-website-ap-southeast-1.amazonaws.com` is the URL to your S3 static hosted 
website.

** Note: Whenever you are trying to implement a maintenance page redirect for your website ensure that it's handled at your 
reverse proxy(if any) and at the loadbalancer/server side as well, like in case of AWS use cloudfront.


## Useful links:

https://medium.com/faun/how-to-host-your-static-website-with-s3-cloudfront-and-set-up-an-ssl-certificate-9ee48cd701f9

https://medium.com/@willmorgan/moving-a-static-website-to-aws-s3-cloudfront-with-https-1fdd95563106

https://aws.amazon.com/blogs/aws/create-a-backup-website-using-route-53-dns-failover-and-s3-website-hosting/

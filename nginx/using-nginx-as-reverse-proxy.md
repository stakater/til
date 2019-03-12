# Using nginx as reverse proxy

One of the benefit of nginx is that we can use it as a proxy server. We need to have a `location` block,  which specifies the `proxy_pass`. `location` block should be in a `server` block, which should be in `http` block. A sample config file is shown below: 

```
server {
     listen 9000;
     listen [::]:9000;

     server_name localhost;

     location = /api/v1/query_range {
        resolver 8.8.8.8;
        proxy_pass https://my-website.com$request_uri;
     } 
   }
```
Refer to the following doc to see how `proxy_pass` work: 

```
https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
```

In the above example, all requests matching the above location `/api/v1/query_range` will be forwarded to `https://my-website.com/all-query-params-in-original-request`. 

`$request_uri` at the end will ensure that all query params are forwarded to the proxy pass.

If we do not use any variable in our `proxy_pass`, then we do not need any `resolver`, but in our above example we have used a variable `$request_uri`, so we need to specify an external dns resolver for proxy pass to work. I've temporarily used the google resolver `8.8.8.8` as it is a very common public dns resolver.
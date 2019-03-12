# Using spring cloud for reverse proxy

We can use `ZUUL` proxy in spring boot to use it as a reverse proxy. Add the following dependacy first:

```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>
``` 

In your MainApplication.java, add the following annotation `@EnableZuulProxy` 

In your properties file, add ZUUL configuration like this: 

```
zuul:
  routes:
    foos:
      path: /prometheus/**
      url: https://my-prometheus-url/api/v1/query_range
```

Now all the calls with `/prometheus` in the request path, will be forwarded to the following address `https://my-prometheus-url/api/v1/query_range`, and the response will be returned from the proxied server

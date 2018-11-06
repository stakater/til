## Self Signed Certificates Connectivity Issues on OpenShift/Kubernetes

This is the scenario:

- I have microservice A which calls microservice B.
- microservice A and microservice B both have a Destination CA Cert configured in their Route (termination is reencrypt)
- I am calling microservice A and microservice B from browser, Chrome says my connection to them is secure
- however, when I call microservice B from microservice A, I get the PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

Hunting an error:

```
{“timestamp”:“2018-11-05T11:24:19.389+01:00",“message”:“Error getLaunchUrl: unable to get launch url I/O error on POST request for \“https://gateway-evolution.ocp.xyz.abc.se/v1/launchUrl\“:  sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; nested exception is javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target”,“logger_name”:“abc.xyz.service.token.handler.service.Service”,“thread_name”:“https-jsse-nio-8443-exec-10”,“applevel”:“ERROR”,“level_value”:40000,“logtype”:“server”}
```

Regardless - do make sure you fully understand the difference between:

- the keystore (in which you have the private key and cert you prove your own identity with) and 
- the trust store (which determines who you trust) - and the fact that your own identity also has a 'chain' of trust to the root - which is separate from any chain to a root you need to figure out 'who' you trust.

I have a Java client trying to access a server with a `self-signed certificate`.

When I try to Post to the server, I get the following error:

```
unable to find valid certification path to requested target
```

java -Djavax.net.debug=all -Djavax.net.ssl.trustStore=trustStore ...

to see if that helps. Instead of 'all' one can also set it to 'ssl', key manager and trust manager - which may help in your case. Setting it to 'help' will list something like below on most platforms.

Generally, Java trusts only certificates that are signed by certificate authorities or public certificates that exist in TrustStores (which contains list of certificates of trusted SSL servers/Certificate Authorities trusted to identify servers).

The problem is caused as Java is unable to find the certificate chain in TrustStore. This results in connection failure as Java does not trust the certificate on the destination server.

```
The CA you are using is not trusted by Java.
```

```
We need to mount right truststore in the pod
```


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

The root cause of above was that app was using a custom truststore and that didn't include the default root CA's which exist OOTB on the truststore!

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

If a truststore has been mounted then we need to ensure that it has correct CA certs:

Trust store generally (actually should only contain root CAs but this rule is violated in general) contains the certificates that of the root CAs (public CAs or private CAs). You can verify the list of certs in trust store using

```
keytool -list -v -keystore truststore.jks
```

How and where to find the keystore/truststore (certs) on OpenShift java docker images?

```
sh-4.2$ which java
/usr/bin/java
sh-4.2$ cd /usr/bin/java
sh: cd: /usr/bin/java: Not a directory
sh-4.2$ ls -l /usr/bin/java
lrwxrwxrwx. 1 root root 22 Aug  1 19:09 /usr/bin/java -> /etc/alternatives/java
sh-4.2$ ls -l /etc/alternatives/java
lrwxrwxrwx. 1 root root 73 Aug  1 19:09 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64/jre/bin/java
sh-4.2$ cd /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el7_5.x86_64/jre/
sh-4.2$ cd lib/security/
sh-4.2$ ls -l
total 52
-rw-r--r--. 1 root root  1273 Jul 16 17:34 blacklisted.certs
lrwxrwxrwx. 1 root root    41 Aug  1 19:09 cacerts -> ../../../../../../../etc/pki/java/cacerts
-rw-r--r--. 1 root root  2466 Jul 16 17:34 java.policy
-rw-r--r--. 1 root root 40921 Aug  1 20:25 java.security
-rw-r--r--. 1 root root   139 Jul 16 17:39 nss.cfg
drwxr-xr-x. 4 root root    38 Aug  1 19:09 policy
sh-4.2$ keytool -v -list -keystore cacerts
Enter keystore password: [changeme]
```

## How to generate & use new RSA key to use for signing in KeyCloak?

First ensure you have `openssl` installed on your machine.

### Step 1: Generate a 2048 bit RSA Key

You can generate a public and private RSA key pair like this:

`openssl genrsa -des3 -out private.pem 2048`

That generates a 2048-bit RSA key pair, encrypts them with a password you provide, and writes them to a file. You need to next extract the public key file. 

### Step 2: Export the RSA Public Key to a File

This is a command that is:

`openssl rsa -in private.pem -outform PEM -pubout -out public.pem`

The `-pubout` flag is really important. Be sure to include it.

Next open the `public.pem` and ensure that it starts with a `-----BEGIN PUBLIC KEY-----`. This is how you know that this file is the public key of the pair and not a private key.

### Step 3: Export the Private Key to a File

This is a command that is:

`openssl rsa -in private.pem -out private_unencrypted.pem -outform PEM`

The `-pubout` was dropped from the end of the command. That changes the meaning of the command from that of exporting the public key to exporting the private key outside of its encrypted wrapper. Inspecting the output file, in this case private_unencrypted.pem clearly shows that the key is a RSA private key as it starts with `-----BEGIN RSA PRIVATE KEY-----`.

### Step 4: Upload new key to KeyCloak

Go to `Realm Settings > Keys > Providers`

To add a keypair obtained elsewhere (e.g. we generated with OpenSSL) select Providers and choose `rsa` from the dropdown. You can change the priority to make sure the new keypair becomes the active keypair.

Click on Select file for Private RSA Key to upload your private key. The file should be encoded in PEM format. You don’t need to upload the public key as it is automatically extracted from the private key.

Keep in mind if you want this new key to be used then its priority should be higher than other ones; e.g. if current max for the active ones is 100 then 101 is good enough for it to be selected.

### Step 5: Base64 encode the public key and private key to provide as secret to KeyCloak

Select the key part excluding following lines and then just base64 encode it.

```
-----BEGIN PUBLIC KEY-----

-----END PUBLIC KEY-----
```

Select the key part excluding following lines and then just base64 encode it.

```
-----BEGIN PRIVATE KEY-----

-----END PRIVATE KEY-----
```

In the KeyCloak secrets we need to provide base64 encoded values.

### Step 6: Provide public key to spring boot in application.yaml as it is i.e. 

In the spring boot we provide public key as it is:

```
lab:
  jwtPublicKey: |
                -----BEGIN PUBLIC KEY-----
                MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz8Y+umAvSQKgOHq4FbIg
                RSfJ436ItzpQ1SsdbO1QDjMxgl7od5wWOUvf4F0qk0Z93ZmZuQopKQysYRazkGd2
                UvVq3edh79BNET5okG3IuX+4gfhC8W1BgpP6BQlF61w0Q86u/4ey7fXWcWRUX4+g
                pC2D7bgUsAwbtGATFJzF9+0Tp3MWj6+tZa6jmiDah8Jpds++NVDu07jsLw/AvyO2
                jaqjr5Fdk7MX8DWFVPebliT/UKijEru6uJ2u5xn9YDWYpxbRGq6pipcqQj/Icvvz
                bh5+lyLaB1foRiOSm4q9MMmG6q+KIxPLhytg/GYokHqA7pUgFqWOlFmVvSMwzeWd
                8QIDAQAB
                -----END PUBLIC KEY-----
```

### References

- https://rietta.com/blog/2012/01/27/openssl-generating-rsa-key-from-command/

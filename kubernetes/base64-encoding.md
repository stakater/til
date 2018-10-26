When manually creating a kubernetes secret, each item must be base64 encoded. On linux you can use the base64 utility to manually encode before adding to the secret.

One thing to keep in mind is to use the -n flag with the utility. Without this flag echo automatically adds a new line at the end of the string so the end result will have a new line character encoded, and so for a case like authentication etc., the secret you have a provided, and the value you may enter will not match because of the difference of a new line character.

```
$ echo 'admin' | base64
YWRtaW4K

$ echo -n 'admin' | base64
YWRtaW4=
```

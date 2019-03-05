# Use Jenkins credentials in pipeline library

For using Jenkins credentials functionality we use `Credentials Binding plugin`. Once we create a credential in the form of a secret or a username/password or any other way supported by the plugin we can use the credentials in our pipeline library via environment variables. 

## Example

Let's suppose we created a secret credential with id: `my-secret-id` and text: `my-secret-token`. In our pipeline we can use the secret like this: 

```
    def mySecret
    withCredentials([string(credentialsId: 'my-secret-id', variable: 'mySecret')]) {
        mySecret = env.mySecret
    }
   
    echo "My secret: ${mySecret}"
```

Note that the above step is for scripted pipeline only as in our case. 
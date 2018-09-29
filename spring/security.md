## Error - Invalid token does not contain resource id (oauth2-resource)

**Problem**

```
{
    "error": "access_denied",
    "error_description": "Invalid token does not contain resource id (oauth2-resource)"
}
```

**Solution**
https://stackoverflow.com/questions/47996344/why-is-my-token-being-rejected-what-is-a-resource-id-invalid-token-does-not-c

```
    @Override
    public void configure(ResourceServerSecurityConfigurer config) {
        LOGGER.debug("Running with OAuth2 Security!");
        config.tokenServices(tokenServices());
        /**
         * If you don't set this then you will get this error
         * "error_description": "Invalid token does not contain resource id (oauth2-resource)"
         *
         * In case of keycloak; the resourceId should be set to the clientId.
         */
        config.resourceId(audience);
    }
```

## Problem

### Unexpected error when authenticating with identity provider
### OR
### Facebook app login error - Can't Load URL: The domain of this URL isn't included in the app's domains
```
"error": {
      "message": "Can't Load URL: The domain of this URL isn't included in the app's domains. To be able to load this URL, add all domains and subdomains of your app to the App Domains field in your app settings.",
      "type": "OAuthException",
      "code": 191,
      "fbtrace_id": "CwaEPJ4+X8p"
}

```

- First of all check your if your "Client ID" and "Client Secret" match with the one in your developer account.
- Now for the re-direct URL
## For Facebook
- Go to 
```

https://developers.facebook.com/apps/{APP_ID}/fb-login/settings/

// APP_ID = Your facebook APP ID

```
- In the "Valid OAuth Redirect URIs" copy paste the Redirect URL from keycloak.
- Now on the left hand of your window. Click "Setting". Select "Basic".
- Scroll to the very bottom. In the "Website" panel under "Site URL" copy paste your redirect URL from keycloak.
- Now scroll up. Find the "App Domain"
- In the text Box under it. Do the following
  ```

  // If your redirect url is this
  https://keycloak.realm-dev.yourdomain.com/auth/realms/realm/broker/facebook/endpoint

  // In the text box put this
  keycloak.realm-dev.yourdomain.com

  ```

## For Google
  - Click credentials on the left of the pannel.
  - Click keycloak 
  - In the text box under "Authorized redirect URIs" type your Redirect url from keycloak. e.g
  ```
  https://keycloak.realm-dev.yourdomain.com/auth/realms/carbook/broker/google/endpoint	
  ```
  - In the "Authorized JavaScript origins" type redirect url like this
```
  https://keycloak.realm-dev.yourdomain.com
```

# Setup google identity provider with keycloak

- Open https://console.developers.google.com and select your project.
- Open credentials tab from lab navigation bar
- Open `OAuth consent screen` tab and select Application type, give application name and add your authorized domain. In our case, it is `stackator.com` and `stakater.com`
- Open `Credentials` tab and create `OAuth 2.0 client IDs`. Give your `Authorized redirect URIs`. In stakater's case it is https://keycloak.tools.stackator.com/auth/realms/stakater/broker/google/endpoint
Here is how to build this url
    - https://keycloak.tools.stackator.com is your external url for keycloak.
    - `stakater` is your realm name.
- Enable api for `google+` by searching it from top search bar in google console.
- After performing these steps, update the `clientId` and `clientSecret` with newly created `OAuth 2.0 client IDs` of your google identity provider in keycloak.
- If you face `Invalid parameter: redirect_uri` or `URI mismatch` when authenticating with google, please make sure
    - You have correct `clientId` and `clientSecret` of google idp.
    - Your keycloak client has `Valid Redirect URIs`. It must contain the internal endpoint keycloak service.
    - Your `Authorized redirect URIs` in google console has valid uris.
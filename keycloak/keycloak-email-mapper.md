# Issue when accessing Jenkins after authentication from Keycloak

When setting up Jenkins, and setting security realm of Keycloak, so after redirecting from keycloak, it gave error `java.lang.Exception: no field 'email' was supplied in the token payload to be used as the username`.

This means that Keycloak was unable to map the email and send that information to Jenkins, so to fix you need to create a Client Scope in your realm of Keycloak and add to default client scope of your client. Follow these steps:

## Manual Steps

1. Go to you Realm -> Client Scopes, create a new Scope, name it email and fill its values as:

Name:  email
Description: OpenID Connect built-in scope: email
Protocol: openid-connect
Display On Consent Screen: ON
Consent Screen Text: ${emailScopeConsentText}
Include In Token Scope: ON
GUI order:

2. Go to Mappers. Click `Add Builtin` and select email & email verified.

3. Go to your client, `stakater-online-platform` in our case, Go to Client Scopes and add email in Assigned Default Client Scopes.

## Through Configmap

Add the following in the Client Scope

```yaml
						{
              "name": "email",
              "description": "OpenID Connect built-in scope: email",
              "protocol": "openid-connect",
              "attributes": {
                "include.in.token.scope": "true",
                "display.on.consent.screen": "true",
                "consent.screen.text": "${emailScopeConsentText}"
              },
              "protocolMappers": [
                {
                  "name": "email",
                  "protocol": "openid-connect",
                  "protocolMapper": "oidc-usermodel-property-mapper",
                  "consentRequired": false,
                  "config": {
                    "userinfo.token.claim": "true",
                    "user.attribute": "email",
                    "id.token.claim": "true",
                    "access.token.claim": "true",
                    "claim.name": "email",
                    "jsonType.label": "String"
                  }
                },
                {
                  "name": "email verified",
                  "protocol": "openid-connect",
                  "protocolMapper": "oidc-usermodel-property-mapper",
                  "consentRequired": false,
                  "config": {
                    "userinfo.token.claim": "true",
                    "user.attribute": "emailVerified",
                    "id.token.claim": "true",
                    "access.token.claim": "true",
                    "claim.name": "email_verified",
                    "jsonType.label": "boolean"
                  }
                }
              ]
						},
```

And in your client, add the above scope in default scopes by

```yaml
					"defaultClientScopes": [
                Other Scopes,
                "email"
              ]
            },
```

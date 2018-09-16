
## Problem - Can't reach login page of KeyCloak when redirecting it from sample react app

In browser I see:

```
This page isn’t working
keycloak.cp.lab187.k8syard.com is currently unable to handle this request.
HTTP ERROR 500
```

In KeyCloak logs I see:

```
06:41:06,382 WARN  [org.keycloak.events] (default task-50) type=LOGIN_ERROR, realmId=51323b4d-018d-4f0d-9e69-94abda19e1d9, clientId=webapp, userId=null, ipAddress=85.0.238.64, error=invalid_user_credentials, auth_method=openid-connect, auth_type=code, response_type=code, redirect_uri=http://localhost:3000/secured, code_id=bbbe66f3-f676-457c-a36b-94d9cea670dd, response_mode=fragment

06:41:06,385 ERROR [org.keycloak.services.error.KeycloakErrorHandler] (default task-50) Uncaught server error: java.lang.NullPointerException

06:41:06,385 ERROR [org.keycloak.services.error.KeycloakErrorHandler] (default task-50) Failed to create error page: java.lang.NullPointerException
```

## Solution

Verify `Themes` in `Realm Settings`

I was referring to a theme which didn't exist; as I have given wrong value in my configs i.e.

```
      "realm": "stakater",
      "enabled": true,
      "loginTheme": "fabric8",
```

And I had never added the theme `fabric8` in my deployment! So, I changed it to:

```
      "realm": "stakater",
      "enabled": true,
      "loginTheme": "keycloak",
      "accountTheme": "keycloak",
      "adminTheme": "keycloak",
      "emailTheme": "keycloak",
```


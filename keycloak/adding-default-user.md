# Adding a user in Keycloak via realm.json

To add a user in Keycloak add a list object `"users"` in realm object

Following is a sample:

```
{
    "realm": "stakater",
    "enabled": true,
    "loginTheme": "keycloak",
    .
    .
    .
    "users": [
    {
        "username": "stakater-user",
        "enabled": true,
        "emailVerified": false,
        "email": "stakater-user@placeholder.org",
        "credentials": [
        { 
            "type" : "password",
            "value" : "stakater@qwerty786"
        }
        ],
        "realmRoles": ["offline_access", "uma_authorization"],
        "clientRoles": {
        "realm-management": ["view-users", "manage-authorization"],
        "broker": ["read-token"],
        "stakater-online-platform": ["uma_protection"],
        "account": ["manage-account", "view-profile"]
        }
    }],
    .
    .
    .
```


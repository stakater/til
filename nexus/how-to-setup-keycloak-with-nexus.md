# How to setup keycloak with nexus

- Nexus needs some sort of user mappings to support external authentication.

- Nexus uses Remote User Token Authentication(RutAuth) to validate external authentication. RutAuth needs a header to setup. This header will contain the value that nexus is going to map with it's local users. After RutAuth has been setup, it looks for that specific header in incoming authentication request to authenticate.

- There are two ways to use keycloak with nexus from web ui,
    1. Create local users in the nexus that you want to authenticate and add X-Auth-Email or X-Auth-Username header in RutAuth. keycloak sends these two headers in authentication request by default. RutAuth will try to map the email or username of local existing user with incoming authentication request's email or username.

    2. Create a single local user in the nexus and add X-Auth-User header in RutAuth. Add X-Auth-User as header in keycloak proxy config and set it's value equals to the username of user created in nexus. RutAuth will try to map the username of local existing user with incoming authentication request's username.

- Currently keycloak with nexus does not support machine users like jenkins or maven etc. Keycloak authentication with nexus only works from web ui.
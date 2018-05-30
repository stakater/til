# Why Authentication not automatically synchronized between application & keycloak?

Authentication is not automatically synchronized between the application and Keycloak. Please note that this the standard OpenID Connect workflow, and that we expect to do some specific improvements in JHipster on this matter. As a result:

-When a user logs out of the application, he will be automatically logged in again if he refreshes his browser: this is because he is still logged in Keycloak, which provides automatic authentication.

- When a user’s session is invalidated in Keycloak, if the user is already logged into the application, he will still be able to use the application for a while. This is because OpenID Connect is a stateful mechanism, and the application doesn’t know immediately that the session has been invalidated.
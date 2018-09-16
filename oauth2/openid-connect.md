## What is id token?

OpenID Connect is a simple identity layer built on top of the OAuth 2.0 protocol.

First, in order to use the identity functionality, we’ll make use of a new OAuth2 scope called openid. This will result in an extra field in our Access Token – “id_token“.

The id_token is a JWT (JSON Web Token) that contains identity information about the user, signed by identity provider (in our case Google).

Finally, both server(Authorization Code) and implicit flows are the most commonly used ways of obtaining id_token

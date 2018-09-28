## Roles

In KeyCloak we have those 3 roles:

1. Realm Role
2. Client Role
3. Composite Role

There are no User Roles in KeyCloak. You most likely confused that with User Role Mapping, which is basically mapping a 
role (realm, client, or composite) to the specific user

Every Realm has one or multiple Clients. And every Client can have multiple Users attached to it.

Now from this it should be easy to conclude how role mappings work.

- Realm Role: It is a global role, belonging to that specific realm. You can access it from any client and map to any user. Ex Role: 'Global Admin, Admin'
- Client Role: It is a role which belongs only to that specific client. You cannot access that role from a different client. You can only map it to the Users from that client. Ex Roles: 'Employee, Customer'
- Composite Role: It is a role that has one or more roles (realm or client ones) associated to it.

Source: https://stackoverflow.com/a/47857926

## Group vs Roles

In the IT world the concepts of Group and Role are often blurred and interchangeable. In Keycloak, Groups are just a collection of users that you can apply roles and attributes to in one place. Roles define a type of user and applications assign permission and access control to roles

Aren’t Composite Roles also similar to Groups? Logically they provide the same exact functionality, but the difference is conceptual. Composite roles should be used to apply the permission model to your set of services and applications. Groups should focus on collections of users and their roles in your organization. Use groups to manage users. Use composite roles to manage applications and services.

Source: https://www.keycloak.org/docs/3.0/server_admin/topics/groups/groups-vs-roles.html

## What is Client Scope?

When an OIDC access token or SAML assertion is created, all the user role mappings of the user are, by default, added as claims within the token or assertion. Applications use this information to make access decisions on the resources controlled by that application. In Keycloak, access tokens are digitally signed and can actually be re-used by the application to invoke on other remotely secured REST services. This means that if an application gets compromised or there is a rogue client registered with the realm, attackers can get access tokens that have a broad range of permissions and your whole network is compromised. This is where client scope becomes important.

Client scope is a way to limit the roles that get declared inside an access token. When a client requests that a user be authenticated, the access token they receive back will only contain the role mappings you’ve explicitly specified for the client’s scope. This allows you to limit the permissions each individual access token has rather than giving the client access to all of the user’s permissions. By default, each client gets all the role mappings of the user. You can view this in the Scope tab of each client.

- Full Scope
- Partial Scope

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

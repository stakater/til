# keycloak custom attributes for users

KeyCloak has different set of screens/components to manage different parts e.g.:

- registration
- account management
- console

As of so, far they don't have any common way to manage them; the plan is to add "Profile SPI" which will be awesome

And we would like to add following to it:

- Required
- Validation
- User changeable (could potentially require a specific role as well)
- Admin changeable
- Attribute type

Adding a new attribute is not a problem but to apply rules on it in a generic way is not possible at the moment; and "Profile 
SPI" will add the possibility.

Read these:

- https://github.com/cfpb/hmda-platform-auth/issues/42
- https://issues.jboss.org/browse/KEYCLOAK-2966

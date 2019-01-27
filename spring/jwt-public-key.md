# KeyCloak RSA Generated PublicKey

Where to find JWT Public Key in Keycloak?

In `application.yaml`

```
keycloak:
  jwtPublicKey: |
                  -----BEGIN PUBLIC KEY-----
                  MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAiv4c6oSQtvGmP++/xBBek/k+F0mUvwIpiMZ7qyfWHo5RaImoCo7ZMhQfxaY4fFcKX1xiS7Qk3+CaXZTyC4BHq4PAV5hokH5l06TrMtKM85RdFTTT44eEfFPTkl+SFCYCR5HgQ6/NsaZVUcIl8I0mog/k9bVzKgQZtt5XXbBtWrBJCehezPOMxVIRoaSfrN7Qyd/1EaGXc7sij1WnFz9UxS0KaTuS8yvWT3R4W4tkl3clqAg1eKNx96E3jS3b77H/JQhoPbBzzopNOuzik5XXAca3u01boU01zTp7PYuuV6xhMxYL8Pv48li6L7nG1ST2Ry+5vPW5pKXqTn371A1ZBwIDAQAB
                  -----END PUBLIC KEY-----
``

Login into Keycloak and go to:

`Realm Settings` > `Keys` > `Active`

Then click and copy `Public Key` for Type `RSA` and that's it!

If this value is wrong then you will get 401 unauthorized

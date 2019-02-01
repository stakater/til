### How to send a post request to a resource authenticated by Keycloak ? 

- In the first cy.request(), there is a "POST" call to retrieve a Bearer token from keycloak.
- In the second cy.request(), the token retrieved from the first call is being used to create a "POST" request to Provider resource.  

```
describe("Carbook Sign in Page", () => {
    let token;
    let providerId;
    beforeEach(() => {
        cy.request({
        method: 'POST',
        url: 'https://keycloak.carbook-dev.gocarbook.com/auth/realms/carbook/protocol/openid-connect/token',
        form : true,
        body: {
            "username":"",
            "password":"",
            "grant_type":"password",
            "client_id": "web_app"
        },
        timeout: 120000,
        failOnStatusCode: false
    }).then((resp) => {
            token = resp.body.access_token;
            cy.task('log',token);
            cy.request({
                method: 'POST',
                url: 'https://providers-service.carbook-mock.gocarbook.com/api/providers',
                auth: {
                    'bearer':token
                },
                body: {
                    "name": "Test Carbook Provider",
                    "rating": 4.5,
                    "logo": "Asylum-logo",
                    "address": {
                        "longitude": null,
                        "latitude": null,
                        "location": "Arkham Asylum Street 57, Area Restricted",
                        "city": "Arkham"
                    },
                    "contact": {
                        "email": "qwe@rty.com",
                        "mobile": "03001112223",
                        "landline": 1348315
                    }
                },
                timeout: 120000,
                failOnStatusCode: false
            }).then((resp) => {
                providerId = resp.body;
                cy.task('log',providerId)
            })
        })
    
    })
    
    it("calling post request", () => {
        cy.log("kashifali")
    })
});
```

- An Important thing to remember is that when using "auth" in a "cy.request" always put "bearer" in single quotes.

```

auth: {
    'bearer':token
    }
```
- If you get a 403 http error one reason for it that you might be doing is sending "send Immediately: false" in auth. Remove that.

```
// Remove "sendImmediately: false" if you get 403 http error.
auth: {
    sendImmediately: false,
    'bearer':token
    }

```
### Credit
- Worked on this issue with @kashifali00.
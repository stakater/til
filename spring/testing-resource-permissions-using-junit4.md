### How to test Resources authenticated by Keycloak 

- In the initialization function build your "MockMvc" with "webAppContextSetup" not "standAloneSetup"

```
// Correct way

import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;

@Autowired
    private WebApplicationContext webApplicationContext;

 @Before
    public void setup() {
        this.restBookingMockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                        .apply(springSecurity())
                        .build();
    }
```

```
// In-correct way, atleast to test permissions.
@Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        final UserResource userResource = new UserResource(IUserService);
        this.restUserMockMvc = MockMvcBuilders.standaloneSetup(userResource)
            .setCustomArgumentResolvers(pageableArgumentResolver)
            .setControllerAdvice(exceptionTranslatorControllerAdvice)
            .addFilter(springSecurityFilterChain)
            .setConversionService(createFormattingConversionService())
            .setMessageConverters(jacksonMessageConverter).build();
    }

```

- Create a call to the keyCloakServer to get a bearer token

```
 public String getAccessToken(String userName, String password, String grantType) throws IOException {

        String url = keycloakProperties.getServerUrl() + "/realms/" + keycloakProperties.getRealm()
                + "/protocol/openid-connect/token";
        MultiValueMap<String, String> params = new LinkedMultiValueMap<String, String>();
        params.add("grant_type", grantType);
        params.add("username", userName);
        params.add("password", password);
        params.add("client_id", keycloakProperties.getAudience());

        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<MultiValueMap<String, String>>
                (params, httpHeaders);

        ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
        ObjectMapper objectMapper = new ObjectMapper();
        Map<String, String> map = new HashMap<String, String>();
        map = objectMapper.readValue(response.getBody(), new TypeReference<Map<String, String>>() {
        });

        return map.get("access_token");
    }
```

- Now use this function to test your resource. 

```
    String USER_TOKEN = testUtil.getAccessToken("admin@admin.com", "admin", "password");
    restBookingMockMvc.perform(post("/api/bookings")
                .contentType(TestUtil.APPLICATION_JSON_UTF8)
                .content(TestUtil.convertObjectToJsonBytes(apibooking))
                .header("Authorization", "Bearer " + USER_TOKEN))
                .andExpect(status().isCreated());

```
- The TOKEN is sent as ".header("Authorization", "Bearer " + USER_TOKEN))".
- In order to get an access token the User MUST exist and be given specific Roles in Keycloak. e.g the User "admin@admin.com" exists inside keycloak with ROLE_ADMIN.
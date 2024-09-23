 # Testing OAuth2 Authorization Server


  Testing involves ensuring that your OAuth2 server is correctly handling client registration, authentication, and token issuance. Here's how to proceed with the testing:

1. Testing OAuth2 Authorization Server Functionality Using Postman
1.1. Testing the /oauth2/token Endpoint
Postman can be used to test your OAuth2 flow to ensure that the token endpoint works correctly.

Step-by-Step Guide:
Start Your Authorization Server Application: Make sure your Spring Boot application is running.

Open Postman and create a new request.

Set Up the Request:

URL: http://localhost:9000/oauth2/token (replace 9000 with your actual port)
Method: POST
Authorization: Select "Basic Auth" and provide your clientId and clientSecret:
Username: messaging-client (or whatever clientId you configured)
Password: secret (or whatever clientSecret you configured)
Set Headers:

Content-Type: application/x-www-form-urlencoded
Set the Body Parameters: Choose "x-www-form-urlencoded" and provide the following key-value pairs:

grant_type: client_credentials (or authorization_code if you want to test with an authorization code)
scope: message.read (replace with the appropriate scopes you have set up)
Send the Request: Click "Send," and you should receive a response containing an access_token if everything is set up correctly.

Expected Response:
```
{
   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
   "token_type": "Bearer",
   "expires_in": 3600,
   "scope": "message.read"
}
```
1.2. Testing the /oauth2/authorize Endpoint (for Authorization Code Grant)
If you are testing the authorization_code grant type:

Open a browser and navigate to:

```
http://localhost:9000/oauth2/authorize?response_type=code&client_id=messaging-client&redirect_uri=http://127.0.0.1:8080/authorized&scope=message.read
```
This will prompt you to log in using the username/password defined in your UserDetailsService.

After logging in, you should be redirected to your redirect_uri with an authorization code in the URL.

Exchange this authorization code for an access token via a POST request to /oauth2/token using Postman.

2. Writing Integration Tests Using JUnit
2.1. Setting Up Your Test Class
Here's how you can write a simple integration test using JUnit 5 and Spring Boot's test utilities:

Add Dependencies for Testing
Ensure your pom.xml has the following dependencies:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```
2.2. Writing the Test Class
```
package om.gov.moh.spring_auth2_server;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.oauth2.core.endpoint.OAuth2AccessTokenResponse;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class OAuth2AuthorizationServerTests {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void testTokenEndpointWithClientCredentials() {
        String baseUrl = "http://localhost:" + port + "/oauth2/token";

        // Set headers
        HttpHeaders headers = new HttpHeaders();
        headers.setBasicAuth("messaging-client", "secret"); // Replace with your client ID and secret

        // Set body parameters
        MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
        body.add("grant_type", "client_credentials");
        body.add("scope", "message.read");

        // Create the HTTP request
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(body, headers);

        // Send the request
        ResponseEntity<OAuth2AccessTokenResponse> response = restTemplate.exchange(baseUrl, HttpMethod.POST, request, OAuth2AccessTokenResponse.class);

        // Assertions
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().getAccessToken().getTokenValue()).isNotEmpty();
    }
}
```
Explanation of the Test:
This test sends a request to the /oauth2/token endpoint using the client_credentials grant type.
It uses TestRestTemplate to simulate the request and verifies that the access token is issued correctly.
3. Additional Tests
You can add additional tests to check:

Unauthorized requests
Access token expiration
Invalid client credentials
Summary
Use Postman to manually test the OAuth2 endpoints (/oauth2/token and /oauth2/authorize).
Write integration tests using JUnit and Spring Boot's test utilities to automate testing.


## check if any error found

1. Check Client Credentials
Ensure that the client_id and client_secret provided in your request match exactly with what you have defined in your database or in-memory configuration.

For example, based on your configuration:

clientId: messaging-client
clientSecret: secret
If you’re testing via Postman or another tool, make sure these values are used correctly.

2. Verify Basic Authentication Header in Postman
When using Postman:

Go to the Authorization tab.
Select Basic Auth as the type.
Enter client_id (messaging-client) in the Username field.
Enter client_secret (secret) in the Password field.
This ensures the proper Authorization: Basic header is set, with the client_id and client_secret encoded in Base64.

3. Verify Registered Client Configuration in Your Application
Make sure that your RegisteredClient is properly saved in your JPA repository or in-memory configuration. Here's a quick check:

Example of RegisteredClient Configuration
```
@Bean
public RegisteredClientRepository registeredClientRepository(PasswordEncoder passwordEncoder) {
    RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
            .clientId("messaging-client")
            .clientSecret(passwordEncoder.encode("secret")) // Ensure it's encoded correctly
            .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
            .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
            .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
            .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
            .redirectUri("http://127.0.0.1:8080/login/oauth2/code/messaging-client-oidc")
            .redirectUri("http://127.0.0.1:8080/authorized")
            .scope(OidcScopes.OPENID)
            .scope(OidcScopes.PROFILE)
            .scope("message.read")
            .scope("message.write")
            .build();

    return new InMemoryRegisteredClientRepository(registeredClient);
}
```
Ensure clientSecret is properly encoded using the BCryptPasswordEncoder.
Make sure the clientAuthenticationMethod is set to ClientAuthenticationMethod.CLIENT_SECRET_BASIC.
4. Verify Password Encoding
If you've changed the clientSecret, make sure you’re using the encoded value when registering the client. If you store client details in your database, the clientSecret should be encoded using BCryptPasswordEncoder.

You can test encoding with the following:

```
PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
System.out.println(passwordEncoder.encode("secret"));
```
Store this encoded value in your database.

5. Common Troubleshooting Steps
Ensure Proper Endpoints: Make sure you're using the correct URL, for example, /oauth2/token.
Correct Headers: Set Content-Type: application/x-www-form-urlencoded in your request.
Database Check: If using JPA, ensure your registered_client table is populated correctly.
Summary
The invalid_client error typically results from incorrect client credentials, improper Basic Auth setup, or an improperly registered client. Ensure all configurations, credentials, and encoding match exactly as expected by the server.

## Troubleshoot the invalid_client Error

Steps to Troubleshoot the invalid_client Error
1. Verify the Client Credentials in Postman
Make sure you are using the exact values when testing with Postman:

Client ID: messaging-client
Client Secret: secret (in plain text, not encoded)
When using Basic Authentication in Postman:

Go to the Authorization tab.
Set the type to Basic Auth.
Enter messaging-client in the Username field.
Enter secret in the Password field.
Postman will automatically encode the credentials in Base64.

2. Verify the Password Encoder Setup in Your Spring Boot Configuration
Ensure that your PasswordEncoder is set up correctly in your SecurityConfig and is being used in your RegisteredClientRepository:

```
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```
Ensure that the clientSecret is stored in the database in the BCrypt-encoded format, as you have done.

3. Double-Check Your Registered Client Configuration
If you're using JPA, verify that your client information is correctly stored in the registered_client table, with the correct client_id and client_secret fields.

You can do this by running an SQL query like:

```
SELECT client_id, client_secret FROM registered_client WHERE client_id = 'messaging-client';
```
Make sure that the client_secret column contains the BCrypt-encoded password "$2a$10$7dUvPLGBgPzfxO1I59YQ3ubEsGJN6AEV9dG1ZBa1M9BdpkY.pTOG".

4. Test Encoding Programmatically
You can test the password encoding to verify that BCrypt is correctly encoding the value. Add this to a simple test or main method:

```
public static void main(String[] args) {
    PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    boolean matches = passwordEncoder.matches("secret", "$2a$10$7dUvPLGBgPzfxO1I59YQ3ubEsGJN6AEV9dG1ZBa1M9BdpkY.pTOG");
    System.out.println("Does the password match? " + matches);
}
```
If this prints true, it means that the BCrypt encoding is working correctly, and the stored password matches the provided one.

5. Check Spring Security OAuth2 Configuration
Ensure that your RegisteredClientRepository is being used correctly and there are no conflicting beans or configurations that could be affecting how client authentication is handled.

Common Issues and Solutions
Ensure Consistent Configuration: Ensure you haven't mixed InMemoryRegisteredClientRepository and JPA-based configurations in your SecurityConfig.
Case Sensitivity: Ensure that the client_id and client_secret are entered with the exact same casing as they are stored in the database.
Testing the Token Endpoint Again
After verifying everything, try again using Postman:

POST request to /oauth2/token.
Use Basic Auth for authentication.
Set grant_type to client_credentials in the request body.
Include scope=message.read or any other scope you have configured.
If everything is set up correctly, you should receive a valid access token in response.

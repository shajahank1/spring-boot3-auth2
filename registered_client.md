# registered_client and user_account Tables

Yes, you should create the registered_client table in your Oracle database, as it serves a different purpose from your user_account table. Here's an explanation of the difference between the two tables and why you need the registered_client table.

Difference Between registered_client and user_account Tables
1. registered_client Table
The registered_client table is used to store information about OAuth2 clients that are allowed to connect to your Authorization Server. These clients represent applications (e.g., web apps, mobile apps, backend services) that need to obtain an access token from the Authorization Server to access protected resources on behalf of users or for their own use.

Key Information Stored in registered_client Table:
Client ID: A unique identifier for each OAuth2 client.
Client Secret: A secret key used to authenticate the client (in confidential clients).
Redirect URIs: The allowed URLs where the authorization server can redirect the client after authorization.
Grant Types: The OAuth2 grant types supported (e.g., authorization_code, client_credentials).
Scopes: The permissions the client is requesting.
Token Settings: Access token validity, refresh token validity, etc.
In summary, this table contains the information necessary to manage and control access for different applications (clients) using OAuth2.

2. user_account Table
The user_account table is used to store information about end-users who log in to your system. These are typically the actual human users who want to access protected resources via the OAuth2 clients.

Key Information Stored in user_account Table:
User ID: A unique identifier for each user.
Login ID / Username: The user's login credentials.
Password: The encoded password for authentication.
Status: Whether the user is active or inactive.
Other User Information: Details such as user profile information.
Why You Need Both Tables:
The registered_client table manages application-level authentication and authorization. It controls which applications are allowed to interact with the authorization server.
The user_account table manages user-level authentication and authorization. It handles the end-user login process.
How They Work Together:
When a client (e.g., a mobile app or a web app) tries to access your API, it authenticates itself with the Authorization Server using the credentials stored in the registered_client table.
Once the client is authenticated, it may prompt the user to log in. The user's login credentials are verified against the user_account table.
Upon successful authentication, the Authorization Server issues an access token, allowing the client to access protected resources on behalf of the user.
Example Use Cases:
A web application wants to access protected APIs from your resource server. It must be registered as a client, with its details stored in the registered_client table. When the web app initiates the OAuth2 flow, the authorization server checks this table to validate the client.
Once the client is validated, the web application prompts the end-user to log in. The user's credentials are checked against the user_account table, and if valid, an access token is issued.
How to Create the registered_client Table in Oracle
You can create the registered_client table with the following SQL script:

```
CREATE TABLE registered_client (
    id VARCHAR2(36) PRIMARY KEY,
    client_id VARCHAR2(50) UNIQUE NOT NULL,
    client_secret VARCHAR2(100) NOT NULL,
    redirect_uris CLOB,
    scopes CLOB,
    authorization_grant_types CLOB,
    client_authentication_methods CLOB,
    token_settings CLOB,
    client_settings CLOB
);
```
Adjust the data types and lengths according to your requirements.
Store redirect_uris, scopes, authorization_grant_types, and other complex fields as CLOB (Character Large Objects) since they might store multiple values.
Summary
You need the registered_client table to manage the OAuth2 clients (applications) that will interact with your authorization server.
The user_account table handles the actual user login and authentication process.
They serve different roles in the OAuth2 architecture, working together to provide secure authorization and authentication.


##Ways to Populate the registered_client Table
The data for the registered_client table is typically entered by an administrator or developer responsible for managing the OAuth2 clients that need access to your authorization server. This process is called "client registration," and there are a few ways to populate this table.

Ways to Populate the registered_client Table
1. Manual Insertion via SQL Scripts
You can manually insert data into the registered_client table using SQL commands. This method is suitable when you have a small number of clients or during the initial setup.

Example SQL Insert Statement
```
INSERT INTO registered_client (
    id, client_id, client_secret, redirect_uris, scopes, authorization_grant_types, client_authentication_methods, token_settings, client_settings
) VALUES (
    'a1b2c3d4-e5f6-7890-ab12-34567890abcd', -- A unique UUID
    'messaging-client',                     -- The client_id
    '$2a$10$7dUvPLGBgPzfxO1I59YQ3ubEsGJN6AEV9dG1bZBa1M9BdpkY.pTOG', -- BCrypt encoded client_secret ("secret")
    'http://127.0.0.1:8080/login/oauth2/code/messaging-client-oidc,http://127.0.0.1:8080/authorized', -- Comma-separated redirect URIs
    'openid,profile,message.read,message.write', -- Comma-separated scopes
    'authorization_code,refresh_token,client_credentials', -- Comma-separated grant types
    'client_secret_basic',                -- Client authentication method
    NULL,                                 -- You can leave token_settings NULL or add any additional settings in JSON if needed
    NULL                                  -- You can leave client_settings NULL or add any additional settings in JSON if needed
);
```
Note: Make sure to encode the client_secret using BCryptPasswordEncoder in your application or use an online tool to generate the hashed value.

2. Programmatic Registration Using a Java Service
If you want to automate client registration, you can create a service in your Spring Boot application to add entries to the registered_client table using JPA. This approach allows you to manage clients via your application code, which is more suitable for dynamic client registration.

Example: Programmatically Registering a Client
```
import om.gov.moh.spring_auth2_server.entity.RegisteredClientEntity;
import om.gov.moh.spring_auth2_server.repository.RegisteredClientRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.oauth2.core.AuthorizationGrantType;
import org.springframework.security.oauth2.core.ClientAuthenticationMethod;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.stereotype.Service;
import javax.annotation.PostConstruct;
import java.util.UUID;

@Service
public class ClientRegistrationService {

    @Autowired
    private RegisteredClientRepository registeredClientRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @PostConstruct // Automatically executed when the application starts
    public void registerDefaultClient() {
        if (registeredClientRepository.findByClientId("messaging-client").isEmpty()) {
            RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
                    .clientId("messaging-client")
                    .clientSecret(passwordEncoder.encode("secret")) // Encoding the client secret
                    .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                    .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                    .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                    .redirectUri("http://127.0.0.1:8080/login/oauth2/code/messaging-client-oidc")
                    .redirectUri("http://127.0.0.1:8080/authorized")
                    .scope("openid")
                    .scope("profile")
                    .scope("message.read")
                    .scope("message.write")
                    .build();

            // Saving to the database using JPA repository
            registeredClientRepository.save(registeredClient);
            System.out.println("Default client registered successfully.");
        }
    }
}
```
This ClientRegistrationService will automatically add a default client when the application starts, as long as it doesn't already exist in the database.

3. Admin Interface / Dashboard
For production systems, you might want to build a simple admin interface or dashboard that allows administrators to add, update, or remove clients via a web form. This interface would interact with the database via your application's service layer, providing a user-friendly way to manage OAuth2 clients.

Key Points for Registering Clients
Client ID: A unique identifier for the client, often a human-readable name like web-app-client.
Client Secret: A confidential password for the client. This should be encoded using BCrypt.
Redirect URIs: URLs where the Authorization Server can send the authorization code or tokens.
Scopes: Define the level of access that the client can request.
Authorization Grant Types: Defines which OAuth2 flows the client can use (e.g., authorization_code, client_credentials).
Testing Your Registered Client
Once the registered_client table is populated, you can test the client registration using tools like Postman or your application's front-end by initiating an OAuth2 authorization flow:

Navigate to: http://localhost:9000/oauth2/authorize?client_id=messaging-client&redirect_uri=http://127.0.0.1:8080/authorized&response_type=code&scope=openid message.read
Summary
You need to register clients in the registered_client table because it allows the OAuth2 Authorization Server to identify and authenticate which applications can request access tokens.
You can register clients using SQL scripts, programmatically via a Java service, or by building an admin interface.
Make sure all confidential data like client_secret is stored securely using a BCryptPasswordEncoder.


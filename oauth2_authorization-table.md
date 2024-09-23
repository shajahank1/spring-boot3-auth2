#  oauth2_authorization table
The oauth2_authorization table is managed and populated automatically by Spring Authorization Server's JdbcOAuth2AuthorizationService. When you configure your Spring Boot OAuth2 Authorization Server to use JdbcOAuth2AuthorizationService, it handles storing and retrieving OAuth2 authorization details (such as access tokens, refresh tokens, etc.) in the database.

How the oauth2_authorization Table Works
When a Token Is Issued:

When a client application requests an access token (e.g., using the authorization_code or client_credentials grant type), the Spring Authorization Server issues the token.
The JdbcOAuth2AuthorizationService automatically saves this information to the oauth2_authorization table in your database.
When a Token Is Validated or Retrieved:

Any time a token is used to access a protected resource, the authorization server retrieves the token from the oauth2_authorization table using JdbcOAuth2AuthorizationService.
How to Set Up and Configure JdbcOAuth2AuthorizationService
If you want to use JdbcOAuth2AuthorizationService to handle storing authorizations in your database, you need to configure it properly.

Step 1: Configure JdbcOAuth2AuthorizationService in Your Spring Boot Application
Ensure you have the following configurations:

Configuration Class
```
package om.gov.moh.spring_auth2_server.config;

import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.server.authorization.JdbcOAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;

@Configuration
public class OAuth2AuthorizationConfig {

    @Bean
    public OAuth2AuthorizationService authorizationService(DataSource dataSource, RegisteredClientRepository registeredClientRepository) {
        // This service will handle storing and retrieving authorizations from the database
        return new JdbcOAuth2AuthorizationService(dataSource, registeredClientRepository);
    }
}
```
Step 2: Ensure Your Database Has the Required Table Structure
Spring Authorization Server does not create the oauth2_authorization table automatically. You need to create the table using the following SQL schema:

SQL Schema for oauth2_authorization Table
```
CREATE TABLE oauth2_authorization (
    id VARCHAR(100) PRIMARY KEY,
    registered_client_id VARCHAR(100) NOT NULL,
    principal_name VARCHAR(200) NOT NULL,
    authorization_grant_type VARCHAR(100) NOT NULL,
    attributes BLOB NULL,
    state VARCHAR(500) NULL,
    authorization_code_value BLOB NULL,
    authorization_code_issued_at TIMESTAMP NULL,
    authorization_code_expires_at TIMESTAMP NULL,
    authorization_code_metadata BLOB NULL,
    access_token_value BLOB NULL,
    access_token_issued_at TIMESTAMP NULL,
    access_token_expires_at TIMESTAMP NULL,
    access_token_metadata BLOB NULL,
    access_token_type VARCHAR(100) NULL,
    access_token_scopes VARCHAR(1000) NULL,
    refresh_token_value BLOB NULL,
    refresh_token_issued_at TIMESTAMP NULL,
    refresh_token_expires_at TIMESTAMP NULL,
    refresh_token_metadata BLOB NULL
);

```

-- Optionally create an index for faster querying
```
CREATE INDEX idx_oauth2_authorization_registered_client_id ON oauth2_authorization (registered_client_id);
CREATE INDEX idx_oauth2_authorization_principal_name ON oauth2_authorization (principal_name);
```
How Data Is Inserted into the oauth2_authorization Table
Once you have JdbcOAuth2AuthorizationService configured and the table schema in place:

User Authentication: When a user or client authenticates and requests an OAuth2 token, the JdbcOAuth2AuthorizationService creates an authorization entry in the oauth2_authorization table.

Access Token Issuance: When an access token is issued, the details (token value, issued_at, expires_at, etc.) are stored in the access_token_value and related columns in the table.

Refresh Token Issuance: If a refresh token is granted, its details are stored similarly in the refresh_token_value and associated columns.

Testing Your Configuration
Start your application and ensure it's connected to the database.
When a client requests an authorization code or access token, you should see rows being inserted into the oauth2_authorization table.
Key Points to Remember
JdbcOAuth2AuthorizationService handles inserting, updating, and deleting authorization records from the database automatically.
You only need to configure JdbcOAuth2AuthorizationService and ensure your database has the correct schema.
Summary
Spring Authorization Server, through JdbcOAuth2AuthorizationService, automatically manages data insertion into the oauth2_authorization table.
You need to ensure your application is configured with JdbcOAuth2AuthorizationService and that your database table is created correctly.
By following these steps, your OAuth2 Authorization Server should be able to store and manage tokens using the oauth2_authorization table effectively. Let me know if you have further questions or need additional guidance!

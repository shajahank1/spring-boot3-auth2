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


## if i create oauth2_authorization , will it data is inserted in this table by spring auth server? how to know the table name by auth2 server

Yes, if you create the oauth2_authorization table as specified, Spring Authorization Server will insert data into this table automatically when you configure it to use JdbcOAuth2AuthorizationService. The Spring Authorization Server uses this table to store information about issued OAuth2 authorizations, such as access tokens, refresh tokens, and authorization codes.

How to Ensure the Correct Table Name Used by the Authorization Server
The table name used by Spring Authorization Server can be controlled or customized if needed. By default, the JdbcOAuth2AuthorizationService uses the table name oauth2_authorization. Here’s how you can check or modify the default behavior.

1. Default Table Name
The default table names used by Spring Authorization Server are:

oauth2_authorization: Stores authorizations (tokens)
oauth2_registered_client: Stores registered client information
If you are using JdbcOAuth2AuthorizationService, it will use these table names by default.

2. Verify Data Insertion
You can verify whether data is being inserted into the oauth2_authorization table by:

Running your Spring Authorization Server application.
Using one of the OAuth2 flows (e.g., authorization_code, client_credentials) to generate a token.
Checking the oauth2_authorization table in your database to see if entries have been created.
You can run a simple SQL query like:

```
SELECT * FROM oauth2_authorization;
```
You should see rows representing the issued tokens, refresh tokens, and authorization codes.

3. How to Customize Table Names (If Needed)
If you want to use a different table name, you can override the default behavior by providing a custom SQL statement when creating JdbcOAuth2AuthorizationService. Here’s an example of how to customize it:

Example of Customizing SQL for JdbcOAuth2AuthorizationService
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.server.authorization.JdbcOAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import javax.sql.DataSource;

@Configuration
public class OAuth2AuthorizationConfig {

    @Bean
    public OAuth2AuthorizationService authorizationService(DataSource dataSource, RegisteredClientRepository registeredClientRepository) {
        JdbcOAuth2AuthorizationService authorizationService = new JdbcOAuth2AuthorizationService(dataSource, registeredClientRepository);
        
        // Override default SQL statements if needed
        authorizationService.setAuthorizationSql("SELECT * FROM custom_authorization_table WHERE id = ?");
        authorizationService.setAuthorizationInsertSql("INSERT INTO custom_authorization_table (id, ...) VALUES (?, ...)");
        
        return authorizationService;
    }
}
```
Note: Replace custom_authorization_table with your desired table name and adjust the SQL accordingly.

4. Enable SQL Logging to Monitor Database Operations
To confirm what queries are being executed by Spring Authorization Server, you can enable SQL logging in your application. This way, you can see all SQL operations performed by the server and confirm that it's using the correct table.

Add the following to your application.properties or application.yml:

application.properties
properties
```
# Enable SQL logging
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
5. Check Spring Documentation
Spring Authorization Server documentation and source code often provide insights into default table names and SQL statements. You can refer to the official documentation or the source code on GitHub if you need further verification.

Summary
By default, Spring Authorization Server uses oauth2_authorization for storing tokens and authorizations.
Data will be automatically inserted into this table when you use JdbcOAuth2AuthorizationService.
You can enable SQL logging to monitor and confirm that data is being inserted into the correct table.
Customization is possible if you need to use a different table name.


## will the spring will create the tABLE or we need to create
ChatGPT said:
ChatGPT
Spring Authorization Server does not automatically create the oauth2_authorization table or any other related tables like oauth2_registered_client. You need to create these tables manually in your database before starting the application.

Why Doesn't Spring Create the Tables Automatically?
Spring Authorization Server does not have built-in database schema management, unlike Spring Data JPA, which can auto-generate schema based on entities. The expectation is that you’ll create the required tables according to the specifications provided by Spring Authorization Server.

Steps to Create the Required Tables
You will need to execute SQL scripts to create the necessary tables. Here’s how you can do it:

1. SQL Script for oauth2_authorization Table
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

CREATE INDEX idx_oauth2_authorization_registered_client_id ON oauth2_authorization (registered_client_id);
CREATE INDEX idx_oauth2_authorization_principal_name ON oauth2_authorization (principal_name);
```
2. SQL Script for oauth2_registered_client Table
You’ll also need to create the oauth2_registered_client table, which stores information about registered OAuth2 clients:

```
CREATE TABLE oauth2_registered_client (
    id VARCHAR(100) PRIMARY KEY,
    client_id VARCHAR(100) NOT NULL,
    client_id_issued_at TIMESTAMP NULL,
    client_secret VARCHAR(200) NULL,
    client_secret_expires_at TIMESTAMP NULL,
    client_name VARCHAR(200) NOT NULL,
    client_authentication_methods VARCHAR(1000) NOT NULL,
    authorization_grant_types VARCHAR(1000) NOT NULL,
    redirect_uris VARCHAR(1000) NULL,
    scopes VARCHAR(1000) NOT NULL,
    client_settings BLOB NULL,
    token_settings BLOB NULL
);

CREATE UNIQUE INDEX idx_oauth2_registered_client_client_id ON oauth2_registered_client (client_id);
```
How to Apply These SQL Scripts
Manually Execute the SQL Scripts: You can run these SQL scripts using your preferred database client (e.g., Oracle SQL Developer, pgAdmin, MySQL Workbench, etc.) before starting your Spring Boot application.

Automate with Flyway or Liquibase: If you want to manage database migrations automatically, you can use tools like Flyway or Liquibase. These tools help manage database schema changes and apply the scripts when your application starts.

Example: Using Flyway
If you choose to use Flyway, add the following dependency to your pom.xml:

```
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```
Create a directory in your project: src/main/resources/db/migration and add your SQL scripts there. Name them as V1__Create_OAuth2_Tables.sql (Flyway follows a naming convention).

Flyway will detect and apply the SQL scripts automatically when your Spring Boot application starts.

Summary
Spring Authorization Server will not create the oauth2_authorization or oauth2_registered_client tables automatically.
You need to create these tables manually or use a database migration tool like Flyway or Liquibase.
Make sure the table structures are in place before starting your application for the first time.
By creating the necessary tables, your Spring Authorization Server will be able to store and manage OAuth2 authorization and client data correctly. Let me know if you need any further assistance!


## oauth2_authorization  is mandatory ?


Yes, the oauth2_authorization table (or an equivalent mechanism) is mandatory if you want to use JdbcOAuth2AuthorizationService to manage OAuth2 authorization data in a database. This table is essential because it stores the information about issued tokens, refresh tokens, and authorization codes used in various OAuth2 flows.

Why Is oauth2_authorization Mandatory?
Storage of Token Data: The oauth2_authorization table holds data related to access tokens, refresh tokens, and authorization codes issued by the authorization server. This data is crucial for validating and managing OAuth2 sessions.
OAuth2 Grant Types: For grant types like authorization_code, client_credentials, password, and refresh_token, the server needs a place to persist token details, expiry times, scopes, and metadata.
When Is It Required?
You need the oauth2_authorization table when:

You want to persist issued tokens, refresh tokens, and authorization codes in a database (i.e., using JdbcOAuth2AuthorizationService).
You want to maintain state between multiple server restarts, ensuring tokens remain valid across application restarts.
Alternative Scenarios
If you are not using JdbcOAuth2AuthorizationService but are instead using an in-memory authorization service (InMemoryOAuth2AuthorizationService), the oauth2_authorization table is not required because all tokens will be managed in memory.

In-Memory Authorization Service
No Database Table Needed: If you use InMemoryOAuth2AuthorizationService, you don't need any database tables since all authorization data (tokens, codes, etc.) is stored in memory.
Limitations: The tokens will be lost every time you restart the application, which is impractical for production environments.
How to Choose Between JdbcOAuth2AuthorizationService and InMemoryOAuth2AuthorizationService
Use JdbcOAuth2AuthorizationService: When you need persistence, scalability, and reliability. This requires the oauth2_authorization table.
Use InMemoryOAuth2AuthorizationService: Only for development, testing, or small-scale applications where persistence isn't a concern.
Summary
Mandatory for Database Persistence: The oauth2_authorization table is mandatory if you're using JdbcOAuth2AuthorizationService to manage OAuth2 tokens.
Not Needed for In-Memory Storage: If you use InMemoryOAuth2AuthorizationService, you don't need this table, but tokens won’t persist across application restarts.
In most production scenarios, you will need the oauth2_authorization table to handle token persistence effectively. If you plan to use Spring Authorization Server with token storage in a database, make sure to create this table accordingly. Let me know if you have any further questions!

# after creating the table oauth2_authorization , do i need to configure anything in uath2 server to automatically insert

Once you've created the oauth2_authorization table, you don't need to configure much more for the Spring Authorization Server to start inserting data into this table. However, there are a few key steps to ensure that your setup is correct and that JdbcOAuth2AuthorizationService is properly wired up to use this table.

Steps to Ensure Automatic Insertion into oauth2_authorization Table
1. Ensure JdbcOAuth2AuthorizationService is Configured Correctly
The most important configuration is to set up JdbcOAuth2AuthorizationService as a bean in your Spring application context. This service handles the interaction between your Spring Authorization Server and the oauth2_authorization table.

Here's how to configure it:

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.server.authorization.JdbcOAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import javax.sql.DataSource;

@Configuration
public class OAuth2AuthorizationConfig {

    @Bean
    public OAuth2AuthorizationService authorizationService(DataSource dataSource, RegisteredClientRepository registeredClientRepository) {
        // This service will handle storing and retrieving authorizations from the database
        return new JdbcOAuth2AuthorizationService(dataSource, registeredClientRepository);
    }
}
```
2. Ensure You Have a Proper DataSource Bean Configured
The JdbcOAuth2AuthorizationService needs a DataSource to interact with your database. Make sure you have your database connection settings defined in application.properties or application.yml.

Example application.properties:
properties
```
spring.datasource.url=jdbc:oracle:thin:@localhost:1521:xe
spring.datasource.username=your_db_username
spring.datasource.password=your_db_password
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.datasource.initialization-mode=always
```
3. Ensure You Have the Required Dependencies
You need the Spring Authorization Server dependency and your database driver in your pom.xml or build.gradle file.

For Maven:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <scope>runtime</scope>
</dependency>
```
Replace ojdbc8 with the appropriate version for your Oracle database.

4. Register Clients Correctly Using RegisteredClientRepository
Ensure that your RegisteredClientRepository is correctly set up and that you have registered OAuth2 clients. This step ensures that your OAuth2 clients can request tokens from the Authorization Server.

Example Registered Client Configuration
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.server.authorization.client.JdbcRegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import javax.sql.DataSource;
import java.util.UUID;

@Configuration
public class ClientRegistrationConfig {

    @Bean
    public RegisteredClientRepository registeredClientRepository(DataSource dataSource) {
        RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
                .clientId("messaging-client")
                .clientSecret("{noop}secret") // Use BCrypt or another encoder in production
                .authorizationGrantType(org.springframework.security.oauth2.core.AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(org.springframework.security.oauth2.core.AuthorizationGrantType.REFRESH_TOKEN)
                .authorizationGrantType(org.springframework.security.oauth2.core.AuthorizationGrantType.CLIENT_CREDENTIALS)
                .redirectUri("http://127.0.0.1:8080/login/oauth2/code/messaging-client-oidc")
                .scope("message.read")
                .scope("message.write")
                .build();

        JdbcRegisteredClientRepository repository = new JdbcRegisteredClientRepository(dataSource);
        repository.save(registeredClient); // Save the client only once
        return repository;
    }
}
```
Note: The JdbcRegisteredClientRepository will manage storing registered clients in the oauth2_registered_client table.

5. Test Your OAuth2 Flow
Now that everything is configured:

Start your Spring Boot application.
Use an OAuth2 client (e.g., Postman) to request a token via one of the OAuth2 grant types (e.g., authorization_code or client_credentials).
When the authorization server issues a token, it will automatically insert a record into the oauth2_authorization table.
Verifying Data Insertion
Run the following SQL query to verify data has been inserted into your oauth2_authorization table:

sql
Copy code
SELECT * FROM oauth2_authorization;
You should see rows corresponding to the tokens and authorizations issued by your server.

Summary
You do not need additional configuration for data insertion as long as JdbcOAuth2AuthorizationService is correctly set up.
Ensure you have a DataSource configured, your table schema created, and that JdbcOAuth2AuthorizationService is properly declared as a bean.
The Spring Authorization Server handles the insertion of data into the oauth2_authorization table automatically.
With this setup, your OAuth2 tokens and authorization codes will be managed and persisted by Spring Authorization Server in your database

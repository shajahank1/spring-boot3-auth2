## Full Configuration for OAuth2 Authorization Server in Spring Boot 6
Add RegisteredClientRepository Bean
This bean defines how registered clients (applications that connect to your OAuth2 server) are stored. You can use an in-memory or JDBC-based approach.

Add ProviderSettings Bean
This defines the issuer URL and other settings for the OAuth2 server.

Complete Configuration
The code below includes all necessary beans, including RegisteredClientRepository, JWKSource for token signing, and AuthorizationServerSettings for configuring the issuer.

Updated SecurityConfig.java
```
package om.gov.moh.spring_auth2_server.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.OAuth2AuthorizationService;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.client.InMemoryRegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configurers.OAuth2AuthorizationServerConfigurer;
import org.springframework.security.oauth2.server.authorization.settings.AuthorizationServerSettings;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configuration.EnableOAuth2AuthorizationServer;
import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;

@Configuration
public class SecurityConfig {

    // 1. Authorization Server Security Configuration
    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
                .oidc(Customizer.withDefaults()); // Enable OpenID Connect 1.0

        http
                // Redirect to the login page when not authenticated from the authorization endpoint
                .exceptionHandling((exceptions) -> exceptions
                        .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login")))
                // Accept access tokens for User Info and/or Client Registration
                .oauth2ResourceServer((oauth2) -> oauth2
                        .jwt(Customizer.withDefaults()));

        return http.build();
    }

    // 2. Default Security Configuration
    @Bean
    @Order(2)
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        .anyRequest().authenticated())
                .formLogin(Customizer.withDefaults()); // Form login handles the redirect to login page

        return http.build();
    }

    // 3. Registered Client Repository - In-Memory
    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient registeredClient = RegisteredClient.withId("client-id")
                .clientId("client-id")
                .clientSecret("{noop}client-secret")  // Use NoOpPasswordEncoder for testing purposes
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                .redirectUri("https://example.com/callback")
                .scope("read")
                .scope("write")
                .build();

        return new InMemoryRegisteredClientRepository(registeredClient);
    }

    // 4. Authorization Server Settings
    @Bean
    public AuthorizationServerSettings authorizationServerSettings() {
        return AuthorizationServerSettings.builder()
                .issuer("https://your-authorization-server.com") // Set the issuer URL
                .build();
    }

    // 5. JWK Source for Token Signing
    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        // Generate a signing key (RSA or EC, for example)
        JWKSet jwkSet = new JWKSet();
        return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
    }

    // 6. OAuth2 Authorization Service Bean
    @Bean
    public OAuth2AuthorizationService authorizationService(RegisteredClientRepository registeredClientRepository) {
        return new InMemoryOAuth2AuthorizationService();
    }
}
```
Explanation of Changes:
RegisteredClientRepository Bean:
This in-memory client repository defines a client with ID client-id and secret client-secret. You can add more clients or switch to a JDBC-based repository if needed.

AuthorizationServerSettings Bean:
This defines the settings for the authorization server, including the issuer URL, which is required for OAuth2.

JWKSource Bean:
This bean is used for generating keys to sign JWT tokens issued by the OAuth2 server. You can replace this with a real key set (e.g., RSA or EC key) for production.

OAuth2AuthorizationService Bean:
This handles the persistence of authorization information, and we're using an in-memory implementation here. For production, you would likely switch to a database-backed implementation.

Dependency Updates
Ensure you have the correct dependencies for Spring Boot 6 and OAuth2:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.1.0</version> <!-- Check for the latest version compatible with Spring Boot 6 -->
</dependency>
```
Next Steps
Customize the RegisteredClientRepository to store clients in a database if needed.
Customize the JWKSource to sign tokens using a stronger key management process in production.

## explain

Let’s break down your SecurityConfig class, which is configuring Spring Boot Security and an OAuth2 Authorization Server. This configuration involves two separate security filter chains: one for the OAuth2 authorization server and another for default security for the rest of the application. Here's a detailed explanation of the code.

1. Two Security Filter Chains
In Spring Security, filter chains determine how requests are processed by the security framework. In your configuration, you have defined two filter chains:

authorizationServerSecurityFilterChain: This filter chain is responsible for securing the OAuth2 Authorization Server endpoints (e.g., token issuance, authorization, and OIDC).
defaultSecurityFilterChain: This is the default security filter chain for other parts of the application, such as login pages or resource access.
Order of Filter Chains:
The @Order(1) annotation ensures that the authorizationServerSecurityFilterChain is processed first. This is important because the authorization server should be handled with priority over other security configurations.
The @Order(2) annotation is applied to the defaultSecurityFilterChain, which is processed after the authorization server filter chain.
2. Authorization Server Filter Chain (authorizationServerSecurityFilterChain)
```
@Bean
@Order(1)
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
    OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
    http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
        .oidc(Customizer.withDefaults()); // Enable OpenID Connect 1.0
```
OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http):
This is a utility method that applies the default security configuration for an OAuth2 Authorization Server. It configures the necessary endpoints, such as /oauth2/authorize, /oauth2/token, and /oauth2/jwks to handle token requests, client registration, and key exposure.
http.getConfigurer(OAuth2AuthorizationServerConfigurer.class).oidc(Customizer.withDefaults()):
This enables OpenID Connect (OIDC) 1.0. OIDC is an identity layer built on top of OAuth2. It allows you to provide identity information (e.g., user profile data) to clients in addition to access tokens. By calling oidc(Customizer.withDefaults()), the basic OIDC functionality is enabled for your Authorization Server.
```
http
    // Redirect to the login page when not authenticated from the authorization endpoint
    .exceptionHandling((exceptions) -> exceptions
        .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login"))
    )
    // Accept access tokens for User Info and/or Client Registration
    .oauth2ResourceServer((oauth2) -> oauth2
        .jwt(Customizer.withDefaults()));
```
exceptionHandling():
This configures how authentication exceptions (like trying to access a protected resource without being authenticated) are handled. Specifically, it uses a LoginUrlAuthenticationEntryPoint to redirect users to the /login page if they are not authenticated when trying to access the authorization endpoints (e.g., /oauth2/authorize).

.oauth2ResourceServer(oauth2 -> oauth2.jwt()):
This configures the Authorization Server to act as an OAuth2 Resource Server that accepts and validates JWT (JSON Web Token) access tokens. This is important when an OAuth2 client submits a request to an OAuth2-protected endpoint using an access token, and the server needs to validate the token.

Return the filter chain:
Finally, http.build() constructs and returns the security filter chain for the Authorization Server.

3. Default Security Filter Chain (defaultSecurityFilterChain)
```
@Bean
@Order(2)
public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests((authorize) -> authorize
            .anyRequest().authenticated()
        )
        .formLogin(Customizer.withDefaults());
    
    return http.build();
}
```
authorizeHttpRequests():
This method is used to configure which requests should be authorized. In this case, it is a very broad configuration: any request (anyRequest()) to the application will require authentication (authenticated()). This means any incoming HTTP request must be from an authenticated user, or they will be redirected to a login page.

formLogin():
The formLogin(Customizer.withDefaults()) method enables form-based authentication. This will present a default login page when the user is not authenticated and tries to access any protected resource. Spring Security provides a default login page, but you can customize it if needed.

Return the filter chain:
Finally, http.build() constructs and returns the security filter chain for handling authentication and authorization for the rest of the application (outside the authorization server).

Summary
Two Security Chains:

The first chain (authorizationServerSecurityFilterChain) deals with the OAuth2 Authorization Server endpoints, handling token issuance and OpenID Connect. It requires clients to authenticate or provide access tokens for protected endpoints.
The second chain (defaultSecurityFilterChain) handles the rest of the application, requiring users to authenticate and providing form-based login.
Authorization Server Configuration:

You enable OAuth2 functionality using OAuth2AuthorizationServerConfiguration.
OpenID Connect 1.0 is enabled with .oidc(Customizer.withDefaults()).
JWT token validation is enabled using .oauth2ResourceServer(oauth2 -> oauth2.jwt()).
Authentication Handling:

When a user tries to access a protected resource without authentication, they are redirected to the /login page.

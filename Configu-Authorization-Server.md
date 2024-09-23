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

xml
Copy code
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.1.0</version> <!-- Check for the latest version compatible with Spring Boot 6 -->
</dependency>
Next Steps
Customize the RegisteredClientRepository to store clients in a database if needed.
Customize the JWKSource to sign tokens using a stronger key management process in production.

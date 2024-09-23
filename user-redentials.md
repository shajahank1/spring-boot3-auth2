 ## implement user credentials

 To implement user credentials in your Spring Boot OAuth2 Authorization Server, you will allow end-users to authenticate using their username and password. This process involves setting up user details, securing endpoints, and using the Resource Owner Password Credentials grant type to enable password-based authentication.

Step-by-Step Implementation Guide
1. Create a User Entity and Repository
Assume you have a USERACCOUNTS table in your Oracle database with user credentials. First, create an entity class and repository for this table.

UserAccount Entity
```
package om.gov.moh.spring_auth2_server.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "USERACCOUNTS")
public class UserAccount {
    
    @Id
    private Long id;
    private String loginId;
    private String password;
    private Boolean activeYn;

    // Getters and Setters
    // Note: Use Lombok's @Data if you prefer
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getLoginId() {
        return loginId;
    }

    public void setLoginId(String loginId) {
        this.loginId = loginId;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Boolean getActiveYn() {
        return activeYn;
    }

    public void setActiveYn(Boolean activeYn) {
        this.activeYn = activeYn;
    }
}
UserAccountRepository
```
package om.gov.moh.spring_auth2_server.repository;

import om.gov.moh.spring_auth2_server.entity.UserAccount;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserAccountRepository extends JpaRepository<UserAccount, Long> {
    Optional<UserAccount> findByLoginId(String loginId);
}
2. Implement UserDetailsService for Custom User Authentication
Create a class that implements UserDetailsService to load user details from the UserAccountRepository.

```
package om.gov.moh.spring_auth2_server.service;

import om.gov.moh.spring_auth2_server.entity.UserAccount;
import om.gov.moh.spring_auth2_server.repository.UserAccountRepository;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserAccountRepository userAccountRepository;
    private final PasswordEncoder passwordEncoder;

    public CustomUserDetailsService(UserAccountRepository userAccountRepository, PasswordEncoder passwordEncoder) {
        this.userAccountRepository = userAccountRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserAccount userAccount = userAccountRepository.findByLoginId(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with username: " + username));
        
        return User.builder()
                .username(userAccount.getLoginId())
                .password(userAccount.getPassword()) // Ensure this is already BCrypt encoded in your DB
                .roles("USER") // Assign user roles as needed
                .accountLocked(!userAccount.getActiveYn()) // Lock the account if inactive
                .build();
    }
}
```
3. Update Your Security Configuration
Modify SecurityConfig to use your custom UserDetailsService:

```
package om.gov.moh.spring_auth2_server.config;

import om.gov.moh.spring_auth2_server.service.CustomUserDetailsService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configurers.OAuth2AuthorizationServerConfigurer;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint;

@Configuration
public class SecurityConfig {

    private final CustomUserDetailsService customUserDetailsService;

    public SecurityConfig(CustomUserDetailsService customUserDetailsService) {
        this.customUserDetailsService = customUserDetailsService;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class).oidc(); // Enable OIDC if needed
        http
                .exceptionHandling((exceptions) -> exceptions
                        .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/login"))
                )
                .oauth2ResourceServer((oauth2) -> oauth2.jwt());

        return http.build();
    }

    @Bean
    @Order(2)
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        .anyRequest().authenticated()
                )
                .formLogin(Customizer.withDefaults()) // Use form login for user authentication
                .userDetailsService(customUserDetailsService); // Inject the custom UserDetailsService

        return http.build();
    }
}
```
4. Testing User Authentication
Using Postman
URL: http://localhost:9000/oauth2/token (replace with your authorization server URL)
Method: POST
Authorization: Set to Basic Auth with your client credentials (client_id and client_secret).
Body Parameters:
grant_type: password
username: your test username (from USERACCOUNTS)
password: your test password
scope: message.read or any other scope you have defined
You should receive an access_token if the user credentials are correct.

Additional Tips
Encode User Passwords: When creating or updating users, ensure their passwords are encoded using BCryptPasswordEncoder.
Customizing User Roles and Permissions: Modify the roles and permissions based on your requirements when returning UserDetails in CustomUserDetailsService.
This setup will allow you to authenticate users using their credentials through the OAuth2 Authorization Server and issue access tokens using the password grant type.

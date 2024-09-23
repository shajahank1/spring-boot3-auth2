## Fetch Tokens Using JPA
Let’s implement this using JPA to list all active tokens from the OAuth2Authorization table.

Step-by-Step Guide to Fetch Tokens Using JPA
Step 1: Understand the OAuth2Authorization Table Structure
When using JdbcOAuth2AuthorizationService, the OAuth2 authorizations (tokens) are typically stored in a table like oauth2_authorization. This table contains columns for the id, client_id, principal_name, access_token_value, refresh_token_value, etc.

Step 2: Create a JPA Entity for the oauth2_authorization Table
Assuming your database has an oauth2_authorization table, create a JPA entity to represent it:

OAuth2AuthorizationEntity.java
```
package om.gov.moh.spring_auth2_server.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Column;
import jakarta.persistence.Table;
import lombok.Data;

@Entity
@Table(name = "oauth2_authorization")
@Data
public class OAuth2AuthorizationEntity {

    @Id
    private String id;

    @Column(name = "registered_client_id")
    private String registeredClientId;

    @Column(name = "principal_name")
    private String principalName;

    @Column(name = "access_token_value")
    private String accessToken;

    @Column(name = "access_token_issued_at")
    private java.time.Instant accessTokenIssuedAt;

    @Column(name = "access_token_expires_at")
    private java.time.Instant accessTokenExpiresAt;
}
```
Note: Modify the column names (access_token_value, access_token_issued_at, access_token_expires_at) to match the exact column names in your database.

Step 3: Create a JPA Repository
Create a repository interface to interact with the oauth2_authorization table.

OAuth2AuthorizationRepository.java
```
package om.gov.moh.spring_auth2_server.repository;

import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import om.gov.moh.spring_auth2_server.entity.OAuth2AuthorizationEntity;

public interface OAuth2AuthorizationRepository extends JpaRepository<OAuth2AuthorizationEntity, String> {
    List<OAuth2AuthorizationEntity> findByAccessTokenIsNotNull();
}
```
Step 4: Create a Controller to List Tokens
Use this repository to fetch all active tokens in your controller:

TokenController.java
```
package om.gov.moh.spring_auth2_server.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import om.gov.moh.spring_auth2_server.entity.OAuth2AuthorizationEntity;
import om.gov.moh.spring_auth2_server.repository.OAuth2AuthorizationRepository;

import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/tokens")
public class TokenController {

    private final OAuth2AuthorizationRepository authorizationRepository;

    public TokenController(OAuth2AuthorizationRepository authorizationRepository) {
        this.authorizationRepository = authorizationRepository;
    }

    @GetMapping
    public List<TokenResponse> listTokens() {
        List<OAuth2AuthorizationEntity> authorizations = authorizationRepository.findByAccessTokenIsNotNull();

        return authorizations.stream()
                .map(auth -> new TokenResponse(
                        auth.getId(),
                        auth.getRegisteredClientId(),
                        auth.getPrincipalName(),
                        auth.getAccessToken(),
                        auth.getAccessTokenIssuedAt(),
                        auth.getAccessTokenExpiresAt()
                ))
                .collect(Collectors.toList());
    }

    // Token response DTO
    public static class TokenResponse {
        private final String id;
        private final String clientId;
        private final String principalName;
        private final String accessToken;
        private final java.time.Instant issuedAt;
        private final java.time.Instant expiresAt;

        public TokenResponse(String id, String clientId, String principalName, String accessToken, java.time.Instant issuedAt, java.time.Instant expiresAt) {
            this.id = id;
            this.clientId = clientId;
            this.principalName = principalName;
            this.accessToken = accessToken;
            this.issuedAt = issuedAt;
            this.expiresAt = expiresAt;
        }

        // Getters
        public String getId() {
            return id;
        }

        public String getClientId() {
            return clientId;
        }

        public String getPrincipalName() {
            return principalName;
        }

        public String getAccessToken() {
            return accessToken;
        }

        public java.time.Instant getIssuedAt() {
            return issuedAt;
        }

        public java.time.Instant getExpiresAt() {
            return expiresAt;
        }
    }
}
```
Explanation
The OAuth2AuthorizationRepository provides a findByAccessTokenIsNotNull method to find all records with non-null access tokens.
The TokenController fetches these tokens and maps them to a simplified TokenResponse DTO.
Testing the API Endpoint
Start your Spring Boot application.
Access the token listing endpoint: http://localhost:8080/api/tokens
Important Considerations
Security: Ensure that this endpoint is protected, as exposing active tokens might be a security risk. Use annotations like @PreAuthorize("hasRole('ADMIN')") or secure it via your security configuration.
Summary
Since OAuth2AuthorizationService does not provide a findAll method, you interact directly with your database using JPA.
You created an entity and a repository to fetch active tokens from your database and exposed them via a REST controller.

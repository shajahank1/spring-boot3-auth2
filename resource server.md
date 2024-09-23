# resource server 
To build a resource server in Spring WebFlux that connects to an OAuth2 Authorization Server built with Spring Security 6, follow these steps:

1. Add Dependencies
In your pom.xml or build.gradle, include the necessary dependencies:

xml
Copy code
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>
</dependencies>
2. Configure Security for the Resource Server
Create a security configuration class to set up the OAuth2 Resource Server:

java
Copy code
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers("/public/**").permitAll()
                .anyExchange().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt()
            );

        return http.build();
    }
}
3. Configure the Application Properties
In your application.yml or application.properties, specify the connection details for your authorization server's issuer URI:

yaml
Copy code
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080  # Replace with your authorization server URL
4. Implement a Controller (Example)
Create a simple controller to test your resource server:

java
Copy code
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/api")
public class TestController {

    @GetMapping("/secure")
    public Mono<String> secureEndpoint() {
        return Mono.just("This is a secure endpoint!");
    }

    @GetMapping("/public")
    public Mono<String> publicEndpoint() {
        return Mono.just("This is a public endpoint!");
    }
}
5. Run and Test
Start your Authorization Server and Resource Server.
Access http://localhost:8080/api/secure with a valid token issued by your authorization server.
Access http://localhost:8080/api/public without a token to verify open access.
This setup will allow your WebFlux resource server to validate JWT tokens issued by your Spring Security 6 authorization server.

The withDefaultPasswordEncoder() method is indeed deprecated and should not be used in production applications due to security concerns. Instead, you should define a password encoder bean and use it to encode passwords more securely.

Here’s how you can properly implement the UserDetailsService with a modern, secure approach using BCryptPasswordEncoder:

Updated Example with BCryptPasswordEncoder
1. Define a PasswordEncoder Bean
First, define a PasswordEncoder bean in your SecurityConfig class:

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
This BCryptPasswordEncoder will handle encoding and verifying passwords using the BCrypt hashing algorithm, which is widely used and secure.

2. Use the PasswordEncoder in UserDetailsService
Modify your UserDetailsService to use the passwordEncoder bean:

```
import org.springframework.context.annotation.Bean;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class UserDetailsServiceConfig {

    private final PasswordEncoder passwordEncoder;

    public UserDetailsServiceConfig(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails userDetails = User.builder()
                .username("user")
                .password(passwordEncoder.encode("password")) // Encoding the password
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(userDetails);
    }
}
```
Explanation:
User.builder(): Instead of User.withDefaultPasswordEncoder(), use User.builder() to create the user details.
password(passwordEncoder.encode("password")): Use the passwordEncoder bean to encode the password before storing it. This ensures that the password is stored securely.
Summary
withDefaultPasswordEncoder() is deprecated because it uses an insecure, hard-coded password encoder. Always use BCryptPasswordEncoder or another strong password encoding mechanism.
Inject the PasswordEncoder bean into your configuration to properly encode passwords using passwordEncoder.encode("your-password").

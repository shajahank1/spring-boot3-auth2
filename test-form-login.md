## To test the form login functionality with your CustomUserDetailsService and ensure that user authentication works correctly, you can use both manual testing (using a browser) and automated testing (using tools like Postman or JUnit).

1. Testing Form Login Manually Using a Browser
The formLogin(Customizer.withDefaults()) configuration enables a built-in login page provided by Spring Security. Here’s how you can test it manually:

Step-by-Step Manual Testing
Start Your Spring Boot Application Make sure your application is running on a local port (e.g., http://localhost:8080).

Access a Protected Resource

Open a browser and navigate to any endpoint of your application that requires authentication, e.g., http://localhost:8080/protected-resource.
If the endpoint is protected, Spring Security will redirect you to the default login page (/login).
Log in Using Your User Credentials

Enter the username and password that exist in your USERACCOUNTS table.
Click the "Sign In" button.
Check the Result

If the credentials are valid, you should be redirected to the original protected resource.
If the credentials are invalid, you should see an error message indicating authentication failure.
2. Testing Form Login Using Postman
You can also test the login functionality using Postman to simulate a form submission.

Step-by-Step Testing Using Postman
Open Postman and Create a New Request

Set the request method to POST.
Set the URL to http://localhost:8080/login (or whatever your server URL is).
Set the Request Body

Choose the x-www-form-urlencoded option in the body.
Add the following key-value pairs:
username: your test username (e.g., testuser)
password: your test password (e.g., password)
Send the Request

Click the "Send" button.
If the login is successful, you should receive a 200 OK status code along with the default home page HTML.
If login fails, you’ll see an error message with a 401 Unauthorized status.
3. Automated Testing Using JUnit and Spring Security Test
You can write integration tests using JUnit and Spring Security's testing utilities to verify that form login works with your CustomUserDetailsService.

Add the Required Dependencies to pom.xml
Make sure you have the Spring Security test dependency:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```
Write a JUnit Test Case
Here’s a sample test case to validate form login using Spring Security's MockMvc:

```
package om.gov.moh.spring_auth2_server;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestBuilders.formLogin;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
public class FormLoginTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testValidLogin() throws Exception {
        mockMvc.perform(formLogin().user("validUsername").password("validPassword"))
                .andExpect(status().is3xxRedirection()); // Expect a redirection after successful login
    }

    @Test
    public void testInvalidLogin() throws Exception {
        mockMvc.perform(formLogin().user("invalidUsername").password("wrongPassword"))
                .andExpect(status().is4xxClientError()) // Expect a client error (401) on invalid login
                .andExpect(MockMvcResultMatchers.forwardedUrl("/login?error"));
    }
}
```
Explanation of the Test Cases
testValidLogin: This test simulates a form login using valid credentials. If authentication is successful, a redirect response (3xx status code) is expected.
testInvalidLogin: This test uses invalid credentials, and a 401 Unauthorized status is expected, indicating a failed login attempt.
Summary
Use a browser to test the default form login page by accessing a protected resource.
Use Postman to simulate form submissions by sending x-www-form-urlencoded POST requests to /login.
Write automated tests using JUnit and Spring Security Test to validate the login behavior programmatically.
This approach ensures that your CustomUserDetailsService is correctly integrated and that form-based authentication is functioning as expected. Let me know if you need further assistance!

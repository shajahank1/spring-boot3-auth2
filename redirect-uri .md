# redirect-uri 
http://127.0.0.1:8080/login/oauth2/code/my-client with your redirect_uri.  how to make this redirect-uri ,  can i make it the controller in same auth-server?

Yes, you can configure the redirect_uri to point to a controller within the same authorization server application if that's what you need. This can be useful if you want to handle the redirection logic or if you're building a user-facing application that needs to process the authorization code.

Steps to Set Up a Custom Redirect URI in Your Authorization Server
Register the OAuth2 Client with your desired redirect_uri.
Create a Controller in your authorization server to handle the redirection.
Handle the OAuth2 Authorization Code Flow in your controller.
Step 1: Register the OAuth2 Client with the Custom Redirect URI
Make sure your OAuth2 client is registered with the redirect_uri pointing to your desired controller endpoint. For example, let's say you want your redirect_uri to be http://localhost:8080/authorized.

SQL to Register the Client
```
INSERT INTO oauth2_registered_client (
    id, client_id, client_secret, client_id_issued_at, client_name,
    client_authentication_methods, authorization_grant_types, redirect_uris,
    scopes, client_settings, token_settings
) VALUES (
    '1', 'my-client', '{noop}secret', CURRENT_TIMESTAMP, 'My Client',
    'client_secret_basic', 'authorization_code,refresh_token',
    'http://localhost:8080/authorized',
    'message.read,message.write',
    '{"requireAuthorizationConsent":true}',
    '{"accessTokenTimeToLive":3600, "refreshTokenTimeToLive":86400}'
);
```
Step 2: Create a Controller to Handle the Redirect
Create a Spring Boot controller in your authorization server application that handles the redirection after the user is authenticated.

Example Controller
```
package om.gov.moh.spring_auth2_server.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class OAuth2RedirectController {

    @GetMapping("/authorized")
    public String handleRedirect(@RequestParam(name = "code") String authorizationCode) {
        // Here, you can handle the authorization code however you need to
        System.out.println("Received authorization code: " + authorizationCode);

        // You might want to process this authorization code further
        // For now, just display a confirmation page or log the code
        return "authorized"; // This returns an HTML view named 'authorized.html' (you can customize this)
    }
}
```
In this example:

The @GetMapping("/authorized") endpoint matches the redirect_uri you registered.
When the user is redirected to this endpoint with an authorization code, you can process the code as needed.
Step 3: Create a View (Optional)
If you want to display a success page, create an authorized.html file in src/main/resources/templates if you're using Thymeleaf as a template engine.

Example authorized.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Authorization Successful</title>
</head>
<body>
    <h1>Authorization Successful!</h1>
    <p>You have successfully authorized the client application.</p>
</body>
</html>
```
How This Works
When the user completes the OAuth2 authorization flow, the Spring Authorization Server redirects them to the redirect_uri specified by your registered client (e.g., http://localhost:8080/authorized?code=AUTH_CODE).
The OAuth2RedirectController handles this request and can further process the authorizationCode.
Testing the Flow
Start Your Authorization Server.
Initiate the OAuth2 Authorization Request:
```
http://localhost:8080/oauth2/authorize?response_type=code&client_id=my-client&redirect_uri=http://localhost:8080/authorized&scope=message.read
```
Login and Authorize the client.
You should be redirected to http://localhost:8080/authorized?code=AUTH_CODE, and the OAuth2RedirectController should handle this.
Summary
You can handle the redirect_uri within the same Spring Boot application by registering an endpoint (@Controller) that matches the redirect_uri.
This approach lets you control how the authorization code is handled, either by displaying a success message or further processing the authorization code.

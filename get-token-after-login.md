After successfully logging in using the form-based login, you need to obtain an access token. Since you're working with an OAuth2 Authorization Server, the typical flow to get a token involves using one of the OAuth2 grant types, such as the authorization_code grant.

Here's how you can obtain an access token after logging in:

Step-by-Step Guide to Obtain a Token Using the Authorization Code Grant Flow
1. Overview of the Authorization Code Grant Flow
The authorization_code flow is commonly used for web applications and involves the following steps:

User Login: The user logs in via a login page.
Authorization Code: The user is redirected to your application's redirect URI with an authorization code.
Exchange Code for Token: Your application exchanges the authorization code for an access token.
Step 1: Initiate the Authorization Request
Open your browser and visit the following URL:

```
http://localhost:9000/oauth2/authorize?response_type=code&client_id=messaging-client&redirect_uri=http://localhost:8080/authorized&scope=message.read
```
Replace:

localhost:9000 with your authorization server URL.
messaging-client with your client_id.
http://localhost:8080/authorized with your desired redirect_uri.
The browser will redirect you to the login page. Enter your credentials (username and password) managed by your CustomUserDetailsService.

After successful login, you will be redirected to http://localhost:8080/authorized?code=AUTH_CODE where AUTH_CODE is a temporary authorization code provided by the authorization server.

Step 2: Exchange Authorization Code for Access Token
To exchange the authorization code for an access token, you can use Postman or any HTTP client.

Using Postman to Exchange the Code
Open Postman and create a new POST request to the token endpoint:

```
http://localhost:9000/oauth2/token
```
Set Authorization:

Choose Basic Auth.
Enter client_id (messaging-client) as the Username.
Enter client_secret (secret) as the Password.
Set the Body Parameters:

Choose x-www-form-urlencoded as the body type and set the following key-value pairs:
grant_type: authorization_code
code: AUTH_CODE (Replace this with the actual authorization code you received)
redirect_uri: http://localhost:8080/authorized (The same URI you used in the initial request)
Send the Request

If everything is set up correctly, you will receive a response containing an access_token, refresh_token, and other OAuth2 token details.

Example Response
```
{
   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
   "token_type": "Bearer",
   "expires_in": 3600,
   "refresh_token": "dcb6fc76-e9f8-44a9-8e79-818f481cde4d",
   "scope": "message.read"
}
```
Step 3: Use the Access Token to Access Protected Resources
Once you have the access token, you can use it to make authorized requests to your protected APIs.

Example Using Postman
Create a new GET request to your protected resource endpoint, e.g., http://localhost:9000/protected-resource.
In the Authorization tab, select Bearer Token and paste your access_token.
Testing With cURL
You can also test using cURL. For example:

```
# Step 1: Get Authorization Code
curl -X GET "http://localhost:9000/oauth2/authorize?response_type=code&client_id=messaging-client&redirect_uri=http://localhost:8080/authorized&scope=message.read"

# Step 2: Exchange the code for an access token
curl -X POST "http://localhost:9000/oauth2/token" \
     -H "Authorization: Basic $(echo -n 'messaging-client:secret' | base64)" \
     -d "grant_type=authorization_code&code=AUTH_CODE&redirect_uri=http://localhost:8080/authorized"
```
Summary
Use the authorization_code grant type to obtain an access token after logging in via the form-based login.
Exchange the authorization code for an access token using the /oauth2/token endpoint.
Use the obtained token to access your protected resources.
This method follows OAuth2 best practices and ensures that your access token is securely obtained and used. Let me know if you have any further questions or need assistance!

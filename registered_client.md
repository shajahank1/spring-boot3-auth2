# registered_client and user_account Tables

Yes, you should create the registered_client table in your Oracle database, as it serves a different purpose from your user_account table. Here's an explanation of the difference between the two tables and why you need the registered_client table.

Difference Between registered_client and user_account Tables
1. registered_client Table
The registered_client table is used to store information about OAuth2 clients that are allowed to connect to your Authorization Server. These clients represent applications (e.g., web apps, mobile apps, backend services) that need to obtain an access token from the Authorization Server to access protected resources on behalf of users or for their own use.

Key Information Stored in registered_client Table:
Client ID: A unique identifier for each OAuth2 client.
Client Secret: A secret key used to authenticate the client (in confidential clients).
Redirect URIs: The allowed URLs where the authorization server can redirect the client after authorization.
Grant Types: The OAuth2 grant types supported (e.g., authorization_code, client_credentials).
Scopes: The permissions the client is requesting.
Token Settings: Access token validity, refresh token validity, etc.
In summary, this table contains the information necessary to manage and control access for different applications (clients) using OAuth2.

2. user_account Table
The user_account table is used to store information about end-users who log in to your system. These are typically the actual human users who want to access protected resources via the OAuth2 clients.

Key Information Stored in user_account Table:
User ID: A unique identifier for each user.
Login ID / Username: The user's login credentials.
Password: The encoded password for authentication.
Status: Whether the user is active or inactive.
Other User Information: Details such as user profile information.
Why You Need Both Tables:
The registered_client table manages application-level authentication and authorization. It controls which applications are allowed to interact with the authorization server.
The user_account table manages user-level authentication and authorization. It handles the end-user login process.
How They Work Together:
When a client (e.g., a mobile app or a web app) tries to access your API, it authenticates itself with the Authorization Server using the credentials stored in the registered_client table.
Once the client is authenticated, it may prompt the user to log in. The user's login credentials are verified against the user_account table.
Upon successful authentication, the Authorization Server issues an access token, allowing the client to access protected resources on behalf of the user.
Example Use Cases:
A web application wants to access protected APIs from your resource server. It must be registered as a client, with its details stored in the registered_client table. When the web app initiates the OAuth2 flow, the authorization server checks this table to validate the client.
Once the client is validated, the web application prompts the end-user to log in. The user's credentials are checked against the user_account table, and if valid, an access token is issued.
How to Create the registered_client Table in Oracle
You can create the registered_client table with the following SQL script:

```
CREATE TABLE registered_client (
    id VARCHAR2(36) PRIMARY KEY,
    client_id VARCHAR2(50) UNIQUE NOT NULL,
    client_secret VARCHAR2(100) NOT NULL,
    redirect_uris CLOB,
    scopes CLOB,
    authorization_grant_types CLOB,
    client_authentication_methods CLOB,
    token_settings CLOB,
    client_settings CLOB
);
```
Adjust the data types and lengths according to your requirements.
Store redirect_uris, scopes, authorization_grant_types, and other complex fields as CLOB (Character Large Objects) since they might store multiple values.
Summary
You need the registered_client table to manage the OAuth2 clients (applications) that will interact with your authorization server.
The user_account table handles the actual user login and authentication process.
They serve different roles in the OAuth2 architecture, working together to provide secure authorization and authentication.

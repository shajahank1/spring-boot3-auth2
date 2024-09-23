## default user name and password for form login

By default, Spring Boot automatically generates a username and password for form login when the application starts, but only if no user credentials have been explicitly defined in the security configuration.

Default Credentials
Username: "user"
Password: Spring Boot generates a random password at runtime and prints it in the console.
For example, after starting the application, you might see a log message like this:

```
Using generated security password: b1838cf6-736d-4a67-83e8-d989260de798
```
This password (b1838cf6-736d-4a67-83e8-d989260de798 in this example) will be different every time the application starts.

To Customize the Username and Password
If you want to set a custom username and password for form login, you can define it in your application.properties or application.yml file:

In application.properties:
properties
```
spring.security.user.name=admin
spring.security.user.password=secret
```
In application.yml:
yaml
```
spring:
  security:
    user:
      name: admin
      password: secret
```
With this configuration, your username will be admin and your password will be secret, replacing the default credentials.

If You’re Using Custom User Details
If you set up custom user authentication (e.g., from a database), you can provide your own user credentials and won't rely on Spring's default user.

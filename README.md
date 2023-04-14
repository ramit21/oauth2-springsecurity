# oauth2-springsecurity

In this project, we setup an Authorization server (Keycloak), Resource server (Springboot app), and a Client (Postman)
to connect via Oauth2 flow.

## Project setup
Instead of building OAuth servers from scratch, companies prefer to use readymade options like Keycloak, Okta, Cognito etc.

Download Keycloak from https://www.keycloak.org/downloads and see instructions to 
start the server using steps given for bare metal setup on our laptop: https://www.keycloak.org/getting-started/getting-started-zip

Make sure JAVA_HOME is pointing to JDK 11.

Command to start local keycloak server on local:
```
cd bin
kc.bat start-dev --http-port 8180
http://localhost:8180/
```
Create a admin user and password (anything like admin/admin), and login using these credentials.

Next create a realm. Every project/environment etc is given its own realm.

Under the realm, create a client. Choose Client-type = 'OpenID Connect', client-id='eazybankapi' (any name).
Click next, enable Client authentication. 
For AuthN flow, select 'Service account roles', this is when we want to use 'Client Credentails Grant' for this client.
Save the client, go to credentials tab for the client, and get the client secret.
On Roles tab, create couple of roles like ADMIN, DEV, and assign it to the client under Service accounts roles tab.

If you hit this endpoint on the keycloak server, it will show all important parameters like token_endpoint, and jwks_uri.

http://localhost:8180/realms/eazybankdev/.well-known/openid-configuration

Take the token endpoint, and make the call from postman, along with client_id, client_secret, scope=openid email (etc), and grant_type=client_credentials. In response you will get the access_token as well as the id_token. Put the access token in jwt.io, and see the roles under "resource_access" section, it will contain roles assigned above.

When you start the application, from postman make a call to an api endpoint, give the access token in header as Authorization= Bearer <token>.



## Things to notice in code
1. Spring boot project pom has a dependency for spring-boot-starter-oauth2-resource-server.

2. Fetch all roles present in the JWT token's clains section in a Role converter, and register it with spring security configs:
https://github.com/ramit21/oauth2-springsecurity/blob/136e478af6a43aa0e63ae6e35a90185d6ade6714/src/main/java/com/eazybytes/config/KeycloakRoleConverter.java#L17
https://github.com/ramit21/oauth2-springsecurity/blob/c848c3033430c0bf37baa043a754abf79fb81771/src/main/java/com/eazybytes/config/ProjectSecurityConfig.java#L29

3. How does Spring application know what are the details of the keycloak server? In application.properties, we give the path to Keycloak server's certificates. The resource server (spring app) will download the public certificate from this url. The Oauth server (keycloak) will use its private key to encrypt the tokens and the spring app will use the public certificate to decode it.
https://github.com/ramit21/oauth2-springsecurity/blob/c848c3033430c0bf37baa043a754abf79fb81771/src/main/resources/application.properties#L7
This path is the one that appears as jwks_uri on server config url as mentioned above: http://localhost:8180/realms/eazybankdev/.well-known/openid-configuration

4. In spring sec config, we can check for certain roles for a particular api. If the jwt token does not have these roles (when roles were not configured for the client used to get the token), then access to this endpoint would be denied.
https://github.com/ramit21/oauth2-springsecurity/blob/c848c3033430c0bf37baa043a754abf79fb81771/src/main/java/com/eazybytes/config/ProjectSecurityConfig.java#L49


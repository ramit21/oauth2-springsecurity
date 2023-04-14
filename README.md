# oauth2-springsecurity

## OAuth Server setup using Keycloak
Instead of building OAuth servers from scratch, use readymade options like Keycloak, Okta, Cognito etc.

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

## Things to notice in code
1. Spring boot project pom has a dependency for spring-boot-starter-oauth2-resource-server.
2. Fetch all roles present in the JWT token's clains section:

https://github.com/ramit21/oauth2-springsecurity/blob/136e478af6a43aa0e63ae6e35a90185d6ade6714/src/main/java/com/eazybytes/config/KeycloakRoleConverter.java#L17

3. 


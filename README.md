# oauth2-springsecurity

This project has 2 sub projects, one showing backend channel OAuth flow using springboot application as the 'resource server' and Postman as the 'Client', and another showing frontend channel using angular.

We use KeyCloak to setup our Authorization server.

Instead of building OAuth servers from scratch, companies prefer to use readymade options like Keycloak, Okta, Cognito etc.

## Project setup
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

Click next, enable 'Client-authentication' -> as first we want the client app to prove its own identity.
 
For AuthN flow, select 'Service account roles', this is when we want to use 'Client Credentails Grant' for this client.
Save the client, go to credentials tab for the client, and get the client secret.
On Roles tab, create couple of roles like ADMIN, DEV, and assign it to the client under Service accounts roles tab.

If you hit this endpoint on the keycloak server, it will show all important parameters like token_endpoint, and jwks_uri.

http://localhost:8180/realms/eazybankdev/.well-known/openid-configuration

Take the token endpoint, and make the call from postman, along with client_id, client_secret, scope=openid email (etc), and grant_type=client_credentials. In response you will get the access_token as well as the id_token. Put the access token in jwt.io, and see the roles under "resource_access" section, it will contain roles assigned above.

When you start the spring boot application, from postman make a call to an api endpoint, give the access token in header as Authorization= Bearer <token>.

## Get authorization code, and use it to get the access token.
Create a new client (easybankclient) with Authentication flow selected as 'Standard flow' for Authorization code flow. Do give redirect url in this case, as it will be a front channel request.

Next create a User in the OAuth server, and in credentials tab, set the password for this user.

Now first you need to make a call to /auth endpoint of oauth server to get the auth token, and then use this token in the post call to the /token endpoint to get the token. (see different endpoints in the postman collection attached). 

In the postman, for the Get call for auth code, notice the 'response_type=code'. Take the url formed using the header values form the postman and paste it in the browser. When you hit it, it will give you a password login page, enter the details of the user created above. Once you login, you will be redirected to the redirect uri, with other paramters in the redirect url like the state (sent earlier) and the code.

Now take this code, and call the post to /token endpoint, with 'grant_type=Authorization_code'. Now you will get the access token using which you can invoke backend endpoints.


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

5. Keycloak dependency in the angular app. This library contains all code required for PKCE code challenge handling, redirecting user to auth server login page etc.:
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/package.json#L26
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/app.module.ts#L17
Initialize Keycloak with PKCE params:
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/app.module.ts#L62
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/app.module.ts#L19
KeyCloak login action:
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/components/header/header.component.ts#L33

6. All routes are protected using a component that extends KeyCloakAuthGaurd:
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/app-routing.module.ts#L23

The AuthKeyClockGaurd component has logic to check the this.authenticated variable which is set by the library on successful authentication:
https://github.com/ramit21/oauth2-springsecurity/blob/7b9d6caecfd6c830ca62b9faa6cf478cfa069e90/bank-app-ui/src/app/routeguards/auth.route.ts#L29



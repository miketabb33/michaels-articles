
This article is based on Aaron Parecki’s Udemy's course: “The Nuts and Bolts of OAuth 2.0” [Course Link](https://www.udemy.com/course/oauth-2-simplified/). I highly recommend this course!

## Section 1: Welcome

OAuth and Open ID services typically live in their own server called the Authorization Server, separate from Resource Servers.

#### OAuth

The main responsibility of OAuth is to provide a token used for accessing APIs. Think of this hotel room key scenario, the key gives access to your room, pool, and business center. However, it does not give access to the conference room or offices, additional privileges are required.

#### Open ID
The main responsibility of Open ID is to provide the users identity, an extension of OAuth.

## Section 2: API Security Concepts

### OAuth Roles (Terms)
1. User (Resource Owner)
2. Device (User Agent)
3. Application (Client)
4. Authorization Server
5. API (Resource Server)

### Application Types
#### Confidential Clients: Have Credentials

Think of a 1st party API or similar. The main quality of a confidential client is that the source code is not viewable.

Confidential credential examples:
- Client secrets
- Private Key JWT
- mTLS

#### Public Clients: No Credentials
Think of an SPA or mobile app. There is no way to ship a secret to the client that the user controls, and it stays a secret. The source code can be viewed.

The authorization server can’t be sure if requests are genuine or being made by someone mimicking the client. This is true with mobile app because the binary files can be inspected.

### User Consent
User consent is a critical part of a 3rd party authorization flow because it ensures that the user is in front of the keyboard explicitly consenting to access.

Internal authorization and user consent typically occurs on a page that lives in the authorization server, which subsequently provides an access token.

- For internal authorization, think of Googles sign in page.
- For 3rd party user consent, think of sign into "some application" with Facebook.

### How OAuth Sends Data
#### Back Channel
The secure way of delivering an access token. It uses a client to server HTTPS connection and is similar to hand-delivering a package, you see the recipients.

1. Certificate validation
2. Encryption
3. The response can be trusted because you know where it came from.

The Back Channel does not mean back end. A JavaScript app can use fetch or AJAX and that would be considered Back Channel because the JavaScript is handling the HTTP request directly.

#### Front Channel
Uses the address bar to move data between 2 systems. It’s similar to using a package delivery service to deliver a package.

1. There’s no direct link between the Application and the Authorization Server.
2. Was the data intercepted?
3. From the sender’s perspective: did the data get to the intended recipient?
4. From the receiver’s prospective: did the data come from a legitimate source?

#### Channel Usage
The end goal is for the Application to get an access token from the Authentication Server and the most secure way is the back channel. However, when we want the user to give consent to provide an access token, we need to use the Front Channel.

#### Implicit Flow
Uses the Front Channel for both the Applications authentication request and the Authentication Servers response.

First, the Application makes a request to the Authorization Server via the address bar. To Application includes information in the query string, none of which is particularly sensitive or secret, like:
1. Who the app is (Client Id)
2. What it’s trying to do
3. Requested scopes

Then, after user sign in, the Authentication Server redirects back the the Application via the address bar. The redirect url will provide the access token as a query string, which is where the security vulnerability comes in. This flow is not secure because the Authorization Server is unable to reliably know for sure that they are providing an access token to a trusted Application.


### Application Identity
Client ID: The application's OAuth identifier.

Client Secret: The application's password.

#### Authorization Code Flow
Similar to Implicit flow, expect, the access token is delivered in the back channel. The authorization server does not send the access token in the redirect url, instead, it sends a 1 time use Authorization Code with a short expiration time. Next, the application verifies the Authorization Code with the authentication server through the back channel, by using the Client Secret. In exchange, the authorization server sends the access token to the application. However, mobile and SPA apps can’t be deployed with a Client Secret…  This is where PKCE comes in. (Proof Key for Code Exchange).

1. The Authentication Server makes a unique secret for each request, to be used when the Application redeems the Authorization Code.
2. PKCE alone does not prevent someone from imitating a client app, all the information is public.
3. The redirect URI is really the only thing that can be used to verify the identity of the application. However, this is not 100% because app there is no global registration for custom URL schemes, duplicates are possible.
4. When using PKCE, it’s important to register approved application URL’s in the authentication server and confirming with the Front Channel’s redirect URL. Don't allow the Authorization Server to redirect if the redirect URL is not registered.

The bottom line is that there really is no perfect solution for mobile and SPA, client secrets are the best way to verify Authorization Codes. However, for mobile and SPA's, PCKE is the recommend approach and protects against an authorization code injection attack.

## Section 3-6: Server Side, Native, and SPA’s
### Server Side
1. Server Side apps are confidential clients and can use client secrets
This basic idea of this flow can be used both mobile and JavaScript apps.

### Native
2. Native are public client and should use the PKCE extension.

### SPA
1. JavaScript app’s can securely obtain the access token using the PKCE extension.
2. Storing the access token is not safe.
3. Any scripts run in the browser can access the same storage locations as the code:
    1. Code runtime scripts (hot linking)
    2. Scripts the users install on their browser.
4. The recommend approach for JavaScript apps is to deny it access to the access token.
    1. Use a dynamic backend server which serves the client code and acts as a proxy for all API calls.
    2. Use the front channel to get the authorization code.
    3. The browser sends the authorization code to a dynamic backend server, which then interacts with the authorization server to obtains the access token.
    4. The browser uses an HTTP only session cookie that maps to the access token in the proxy backend.

## Section 7: Internet of Things
1. IoT don’t always have keyboards, and when they do, it’s typically not ideal for the user.
2. There typically isn't always a browser to use a front channel to have the user login.

### Device Flow
1. Separates the device that gets the access token from the device that the user is using to login.
2. Device says to auth server, someone is trying to log in. The auth server only knows the client ID.
3. The auth server sends up a temporary code and the device asks the user to go to a URL and enter the code. The URL is typically short because the user will need to type it in by hand.
4. The URL takes the user to the authentication server, where the user can type in the code and login/authenticate.
5. Once the user is logged in and authenticated, they see a prompt stating “Another device is trying to access your account, would you like to allow it?” They hit yes.
6. During this time, the device requesting an access token is pooling every couple of seconds asking, “Is the user finished logging in yet?”
7. Once the user logs in, one of these pooled requests will return with the access token.
8. Complete!

## Section 8: Client Credential Flow
### Client Credential Grant
- Gives a way for the client to get an access token without any user interaction.
- Machine to machine communication.
- Exchanging the client credentials for an access token simplifies the APIs, so they only need to worry about validating access tokens.
- APIs won't need to know anything about client credentials.

### Steps
1. Register the application at the OAuth server.
    1. If asked for an option - go with machine to machine or service account.
    2. Essentially, we need the client secret.
2. Post request to the auth server has a grant type of client credentials.
    1. The details may vary.
1. The response should return an access token.

## Section 9: Introduction to OpenID Connect
- The ID token are always JWTs, access tokens have no defined format in the spec.
- JWTs are broken into 3 parts:
    1. Header: talks about the token, like which signing algorithm was used.
    2. Payload: Contains the data, like user ID, email, other profile info, etc.
    3. Signature: how the token validated.
- JWTs are base 64 encoded.
- The ‘sub’ value in the JWTs payload is the user’s identifier.It's the users unique id for the system.

### ID Tokens vs Access Tokens
1. They both can be JWTs and have the same format.
2. However, they are intended to serve 2 very different purposes.
    1. Access Token: is what the application gets to make requests to APIs. The access token is meant to be opaque, it should be like a hotel key, not knowing anything about the person using it.
    2. ID Token: is meant to be read by the application, like a passport of the user. It should be unpacked.

### Obtaining an ID Token
1. Applications typically want both the access token and id token.
2. hen using the Authorization Code Flow:
    1. Add OpenID scope to the request, and you'll get back an ID Token and Access Token.
    2. Doesn’t need to verify the token with the signature, because we securely used the back channel.
3. Response type ID token (Like implicit flow):
    1. Does not return an access token.
    2. Returns an ID token in the redirect URL instead of the authorization code… or access token for implicit flow.
    3. We can use the JWT signature to validate the ID Token.
    4. Front channel is not too secure because if anyone is snooping, they will be able to get data like email address, address, or any other data included in the ID Token.
4. Just using the OpenID scope will return bare minimum data in the ID Token.
    1. Adding new scopes (like name, address, etc.) to the request will add additional information to the JWT.
    2. Some servers do not provide additional info in the ID Token, in those cases, use a user info endpoint.

### Hybrid Open ID Connect Flows
1. Essentially, combining ID Token and Authorization Code response types
    - Ex: response_type=code+id_token
2. While it’s okay to combine Authorization code and ID Token, it's bad practice to combine front channel access token with ID Token because of access token leakage.
    - Ex: response_type=token+id_token
3. Using id_token+code
    - We get the unverified ID Token in the front channel, but we get the access token later in the flow by exchanging the authorization code in the back channel.
    - The ID token needs to be verified.
4. The simplest and most secure solution is not to use Hybrid Flows.
    - Instead, get both ID Token and Access Token with PKCE via the backend channel by adding open_id to the scope.

### Validating and Using an ID Token
1. Any ID Token is received via the front channel absolutely needs to be validated.
2. Validate the JWT signature.
    1. Determine which key to use.
    2. Use a library to validate the token.
    3. Now, validate the claims,
        1. is the issuer correct?
        2. Is the audience correct?
        3. Are the timestamps correct?
        4. Double-check the nonce value.
    4. Further claims can be validated
        1. The amr, how the user logged in.
        2. The auth time, when was the last time the user actually signed in.
3. The only time we do not need to validate the ID Token is at the time it comes from a trusted source, like in the back channel during the authorization code flow.
    1. Storing an ID Token in the browser, like a cookie, requires that the ID Token be validated before being used.



## Section 11: Access Token Types And Their Tradeoffs
### Reference Tokens Vs Self-Encoded Tokens
What kind of token data is stored?
- User
- Application
- Expiration
- Time Created
- Scopes
- Authorization Server
- Last Login Time

### Reference Token:
- Long random string of characters that doesn't mean anything, it's a pointer.
- Token Data is stored in a db table or something like Redis.
- The validation happens at the Authentication Server.
- Shines when the Authentication Server is within the application.

### Self-Encoded Token:
- The token string itself contains some kind of data.
- Puts the token data in the Self-Encoded Token, typically as a JWT token.
- Only the API should inspect the token, the client should only use the token like the hotel key analogy.
- The validation happens outside the Authentication Server.
- Shines when the Authentication Server is NOT within the application.
- Scalable

### Pros and Cons of Reference Tokens
Pros:
- Simple
- Easy to manage, add and delete entries
- Can point to data that isn’t visible to the application or the API, like sensitive data. Good for hiding data from employees and 3rd party applications.

Cons:
- The data needs to be stored. The token means nothing and if there isn’t an entry then the token is invalid.
- Storage scalability.
- APIs are forced to look up data when validating the access token.

### Pros and Cons of Self-Encoded Tokens
Pros:
- The token has all the data required to validate it.
- Don’t need to be stored.
- Don’t need to be looked up
- Scalable, because no shared storage is needed.

Cons:
- JWTs are typically not encrypted, it’s public data.
- No way to revoke the token until the token expires.

## Section 12: Access Token Types And Their Tradeoffs
### The Structure of JWT Access Tokens
- Base 64 encoded.
- Typically, not encrypted, but signed.
- Split into 3 sections, by the ‘.’
    1. Header
        1. Talks about the token
        2. Which signing algorithm was used
        3. Which key was used to sign it.
    2. Payload
        1. The data we care about
        2. Claims, ways to validate that the token has not been tampered with.
            1. iss: identifies the issuer
            2. exp: expiration
            3. iat: issued at
            4. aud: audience the token is intended for.
            5. sub: ID for whom the token represents, like user ID.
            6. client_id: the client ID
            7. jti: unique ID for the particular JWT token
    3. Signature

### Remote Token Introspection
APIs need to validate tokens, they can use Remote Token Introspection.
- A means to verify the token with the authorization server.
- New endpoint (HTTP POST): Token Introspection Endpoint
    - The endpoint most often required some kind of authentication of the call.
- The endpoint will respond with the is-valid status.
- CON: This requires network traffic and can be expensive.

### Local Token Validation
APIs need to validate tokens, they can use Local Token Validation.
- The fast way of validating a token, because it does not require any network traffic.
- The best approach is to use a library for everything in validating a token locally.
- Steps:
    1. Base 64 decode the JWT.
    2. Match the public key in JWT header with the authorization server's public key.
    3. Validate the signature (use library).
    4. Validate the claims.
        1. Issuer
        2. Audience
        3. Expiration
        4. Issued At
        5. Custom claims like - Scopes, user groups, etc.
- CON: Local validation only validates the token in the state that it was in when issued.
    - If the token was invalidated before the expiration date, the token will not reflect that change.
    - The longer the token lifetime, the higher the risk of making auth decisions on out of date information.

### The Best Of Both Worlds: Using An API Gateway
First line of defense, handles every API request that comes through.
1. API gateway performs ONLY local validation, which thins out the amount of requests coming through the system by removing attackers request and bad tokens.
2. As a result, the services behind the gateway only get valid or potentially valid tokens (they could have been invalidated before the expiration).
3. The services determine whether remote token introspection is required or not.
4. For example, non-sensitive API requests most likely won’t need to use remote token introspection, but something like change password will.

API gateways provide the benefits of both worlds.

## Section 13: Choosing Token Lifetimes
### Increasing Security With Short Token Lifetimes
Local Token Validation is not able to detect if a token has been invalidated before its expiration.

- Making a tokens' lifetime shorter is a way to decrease the risk of using an invalid token to authenticate.
- Shortening the token lifetime also strengthens security against token leaks. Example: from an attack or an accidental log.

### Improving User Experience With Long Token Lifetimes
- Short access tokens can improve security, but It also has a negative effect on user experience.
- Access tokens that never expire are typically a bad idea, because of leaks or attacks.
- Applications can use the refresh token after an access token has expired, causing the user's request to take a little longer.
- If an app doesn’t have/support a refresh token, it will need to go back to the authorization server to get a new access token. This process can be so fast that it may be unnoticeable, but it is still a full page redirect.
    - Mobile apps are typically more disruptive because they need to open a browser.
- In most cases, the more disruptive it is to get an access token, the longer the token lifetime should be.
    - In web: refresh tokens are typically not secure and are not used. Access token lifetimes can be longer.
    - In mobile: refresh tokens can securely be used and opening the browser is very disruptive. Shorter token lifetimes with a refresh token can be used. Refresh tokens should be long enough for the user experience.
- The authorization server's session lifetime can be much longer than a specific access/refresh tokens lifetime

### Contextually Choosing Token Lifetimes
- Access token lifetimes can vary based on different properties, like client ID, scope, group, user, etc.
A- n SPA can have longer token lifetimes than its mobile app counterpart.
- An Admin/staff may have a shorter token lifetime than other users.
- Using scopes, an app may not require a new login while viewing a catalog, but can require a login when the user wants to place an order.
    - The regular token can be long-lived.
    - The checkout token can be short-lived.

## Section 14: Handling Revoked Or Invalidated Access Tokens
### Reasons Why An Access Token May Become Invalidated
- Account Deactivation
- User Revokes Application
- Change Password
- OAuth Config Changes

On the App side: If the request is rejected due to an invalid access token, the app can try and use the refresh token or send the user back to the authorization server.

On the API side:
- The client may not know when the token has expired, clients are not supposed to read an access token.

### The Problem With Local Validation
- It’s the APIs job to make sure it does not respond if an access token has been revoked.
- Local Validation 100% depends on the data in the JWT to know if a token is valid. If a token becomes invalid due to a reason other than expiration, the JWT will not know about it. Only Remote Token Introspection can the real time status of an access token.

### Token Lifetime Considerations
- Tokens with long lifetimes need Remote Token Introspection more often because there is more of a chance that the token has become invalid between issue time and expiration time.
- Shorter token lifetimes have less of a chance that the token was invalidated between issue time and expiration time, the time is shorter.

### How Applications Can Revoke Access Token
- A token should be invalidated on Logout.
- An application can use the revocation endpoint to tell the authorization server that the access token should be invalidated.

## Section 15: OAuth Scopes
### The Purpose Of OAuth Scopes
Scopes are a way for an application to request access to limited parts of a user account.
- Scopes allow for segmenting access to data.
- Scopes are not/should not be used for permissions.
- Scopes are simply used to limits what an access token can do, its not a permission system.

### Defining Scopes For Your API
- There are no rules and little guidance.
- Scopes will be included in the access token.

Keep in mind, scopes are just a string. It's up to the API’s to know which scopes are required by each of its methods, and enforce the scopes.

Scopes are just strings and can be defined in format as, but are not limited to:
- upload
- photos:upload
- example.com/photos/uploads

Once scopes are defined, document them so that developers know what the available scopes.

### When to use Scopes:
- Bare minimum scopes:
    - Read
    - Write
- Restricting access to sensitive information
    - Personal Information
    - Account details
    - Etc
- Segment out unrelated parts of a large API, Google Example:
    - Mail API
    - Drive API
    - YouTube API
    - Etc
    - There’s not just 1 global write scope.
- Add scope for APIs that can charge the user
    - Charging a credit card
    - Billable resources

### Promoting The User For Consent
- Scopes allow users to grant permissions for specific access.
- Scopes represent access to specific parts of a system, so users can be made known exactly what accesses are granted when an app asks the user for permission to access certain abilities.
- For third party apps, this can be done with a consent screen.
- When it comes to the consent screen - make sure you inform the user, but don't overwhelm the user. Write good text.

### Additional Resources
- [Interactive walkthrough of different OAuth flows](https://www.oauth.com/playground/)
- [OAuth Community Site](https://oauth.net/)
- [Okta YouTube Channel](https://www.youtube.com/OktaInc)



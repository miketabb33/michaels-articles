Here are my notes to Aaron Parecki’s Udemy's course, “The Nuts and Bolts of OAuth 2.0.” [Course Link](https://www.udemy.com/course/oauth-2-simplified/). I thoroughly enjoyed the course and took these notes to be a helpful reference for OAuth topics. I highly recommend this course!  (Consider adding a description of whats about to be covered)

## Section 1: Welcome

OAuth and Open ID services typically live in their own server called the Authorization Server, separate from Resource Servers.

#### OAuth

The main responsibility of OAuth is to provide a token used for accessing APIs. Think of this hotel room key scenario. The key gives access to your room, pool, and business center, however, it does not give access to the conference room or offices. For access to these, additional privileges are required.

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

Think of a first party API or something similar. The main quality of a confidential client is that the source code is not viewable.

Confidential credential examples:
- Client secrets
- Private Key JWT
- mTLS

#### Public Clients: No Credentials
Think of an SPA or mobile app. There is no way to ship a secret to the client that the user controls, and it stays a secret. The source code can be viewed.

The authorization server cannot be sure if requests are genuine or being made by someone mimicking the client. This is true with mobile app because the binary files can be inspected.

### User Consent
User consent is a critical part of a 3rd party authorization flow because it ensures that the user is in front of the keyboard explicitly consenting to access.

Internal authorization and user consent typically occurs on a page that lives in the authorization server, which subsequently provides an access token.

- For internal authorization, think of Googles sign in page.
- For 3rd party user consent, think of sign into "some application" with Facebook. (consider numbering?)

### How OAuth Sends Data
#### Back Channel
The secure way of delivering an access token. It uses a client to server HTTPS connection and is similar to hand-delivering a package, you see the recipients.

1. Certificate validation
2. Encryption
3. The response can be trusted because you know where it came from.

The Back Channel does not mean back end. A JavaScript app can use fetch or AJAX and that would be considered Back Channel because the JavaScript is handling the HTTP request directly.

#### Front Channel
Uses the address bar to move data between two systems. It is similar to using a package delivery service to deliver a package.(use a sentence)

1. There’s no direct link between the Application and the Authorization Server.
2. Was the data intercepted?
3. From the sender’s perspective, did the data get to the intended recipient?
4. From the receiver’s prospective, did the data come from a legitimate source?

#### Channel Usage
The end goal is for the Application to get an access token from the Authentication Server and the most secure way is the back channel. However, when we want the user to give consent to provide an access token, we need to use the Front Channel.

#### Implicit Flow
Uses the Front Channel for both the Applications authentication request and the Authentication Servers response. (awkward)

First, the Application makes a request to the Authorization Server via the address bar. To Application includes information in the query string, none of which is particularly sensitive or secret, like:
1. Who the app is (Client Id)
2. What it is trying to do
3. Requested scopes

Then, after user sign in, the Authentication Server redirects back the the Application via the address bar. The redirect URL will provide the access token as a query string, which is where the security vulnerability comes in. This flow is not secure because the Authorization Server is unable to reliably know for sure that they are providing an access token to a trusted Application.


### Application Identity
Client ID: The application's OAuth identifier.

Client Secret: The application's password.

#### Authorization Code Flow
Similar to Implicit flow, expect, the access token is delivered in the back channel. The authorization server does not send the access token in the redirect URL, instead, it sends a first time use Authorization Code with a short expiration time. Next, the application verifies the Authorization Code with the authentication server through the back channel, by using the Client Secret. In exchange, the authorization server sends the access token to the application, however, mobile and SPA apps can’t be deployed with a Client Secret. This is where PKCE comes in. (Proof Key for Code Exchange).

1. The Authentication Server makes a unique secret for each request, to be used when the Application redeems the Authorization Code.
2. PKCE alone does not prevent someone from imitating a client app, all the information is public.
3. The redirect URI is really the only thing that can be used to verify the identity of the application. However, this is not 100% because app there is no global registration for custom URL schemes, duplicates are possible.
4. When using PKCE, it’s important to register approved application URL’s in the authentication server and confirming with the Front Channel’s redirect URL. Don't allow the Authorization Server to redirect if the redirect URL is not registered.

The bottom line is that there really is no perfect solution for mobile and SPA, client secrets are the best way to verify Authorization Codes. However, for mobile and SPA's, PCKE is the recommend approach and protects against an authorization code injection attack.

## Section 3-6: Server Side, Native, and SPA’s
### Server Side
Server Side apps are confidential clients and can use client secrets. Server Side authentication can securely authenticate over https.

### Native
Native Applications are public clients because their binaries can be inspected. They should use the PKCE extension for authentication.

### SPA
JavaScript app’s can securely obtain the access token using the PKCE extension. In addition, storing the access token with browser storage is not secure.
Any scripts run in the browser can access the same storage locations as the code:
1. Code runtime scripts (hot linking)
2. Scripts the users install on their browser.
The recommend approach for JavaScript apps is to deny it access to the access token.
1. Use a dynamic backend server which serves the client code and acts as a proxy for all API calls.
2. Use the front channel to get the authorization code.
3. The browser sends the authorization code to a dynamic backend server, which then interacts with the authorization server to obtains the access token.
4. The browser uses an HTTP only session cookie that maps to the access token in the proxy backend.

## Section 7: Internet of Things
IoT don’t always have keyboards, and when they do, it’s typically not ideal for the user. In addition, there isn't always a browser to use a front channel for user login. Imagine a logging into netflix on a smart TV, a search engine isn't always available.

### Device Flow
1. Separates the device that gets the access token from the device that the user is using to login.
2. Device says to auth server, someone is trying to log in. The auth server only knows the client ID.
3. The auth server sends up a temporary code and the device asks the user to go to a URL and enter the code. The URL is typically short because the user will need to type it in by hand.
4. The URL takes the user to the authentication server, where the user can type in the code and login/authenticate.
5. Once the user is logged in and authenticated, they see a prompt stating “Another device is trying to access your account, would you like to allow it?” They hit yes.(revisit sentence)
6. During this time, the device requesting an access token is pooling every couple of seconds asking, “Is the user finished logging in yet?”
7. Once the user logs in, one of these pooled requests will return with the access token.
8. Complete!

## Section 8: Client Credential Flow
### Client Credential Grant
Client Credential Grand provides a way for the client to get an access token without any user interaction. The approach shines in machine to machine communication. Exchanging the client credentials for an access token simplifies the APIs, so they only need to worry about validating access tokens.

### Steps
1. Register the application at the OAuth server. If asked for an option, go with machine to machine or service account. This will provide the client secret.

2. Send a post request to the auth server with a grant type of client credentials. The exact details of what should be in post request may vary, depending on the OAuth Server.

3. The response should return an access token.

## Section 9: Introduction to OpenID Connect

Open ID uses the ID tokens to share data about the user. ID tokens are always a JWT Token, which have three main parts:
1. Header: Talks about the token, like which signing algorithm was used.
2. Payload: Contains the data, like user ID, email, other profile info, etc.
3. Signature: How the token validated.

### ID Tokens vs Access Tokens

They both can be JWTs and have the same format, however, they are intended to serve 2 very different purposes.

__Access Token:__ It's what the application uses to make requests to APIs. The access token is meant to be opaque and never unpacked, it should be like a hotel key, not knowing anything about the person using it.

__ID Token:__ Intended to be read by the application, like a passport of the user. It should be unpacked. (bold definitions?)(Beware etc.)

### Obtaining an ID Token
Applications typically need both the access token and ID Token.

When using the Authorization Code Flow:
1. Add OpenID scope to the request, and you'll get back an ID Token and and Access Token.
2. Theres no need to verify the token with the signature, because we securely used the back channel.

Implicit Flow but for just the ID Token:
1. Specify the response type to be ID Token to return the ID Token, instead of an access token.
2. Returns an ID Token in the redirect URL instead of the authorization code or access token.
3. Use the JWT signature to validate the ID Token.
4. Front channel is not too secure because if anyone is snooping, they will be able to get data like email address, address, or any other data included in the ID Token.

Only using the OpenID scope will return bare minimum data in the ID Token.
1. Adding new scopes (like name, address, etc.) to the request will add additional information to the JWT.
2. Some servers do not provide additional info in the ID Token, in those cases, use a user info endpoint.

### Hybrid Open ID Connect Flows
Combining ID Token and Authorization Code response types. For example, response_type=code+id_token

While it is okay to combine Authorization code and ID Token, it is bad practice to combine front channel access token with ID Token because of access token leakage. For example, response_type=token+id_token

When using id_token+code, the Authorization Server returns an unverified ID Token in the front channel, but we get the access token later in the flow by exchanging the authorization code in the back channel. Since the ID Token was provided via the front channel, it needs to be verified.

The simplest and most secure solution is not to use Hybrid Flows. Instead, get both ID Token and Access Token with PKCE via the backend channel by adding open_id to the scope.

### Validating and Using an ID Token
Any ID Token received via the front channel absolutely needs to be validated.

1. Determine which key to use.
2. Use a library to validate the token.
3. Validate the claims. Is the issuer, audience, and timestamps correct?
4. Further claims can be validated. Check the amr (how the user logged in) and the auth time (the last time the user actually signed in).

The only time we do not need to validate the ID Token is at the time it comes from a trusted source, like in the back channel during the authorization code flow. Storing an ID Token in the browser, like a cookie, requires that the ID Token be validated before being used.

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
A Long random string of characters that doesn't mean anything, it is a pointer. The Token Data is stored in a database table or something like Redis and the validation happens at the Authentication Server. Reference Tokens shine when the Authentication Server is within the application.

### Self-Encoded Token:
The token string itself contains some kind of data, typically a JWT Token. Only the API should inspect the token, the client should only use the token like the hotel key analogy. The validation happens outside the Authentication Server. Self-Encoded Tokens are more scalable and shine when the Authentication Server is NOT within the application.

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
- Don’t need to be looked up.
- Scalable, because no shared storage is needed.

Cons:
- JWTs are typically not encrypted, it’s public data.
- No way to revoke the token until the token expires.

## Section 12: Access Token Types And Their Tradeoffs
### The Structure of JWT Access Tokens
JWTs are base 64 encoded and are typically not encrypted, but they are signed.
The JWT itself is split into 3 sections, denoted by the ‘.’

#### Header
Contains the meta data about the token like which signing algorithm was used and which key was used to sign the Token.

#### Payload
The data we care about and the claims (ways to validate that the token has not been tampered with):
1. iss: identifies the issuer.
2. exp: expiration timestamp.
3. iat: issued at timestamp.
4. aud: intended audience for the token.
5. sub: ID for whom the token represents, like user ID.
6. client_id: the client ID.
7. jti: unique ID for the particular JWT Token.

#### Signature
Used for validating the JWT.

### Remote Token Introspection
APIs can use Remote Token Introspection to validate tokens. It is a means to verify the token with the Authorization Server. The Authorization Server may expose an HTTP POST endpoint: "Token Introspection Endpoint". The endpoint most often requires some kind of authentication and will respond with the is-valid status of the token.

A negative is that Remote Token Introspection requires network traffic and can cost time and resources.

### Local Token Validation
APIs can use Local Token Validation to validate tokens. This approach is the fast way of validating a token because it does not require any network traffic. When using Local Token Validation, use a library for validating the token locally.

#### Steps:
1. Base 64 decode the JWT.
2. Match the public key in JWT header with the authorization server's public key.
3. Validate the signature (use library).
4. Validate the claims: Issuer, Audience, Expiration, Issued At and custom claims like - Scopes, User Groups, etc.

A negative is tha Local validation only validates the token in the state that it was in when issued. If the token was invalidated in the Authorization Server before the expiration date, the token itself will not reflect that change. As a result, the longer the token lifetime, the higher the risk of making auth decisions on out of date information.

### The Best Of Both Worlds: Using An API Gateway
The first line of defense handles every API request that comes through.
1. API gateway performs ONLY local validation, which thins out the amount of requests coming through the system by removing attackers request and bad tokens.
2. As a result, the services behind the gateway only get valid or potentially valid tokens (they could have been invalidated before the expiration).
3. The services determine whether remote token introspection is required or not.
4. For example, non-sensitive API requests most likely won’t need to use remote token introspection, but something like change password will.

## Section 13: Choosing Token Lifetimes
### Increasing Security With Short Token Lifetimes
Local Token Validation is not able to detect if a token has been invalidated before its expiration. Making a tokens' lifetime shorter is a way to decrease the risk of using an invalid token to authenticate. Shortening the token lifetime also strengthens security against token leaks. For example, short token lifetimes protect from an attack or an accidental log.

### Improving User Experience With Long Token Lifetimes
While short access tokens can improve security, they also has a negative effect on user experience. Access tokens that never expire are typically a bad idea, because of leaks or attacks. Applications can use the refresh token after an access token has expired, causing the user's request to take a little longer. If an app doesn’t have/support a refresh token, it will need to go back to the authorization server to get a new access token. This process can be so fast that it may be unnoticeable, but it still causes a full page redirect. Mobile apps are typically more disruptive because they need to open a browser.

In most cases, the more disruptive it is to get an access token, the longer the token lifetime should be. For web applications, refresh tokens are typically not secure and are not used. Access token lifetimes can be longer. For mobile applications, refresh tokens can securely be used and opening the browser is very disruptive. Shorter token lifetimes with a refresh token can be used and refresh tokens should be long enough for the user experience.

Regardless, the authorization server's session lifetime can be much longer than a specific access/refresh tokens' lifetime.

### Contextually Choosing Token Lifetimes
Access token lifetimes can vary based on different properties like client ID, scope, group, user, to name a few. A SPA can have longer token lifetimes than its mobile app counterpart. An admin or staff user may have a shorter token lifetime than other users. While using scopes, an application may not require a new login while viewing a catalog, but can require a login when the user wants to place an order. As a result, the regular token can be long-lived while, for example, the checkout token can be short-lived. Token lifetimes can very base on use.

## Section 14: Handling Revoked Or Invalidated Access Tokens
### Reasons Why An Access Token May Become Invalidated
- Account Deactivation
- User Revokes Application
- Change Password
- OAuth Config Changes

#### On The App Side
If the request is rejected due to an invalid access token, the app can try and use the refresh token or send the user back to the authorization server.

#### On The API Side:
The client may not know when the token has expired, clients are not supposed to read an access token.

### The Problem With Local Validation
It’s the API's job to make sure it does not respond if an access token has been revoked. If a token becomes invalid due to a reason other than expiration, the JWT will not know about it. Only Remote Token Introspection can the real time status of an access token. Local Validation 100% depends on the data in the JWT to know if a token is valid. (review use of apostrophes ex: API's)

### Token Lifetime Considerations
Tokens with long lifetimes need Remote Token Introspection more often because there is more of a chance that the token has become invalid between issue time and expiration time. Shorter token lifetimes have less of a chance that the token was invalidated between issue time and expiration time, the time is shorter.

### How Applications Can Revoke Access Token
An application can use the revocation endpoint to tell the authorization server that the access token should be invalidated. For example, a token should be invalidated on Logout.

## Section 15: OAuth Scopes
### The Purpose Of OAuth Scopes
Scopes are a way for an application to request access to limited parts of a user account. They allow for segmenting access to data and are simply used to limit what an access token can do. Scopes are not a permission system and should not be used for in-app permissions like what the UI should show. Instead, use a permissions system.

### Defining Scopes For Your API
Scopes will be included in the access token but there are no rules and little guidance.

Keep in mind, scopes are just a string. It's up to the API to know which scopes are required by each of its methods, and enforce the scopes.

Scopes can be defined in format as, but are not limited to:
- upload
- photos:upload
- example.com/photos/uploads

Once scopes are defined, document them so that developers know what the available scopes.

### When to use Scopes:
#### Bare minimum scopes:
- Read
- Write

#### Restricting access to sensitive information
- Personal Information
- Account details
- Etc

#### Segment out unrelated parts of a large API, Google Example:
- Mail API
- Drive API
- YouTube API
- Etc
- There’s not just 1 global write scope.

#### Add scope for APIs that can charge the user
- Charging a credit card
- Billable resources

### Promoting The User For Consent
Scopes allow users to grant permissions for specific access in an easy to understand way. They represent access to specific parts of a system, so users can be made known exactly what accesses are granted when an app asks the user for permission to access certain abilities. For third party apps, this can be done with a consent screen. For the consent screen, make sure you inform but no overwhelm the user.

## The End

Thanks for reading! I hope you found these notes helpful and to learn more, take Aaron Parecki’s Udemy's course, “The Nuts and Bolts of OAuth 2.0.” [Course Link](https://www.udemy.com/course/oauth-2-simplified/). (Consider adding a recap of what was covered)

#### Additional OAuth Resources
- [Interactive walkthrough of different OAuth flows](https://www.oauth.com/playground/)
- [OAuth Community Site](https://oauth.net/)
- [Okta YouTube Channel](https://www.youtube.com/OktaInc)

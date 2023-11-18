Here are my notes to Aaron Parecki’s Udemy's course, “The Nuts and Bolts of OAuth 2.0.” [Course Link](https://www.udemy.com/course/oauth-2-simplified/). I wanted to publish the notes I took during the course. First, as a resource for myself and second, as a resource for others. I thoroughly enjoyed and recommend this course!

These notes are structured in the same format as the course. The notes cover what is Open Authentication (OAuth), Application Programming Interface (API) security concepts, Server Side Applications, Native Applications, Single Page Application (SPA) OAuth, Internet of Things (IoT), client credential flow, open id connect, access token types, JavaScript Web Tokens (JWT), choosing token lifetimes, handling invalid access tokens, and OAuth scopes.

## Section 1: Welcome

#### OAuth
The main responsibility of OAuth is to provide an access token to a Client, to then be used for getting sensitive data from a Resource Server. Think of an access token as a hotel room key. The key gives access to certain areas in the hotel, like your room, pool, and business center. However, the key may not give access to the conference room or offices. For access to other, additional privileges are required. Similarly, an access token may provide a User to get their profile information, but they wont have privileges to access other User's data. Nor would a User have access to administrator related functionality.

#### Open ID
The main responsibility of Open ID is to provide the User's identity. Open ID is an extension of OAuth.

OAuth and Open ID services typically live in their own server called the Authorization Server, separate from Resource Servers.

## Section 2: API Security Concepts

### OAuth Roles (Terms)
1. User (Resource Owner)
2. Device (User Agent)
3. Application (Client)
4. Authorization Server
5. API (Resource Server)

### Application Types
#### Confidential Clients: Have Credentials

A confidential client is one where it's source code is not publicly viewable. Hidden source code allows confidential clients to securely authenticate with confidential credentials. Confidential credential examples are client secrets, private key JWTs, and Mutual Transport Layer Security (mTLS).

A first party API is an API developed for internal use and is considered a confidential client.

#### Public Clients: No Credentials
A public client is one where it's source code is publicly viewable. There is no way to ship a client secret to a public client and the secret stay secret. The public clients source code can be viewed by the public.

The Authorization Server cannot be sure if requests are genuine or being made by someone mimicking the Client. This is true with a mobile Application (because the binary files can be inspected) and with a SPA (because the source cod can be inspected).

### User Consent
User consent is a critical part of a third party authorization flow because consent ensures that the User is in front of the keyboard explicitly granting access to a resource. The software that provides internal authorization and the user consent page typically live in the Authorization Server.

Consider the Google ecosystem which has many Resource Services like email, docs, chat, an countless others. A sign into Google automatically grants access to all google Resource Servers. Whereas, signing into a third party website using a Google Account requires explicit user consent. Google's user consent page will prompt the User to grant access for the third party to use some of Google's Resource Servers.

### How OAuth Sends Data
#### Back Channel
Using the back channel is considered the secure way of delivering an access token. It uses HTTP for a direct client to server connection. This channel is analogous to hand-delivering a package, you see the recipients.

Pros to using the back channel:
1. Certificate validation
2. Encryption is available
3. The response can be trusted because the request has information about the requestor.

The back channel does not equate to back end. A JavaScript Application can use fetch or AJAX and that would be considered back channel because the JavaScript is handling the HTTP request directly.

#### Front Channel
The front channel uses the address bar to move data between two systems. It is analogous to using a package delivery service to deliver a package, opposed to hand delivering the package.

The front channel is not secure because:
1. There’s no direct link between the Application and the Authorization Server.
2. Was the data intercepted?
3. From the sender’s perspective, did the data get to the intended recipient?
4. From the receiver’s prospective, did the data come from a legitimate source?

#### Channel Usage
The end goal is for the Application to acquire an access token from the Authorization Server. The most secure way to acquire an access token is through the back channel. However, in order for the User to give consent to an Authorization Server to provide an access token for a Client, we need to use the front channel.

#### Implicit Flow
The implicit flow uses the front channel for all stages of the authentication process.

First, the Application makes a request to the Authorization Server via the address bar. The request includes information in the query string, none of which is particularly sensitive or secret, like:
1. Who the application is
2. What the application is trying to do
3. Requested scopes

After sign in, the Authorization Server redirects the User back the Application via the address bar. The redirect URL will provide the access token as a query string, which is where the security vulnerability comes in. This flow is not secure because the Authorization Server is unable to reliably know for sure that the access token is being given to a trusted Application.

### Application Identity
__Client ID:__ The Application's OAuth identifier. For example: "google_email"

__Client Secret:__ The Application's password. For example: "ZYDPLLBWSK3MVQJSIYHB1OR2JXCY0X2C5UJ2QAR2MAAIT5Q"

#### Authorization Code Flow
The authorization code flow is similar to the implicit flow, expect, the access token is delivered via the back channel.

After sign in, instead of sending the access token, the Authorization Server sends a single time use authorization code with a short expiration time in the redirect URL to the Application. Next, the Application includes the authorization code and client secret in an HTTP request to the Authorization Server through the back channel. The Authorization Server trades the authorization code for an access token and responds to the Application. However, mobile and SPA Applications are public clients and cannot be deployed with a client secret. To solve for this issue, the Proof Key for Code Exchange (PKCE) extension was introduced.

The PKCE Extension simply specifies that the Authorization Server use a unique secret for each request, to be provided with the authorization code when the Application redeems the authorization code. In other words, the unique key is used instead of a client secret. PKCE alone does not prevent someone from imitating a Client because all the information is public. The redirect URI registry is the only data point that can be used to verify the identity of the Application. However, this is not 100% because there is not a global registry for custom URL schemes and duplicates are technically possible. When using PKCE, it’s important to register approved Application URL’s in the Authorization Server and confirm with the front channel’s redirect URL. Don't allow the Authorization Server to redirect back to the Application if the redirect URL is not registered.

The bottom line is there is no perfect solution for mobile and SPA. While client secrets are the best way to verify authorization codes, The PCKE Extension is the recommended approach for mobile and SPA Applications as the extension protects against an authorization code injection attack.

## Section 3-6: Server Side, Native, and SPA’s
### Server Side
Server Side Applications are confidential clients and can use client secrets. Server Side authentication can securely authenticate over HTTP.

### Native
Native Applications are public clients because their binaries can be inspected. They should use the PKCE extension for authentication.

### SPA
JavaScript Applications can securely obtain the access token using the PKCE extension. However, storing the access token with browser storage is not secure. Any scripts run in the browser can access the same storage locations as the code:
1. Code runtime scripts or "hot linking".
2. Scripts the Users install on their browser.

The recommend approach for JavaScript Applications is to keep the access token out of the browser altogether.
1. Use a dynamic backend server which serves the client code and acts as a proxy for all API calls.
2. Use the front channel to get the authorization code.
3. The browser sends the authorization code to a dynamic backend server, which then interacts with the Authorization Server to obtains the access token.
4. The browser uses an HTTP only session cookie that maps to the access token in the proxy backend.

## Section 7: Internet of Things
The internet of things (IoT) are devices that connected to the internet that are not desktops, laptops, and mobile devices. They usually don’t have keyboards. When IoT devices do have keyboards, the experience generally is not ideal for the User. In addition, there isn't always a browser to use a front channel for User login. Imagine logging into netflix on a smart TV, a search engine isn't always available.

### Device Flow
The device flow separates the Device that needs the access token from the Device where login occurs. The Device that needs the access token makes a request with its client id to the Authorization Server, letting the Authorization Server know that someone is trying to login. The Authorization Server responds with a temporary code and the Device that needs the access token prompts the User to use another Device to go to a specific URL and enter the temporary code. The URL is typically short because the User will need to type it in by hand. The URL takes the User to the Authorization Server, where the User can type in the code and login.

Once the User is authenticated on the Device used for login, a prompt appears stating “Another Device is trying to access your account, would you like to allow it?” During this time, the Device requesting an access token is pooling every couple of seconds asking, “Is the User finished logging in yet?”. Once the User logs in, one of these pooled requests will return with the access token. Complete!

## Section 8: Client Credential Flow
### Client Credential Grant
Client Credential Grant provides a way for the Client to get an access token without any User interaction. The approach shines in machine to machine communication. Exchanging the client credentials for an access token simplifies the APIs, so they only need to worry about validating access tokens.

### Steps
1. Register the Application at the OAuth server. If asked for an option, go with machine to machine or service account. This process will provide the client secret.

2. Send a post request to the Authorization Server with a grant type of client credentials. The exact details of what should be in post request may vary, depending on the OAuth Server.

3. The response will return an access token.

## Section 9: Introduction to OpenID Connect

Open ID uses an ID token to share data about the User.

### ID Tokens vs Access Tokens

ID tokens and access tokens can both be JWTs and have the same format. However, they are intended to serve 2 very different purposes.

__Access Token:__ Intended for Application to use in requests to APIs. The access token is meant to be opaque and never unpacked by the Application.

__ID Token:__ Intended to be read by the Application, like a passport of the User. The ID token should be unpacked by the Client.

### Obtaining an ID Token
Applications typically need both the access token and ID token.

When using the authorization code flow:
1. Add the OpenID scope to the request and the Authorization Server will return the ID token and access token.
2. Theres no need to verify the token with the signature, because we securely used the back channel.

Implicit flow, but for only the ID token:
1. Specify the response type to be ID token.
2. Returns an ID token in the redirect URL instead of the authorization code or access token.
3. Use the JWT signature to validate the ID token.
4. The front channel is not secure because someone may be snooping. A snooper will be able to get data like email address, address, or any other data included in the ID token. Be careful to exclude sensitive data.

Only using the OpenID scope will return bare minimum data in the ID token.
1. Add new scopes like name, address, or email to the request to add additional information to the ID token.
2. Some servers do not provide additional info in the ID token. In those cases, utilize a user info endpoint.

### Hybrid Open ID Connect Flows
A hybrid open ID connect flow is used to combine the ID token and authorization code response types, effectively getting both data points back at the same time. For example, response_type=code+id_token.

While it is okay to combine the authorization code and ID token, it is bad practice to combine front channel access token with ID token. For example, response_type=token+id_token. Combining access token and Id token can cause access token leakage.

When using code+id_token, the Authorization Server returns the authorization code and an unverified ID token in the front channel. Since the ID token was provided via the front channel, it needs to be verified.

The simplest and most secure solution is not to use Hybrid Flows. Instead, get both ID token and Access Token with PKCE via the backend channel by adding open_id to the scope.

### Validating and Using an ID Token
Any ID token received via the front channel absolutely needs to be validated.

1. Determine which key to use.
2. Use a library to validate the token.
3. Validate the claims. Is the issuer, audience, and timestamps correct?
4. Further claims can be validated. Check the amr (how the User logged in) and the auth time (the last time the User actually signed in).

The only time we do not need to validate the ID token is at the time it comes from a trusted source, like in the back channel during the authorization code flow. Storing an ID token in the browser requires that the ID token be validated before being used.

## Section 11: Access Token Types And Their Tradeoffs
### Reference Tokens Vs Self-Encoded Tokens
There is many kinds of token data:
- User
- Application
- Expiration
- Time Created
- Scopes
- Authorization Server
- Last Login Time

### Reference Token:
A long random string of characters that doesn't mean anything, the string is a pointer. The token data is stored in a database table or something like Redis. Validation happens in the Authorization Server. Reference Tokens shine when the Authorization Server is within the Application.

### Self-Encoded Token:
The token string itself contains data. Only the API should inspect the token. The validation happens outside the Authorization Server. Self-Encoded Tokens are more scalable and shine when the Authorization Server is outside the Application.

### Pros and Cons of Reference Tokens
Pros:
- Simple.
- Easy to add and delete entries.
- Reference tokens can point to data that isn’t visible to the Application or the API, like sensitive data. These tokens are good for hiding data from employees and third party Applications.

Cons:
- The data needs to be stored. If there isn’t an associated entry in the database, the token is invalid.
- Storage scalability.
- APIs are forced to look up data when validating the access token.

### Pros and Cons of Self-Encoded Tokens
Pros:
- The token has all the data required to validate it.
- The token doesn't need to be stored or looked up.
- Self encoded tokens are scalable because no shared storage is needed.

Cons:
- JWTs are typically not encrypted and can expose public data.
- There is no way to revoke the token until the token expires.

## Section 12: JWT Tokens
### The Structure of JWT Access Tokens
JWTs are base 64 encoded tokens and are typically not encrypted, but they are signed.
The JWT itself is split into 3 sections, denoted by the ‘.’

#### Header
Contains the meta data about the token like which signing algorithm was used and which key was used to sign the Token.

#### Payload
The data we care about and the claims (ways to validate that the token has not been tampered with):
1. iss: identifies the issuer.
2. exp: expiration timestamp.
3. iat: issued at timestamp.
4. aud: intended audience for the token.
5. sub: ID for whom the token represents, like User ID.
6. client_id: the client ID.
7. jti: unique ID for the particular JWT Token.

#### Signature
Used for validating the JWT.

### Remote Token Introspection
Resource Servers can use remote token introspection to validate tokens. This introspection is a means to verify the token with the Authorization Server. The Authorization Server may expose an HTTP POST endpoint: "Token Introspection Endpoint". The endpoint most often requires some kind of authentication and will respond with the is-valid status of the token.

A negative to remote token introspection is that it requires network traffic, costing time and resources.

### Local Token Validation
Resource Servers can use local token validation to validate tokens. This approach is the fast way of validating a token because it does not require any network traffic. When using local token validation, use a library for validating the token locally.

#### Steps:
1. Base 64 decode the JWT.
2. Match the public key in JWT header with the Authorization Server's public key.
3. Validate the signature (use a library).
4. Validate the claims: Issuer, Audience, Expiration, Issued At and custom claims like - Scopes, User Groups, and other custom claims.

A negative to local token validation is that it only validates the token in the state that it was in when issued. If the token was invalidated in the Authorization Server before the expiration date, the token itself will not reflect that change. As a result, the longer the token lifetime, the higher the risk of making auth decisions with out of date information.

### The Best Of Both Worlds: Using An API Gateway
The API gateway is the first line of defense and handles every API request that comes through. The API gateway only performs local validation, which thins out the amount of requests coming through the system by removing attackers request and bad tokens. As a result, the Resource Server behind the gateway only receives valid or potentially valid tokens. The Resource Server determines whether remote token introspection is required or not. For example, non-sensitive API requests most likely won’t need to use remote token introspection, but something like change password will require introspection.

## Section 13: Choosing Token Lifetimes
### Increasing Security With Short Token Lifetimes
Local token validation is not able to detect if a token has been invalidated before its expiration. Making a tokens' lifetime shorter is a practice to decrease the risk of using an invalid token to authenticate. Shortening the token lifetime also strengthens security against token leaks. For example, short token lifetimes protect from an attack or an accidental log.

### Improving User Experience With Long Token Lifetimes
While short access tokens can improve security, they also have a negative effect on user experience. Access tokens that never expire are typically a bad idea, because of leaks or attacks. Applications can use the refresh token after an access token has expired, causing the User's request to take a little longer. If an Application doesn’t support a refresh token, it will need to go back to the Authorization Server to get a new access token. This process can be so fast that it may be unnoticeable, but it still causes a full page redirect. Mobile applications are typically more disruptive because they need to open a browser.

In most cases, the more disruptive it is to get an access token the longer the token lifetime should be. For web applications, refresh tokens are typically not secure and are not used. Access token lifetimes can be longer. For mobile applications, refresh tokens can be securely used and opening the browser is very disruptive. Shorter token lifetimes with a refresh token can be used and refresh tokens should be long enough for the User experience.

Regardless, the Authorization Server's session lifetime can be much longer than a specific access or refresh tokens' lifetime.

### Contextually Choosing Token Lifetimes
Access token lifetimes can vary based on different properties like client ID, scope, group, and user, to name a few. A SPA can have longer token lifetimes than its mobile counterpart. An admin or staff User may have a shorter token lifetime than other Users. While using scopes, an Application may not require a new login while viewing a catalog, but can require a login when the User wants to place an order. As a result, the regular token can be long-lived while, for example, the checkout token can be short-lived. Token lifetimes can very base on use.

## Section 14: Handling Revoked Or Invalidated Access Tokens
### Reasons Why An Access Token May Become Invalidated
- Account Deactivation
- User Revokes Application
- Change Password
- OAuth Config Changes

If the request is rejected due to an invalid access token, the Application can try and use the refresh token or send the User back to the Authorization Server.

### The Problem With Local Validation
It’s the Resource Server's job to make sure it does not respond when the access token has been revoked. If a token becomes invalid due to a reason other than expiration, the JWT will not know about it. Only remote token introspection can reveal the real time status of an access token. Local validation can only use the sata in the JWT to know if a token is valid.

### Token Lifetime Considerations
Tokens with long lifetimes need remote token introspection more often because there is more of a chance that the token has become invalid between issue time and expiration time. Shorter token lifetimes have less of a chance that the token was invalidated between issue time and expiration time because the gap of time is smaller.

### How Applications Can Revoke Access Token
An Application can use the revocation endpoint to tell the Authorization Server that the access token should be invalidated. For example, a token should be invalidated on logout.

## Section 15: OAuth Scopes
### The Purpose Of OAuth Scopes
Scopes are a way for an Application to request access to limited parts of a User account. Scopes allow for segmenting access to data and are simply used to limit what an access token can do. They are not a permission system and should not be used for in-app permissions like what the UI should show. Use a permission system for Application level permissions, not scopes.

### Defining Scopes For Your API
Scopes will be included in the access token but there are no rules and little guidance on how they should be used. Keep in mind, scopes are just a string. It's up to the Resource Server to know which scopes are required for each of its methods.

Scopes are simply strings and can be formatted in many different ways. Here are some examples of how we can format scopes:
- upload
- photos:upload
- example.com/photos/uploads

Once scopes are defined, document them so that developers know what the available scopes.

### When to use Scopes:
#### Bare minimum scopes:
- Read
- Write

#### Restricting access to sensitive information
- Personal information
- Account details

#### Segment out unrelated parts of a large API, Google Example:
- Mail API
- Drive API
- YouTube API
- There’s not just 1 global write scope.

#### Add scopes for Resource Servers that can charge the User
- Charging a credit card
- Billable resources

### Prompting The User For Consent
Scopes allow Users to grant permissions for specific access in an easy to understand way. Scopes represent access to specific parts of a system, so Users can be made known exactly what accesses are granted when an Application prompts user consent. When designing a consent screen, be thoughtful of how you present the information, make sure to inform but no overwhelm the User.

## The End

Thanks for reading! I hope you found these notes helpful and to learn more, take Aaron Parecki’s Udemy's course, “The Nuts and Bolts of OAuth 2.0.” [Course Link](https://www.udemy.com/course/oauth-2-simplified/).

#### Additional OAuth Resources
- [Interactive walkthrough of different OAuth flows](https://www.oauth.com/playground/)
- [OAuth Community Site](https://oauth.net/)
- [Okta YouTube Channel](https://www.youtube.com/OktaInc)

# Secure APIs using JWT - Self Contained JWT

Microgateway can accept JWTs issued by a **trusted** key manager as a valid token to invoke the APIs.  JWT tokens are self-validated by the API Microgateway without validating it against the authorization server(key manager) that issued the JWT.

When a JWT is used as an access token, API Microgateway validates the following attributes/claims of the JWT.

-   **Signature** - after validating the signature, API Microgateway checks whether the JWT is issued by a trusted key manager, and JWT has not tampered in the middle. This signature validation is done by using the public certificate of the key manager which issued the JWT. Importing the public certificate into the API Microgateway trust store and configuring the certificate alias in the JWT validation config section is explained in the [importng certificates to the microgateway truststore](/how-tos/security/importing-certificates-to-the-api-microgateway-truststore/)
-   **Issuer(iss)** - The issuer claim is a mandatory claim when JWT is used as a security token. Microgateway validates the **iss** claim present in the JWT against the issuer provided in the **jwtTokenConfig** section of the configuration
-   **Audience (aud)** - The audience claim is also a mandatory claim for a JWT that is used as a security token. Microgateway validates the **aud** claim present in the JWT against the audience provided in the **jwtTokenConfig** section of the configuration.
-   **Expiry time(exp)** - "exp" claim is also a mandatory claim. Microgateway validates the validity period of the token using the **exp** claim

**Jwt token validation config**

``` yml
[[jwtTokenConfig]]
    issuer = "https://localhost:9443/oauth2/token"
    audience = "http://org.wso2.apimgt/gateway"
    certificateAlias = "wso2apim310"
    # Validate subscribed APIs
    validateSubscription = false
```

 The certificateAlias, issuer, audience of the above configuration will be used to validate the signature, iss claim and aud claim of the JWT respectively. This configuration should be added to the micro-gw.conf file located in &lt;MICRO-GW-RUNTIME\_HOME&gt;/conf/.

### Subscription Validation

  The [subscription](https://apim.docs.wso2.com/en/3.2.0/learn/consume-api/manage-subscription/subscribe-to-an-api/) validation is configurable for JWT tokens. When WSO2 API Manager is used as the key manager it sends the subscribed APIs as a list in the JWT under the **subscribedAPIs** claim. In order to mandate the subscriptions, subscription validation can be enabled. Microgateway will validate the list under the subscribedAPIs claim and check if the user is currently invoking one of the APIs in the list. If not it will send an error message with error code 900908.

   If an external key manager is used which will not know about the subscription details then, subscription validation can be turned off for that particular JWT issuers.

### Configure Multiple JWT issuers

 There can be use cases in certain organizations, where multiple JWT issuers or key managers are used. In that case, Microgateway can be configured to work with JWTs issued by all of them. Multiple JWT issuer feature allows to configure multiple JWT token configurations in the micro-gw.conf file. Configurations allows to specify array of **jwtTokenConfig** sections. In the case of multiple JWT issuers are provided Microgateway will sequentially check a JWT with all the available issuers. Valid JWT tokens will be cached and then expiry time only will be validated in the subsequent calls with the same token.

 **Multiple JWT Issuers**

``` yml
# JWT token authorization configurations. You can provide multiple JWT issuers
# Issuer 1
[[jwtTokenConfig]]
    issuer = "https://localhost:9443/oauth2/token"
    audience = "http://org.wso2.apimgt/gateway"
    certificateAlias = "wso2apim310"
    # Validate subscribed APIs
    validateSubscription = false
# Issuer 2
[[jwtTokenConfig]]
    issuer = "https://host:port/issuer"
    audience = "http://org.wso2.apimgt/gateway"
    certificateAlias = "alias"
    # Validate subscribed APIs
    validateSubscription = false
```

### Passing a token to the backend.

 When you are using the JWT authentication, you can provide a token to be passed to the backend using a JWT claim "backendJWT". i.e. If your JWT token has **"backendJwt" claim** , then Its value is passed to the backend. To configure which header it is used for passing the backend JWT, the following should be added to the Microgateway runtime configuration in &lt;MICRO-GW-RUNTIME\_HOME&gt;/conf/micro-gw.conf.

``` yml
[jwtConfig]
    # JWT header when forwarding the request to the backend
    header = "X-JWT-Assertion"
```



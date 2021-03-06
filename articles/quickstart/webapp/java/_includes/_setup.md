## Configure Callback URLs

A Callback URL is a URL that Auth0 invokes after the authentication process. Auth0 routes your application back to this URL and attaches some details to it including a token. Callback URLs can be manipulated on the fly which could be harmful. For security reasons, you will need to add your application's callback URL in the **Allowed Callback URLs** box in your app's [Client Settings](${manage_url}/#/applications/${account.clientId}/settings). This will allow Auth0 to recognize the URLs as valid. The same principle applies to **Logout URLs**. They must be whitelisted in the Auth0 Dashboard.

If you are following this guide directly, set the following for your Callback and Logout URLs:

- Allowed Callback URLs: `http://localhost:8080/callback`
- Allowed Logout URLs: `http://localhost:8080/logout`


## Setup Dependencies

To integrate your Java application with Auth0, add the following dependencies:

- **javax.servlet-api**: is the library that allows you to create Java Servlets. You then need to add a Server dependency like Tomcat or Gretty, which one is up to you. Check our sample code for more information.
- **auth0-java-mvc-commons**: is the Java library that allows you to use Auth0 with Java for server-side MVC web apps. It generates the Authorize URL that you need to call in order to authenticate and validates the result received on the way back to finally obtain the [Auth0 Tokens](/tokens) that identify the user. You can always check the latest version in the [library's GitHub](https://github.com/auth0/auth0-java-mvc-common).

If you are using Gradle, add them to your `build.gradle`:

```java
compile 'javax.servlet:javax.servlet-api:3.1.0'
compile 'com.auth0:mvc-auth-commons:1.+'
```

If you are using Maven, add them to your `pom.xml`:

```xml
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>mvc-auth-commons</artifactId>
  <version>1.+</version>
</dependency>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.1.0</version>
</dependency>
```


## Configure your Java App

Your Java App needs some information in order to authenticate against your Auth0 account. The samples read this information from the deployment descriptor file `src/main/webapp/WEB-INF/web.xml`, but you could store them anywhere else. The required information is:

```xml
<context-param>
    <param-name>com.auth0.domain</param-name>
    <param-value>${account.namespace}</param-value>
</context-param>

<context-param>
    <param-name>com.auth0.clientId</param-name>
    <param-value>${account.clientId}</param-value>
</context-param>

<context-param>
    <param-name>com.auth0.clientSecret</param-name>
    <param-value>${account.clientSecret}</param-value>
</context-param>
```

The library we're using has this default behavior:
- Request the scope `openid`, needed to call the `/userinfo` endpoint later to verify the User's identity.
- Request the `code` Response Type and later perform a Code Exchange to obtain the tokens.
- Use the `HS256` Algorithm along with the Client Secret to verify the tokens.

But it also allows us to customize it's behavior:
* To use the `RS256` Algorithm along with the Public Key obtained dynamically from the Auth0 hosted JWKs file, pass a `JwkProvider` instance to the `AuthenticationController` builder.
* To use a different Response Type, set the desired value in the `AuthenticationController` builder. Any combination of `code token id_token` is allowed.
* To request a different `scope`, set the desired value in the `AuthorizeUrl` received after calling `AuthenticationController#buildAuthorizeUrl()`.
* To specify the `audience`, set the desired value in the `AuthorizeUrl` received after calling `AuthenticationController#buildAuthorizeUrl()`.


::: panel Check populated attributes
If you download the seed using our **Download Sample** button then the `domain`, `clientId` and `clientSecret` attributes will be populated for you, unless you are not logged in or you do not have at least one registered client. In any case, you should verify that the values are correct if you have multiple clients in your account and you might want to use another than the one we set the information for.
:::

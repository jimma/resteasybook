Authentication {#Authentication}
==============

Since Resteasy runs within a servlet container you can use most (all?)
mechanism available in your servlet container for authentication. Basic
and Digest authentication are probably the easiest to set up and fit
nicely into REST's stateless principle. Form security can be used, but
requires passing the session's cookie value with each request. We have
done some preliminary work on OAuth and also plan to work on OpenID and
SAML integration in the future.

OAuth core 1.0a
===============

RESTEasy has preliminary support for [OAuth core
1.0a](http://oauth.net/core/1.0a). This includes support for
authenticating with OAuth (as described by the [spec section
6](http://oauth.net/core/1.0a#rfc.section.6)) and OAuth authentication
for protected resources (as described by the [spec section
7](http://oauth.net/core/1.0a#rfc.section.7)).

> **Important**
>
> This API is deprecated and will be removed in subsequent versions of
> Resteasy unless there is an outcry from the community. We're focusing
> on OAuth 2.0 protocols. Please see our [OAuth 2.0 Work](#oauth2).

Authenticating with OAuth 1.0a
------------------------------

OAuth authentication is the process in which Users grant access to their
Protected Resources without sharing their credentials with the Consumer.

OAuth Authentication is done in three steps:

1.  The Consumer obtains an unauthorized Request Token. This part is
    handled by RESTEasy.

2.  The User authorizes the Request Token. This part is *not handled by
    RESTEasy* because it requires a user interface where the User logs
    in and authorizes or denies the Request Token. This cannot be
    implemented automatically as it needs to be integrated with your
    User login process and user interface.

3.  The Consumer exchanges the Request Token for an Access Token. This
    part is handled by RESTEasy.

In order for RESTEasy to provide the two URL endpoints where the Client
will request unauthorized Request Tokens and exchange authorized Request
Tokens for Access Tokens, you need to enable the OAuthServlet in your
web.xml:

``` {.xml}
                
<!-- The OAuth Servlet handles token exchange -->
<servlet>
  <servlet-name>OAuth</servlet-name>
  <servlet-class>org.jboss.RESTEasy.auth.oauth.OAuthServlet</servlet-class>
</servlet>

<!-- This will be the base for the token exchange endpoint URL -->
<servlet-mapping>
  <servlet-name>OAuth</servlet-name>
  <url-pattern>/oauth/*</url-pattern>
</servlet-mapping>
                
            
```

The following configuration options are available using
`<context-param> elements`:

  Option Name                     Default         Description
  ------------------------------- --------------- ------------------------------------------------------------------------------------------
  oauth.provider.provider-class   \*Required\*    Defines the fully-qualified class name of your OAuthProvider implementation
  oauth.provider.tokens.request   /requestToken   This defines the endpoint URL for requesting unauthorized Request Tokens
  oauth.provider.tokens.access    /accessToken    This defines the endpoint URL for exchanging authorized Request Tokens for Access Tokens

  : OAuth 1.0a Servlet options

Accessing protected resources
-----------------------------

After successfully receiving the Access Token and Token Secret, the
Consumer is able to access the Protected Resources on behalf of the
User.

RESTEasy supports OAuth authentication for protected resources using a
servlet filter which should be mapped in your web.xml for all protected
resources:

``` {.xml}
                
<!-- The OAuth Filter handles authentication for protected resources -->
<filter>
  <filter-name>OAuth Filter</filter-name>
  <filter-class>org.jboss.RESTEasy.auth.oauth.OAuthFilter</filter-class>
</filter>
    
<!-- This defines the URLs which should require OAuth authentication for your protected resources -->
<filter-mapping>
  <filter-name>OAuth Filter</filter-name>
  <url-pattern>/rest/*</url-pattern>
</filter-mapping>
                
            
```

The following configuration options are available using
`<context-param> elements`:

  Option Name                     Default        Description
  ------------------------------- -------------- -----------------------------------------------------------------------------
  oauth.provider.provider-class   \*Required\*   Defines the fully-qualified class name of your OAuthProvider implementation

  : OAuth Filter options

Once authenticated, the OAuth Servlet Filter will set your request's
Principal and Roles, which can then be accessed using the JAX-RS
SecurityContext. You can also protect your resources using Roles as
described in the section "Securing JAX-RS and RESTeasy".

Implementing an OAuthProvider
-----------------------------

In order for RESTEasy to implement OAuth it needs you to provide an
instance of `OAuthProvider` which will provide access to the list of
Consumer, Request and Access Tokens. Because one size doesn’t fit all we
cannot know if you wish to store your Tokens and Consumer credentials in
a configuration file, in memory, or on persistent storage.

All you need to do is implement the `OAuthProvider` interface:


    public interface OAuthProvider {
        String getRealm();

        OAuthConsumer getConsumer(String consumerKey)throws OAuthException;
        OAuthToken getRequestToken(String consumerKey, String requestToken) throws OAuthException;
        OAuthToken getAccessToken(String consumerKey, String accessToken) throws OAuthException;
        
        OAuthToken makeRequestToken(String consumerKey, String callback) throws OAuthException;
        OAuthToken makeAccessToken(String consumerKey, String requestToken, String verifier) throws OAuthException;

        String authoriseRequestToken(String consumerKey, String requestToken) throws OAuthException;

        void checkTimestamp(OAuthToken token, long timestamp) throws OAuthException;
    }

                

If a Consumer Key, or Token doesn’t exist, or if the timestamp is not
valid, simply throw an `OAuthException`.

The rest of the interfaces used in `OAuthProvider` are:


    public interface OAuthConsumer {
        String getKey();
        String getSecret();
    }

    public interface OAuthToken {
        OAuthConsumer getConsumer();
        String getToken();
        String getSecret();
        Principal getPrincipal();
        Set<String> getRoles();
    }

                

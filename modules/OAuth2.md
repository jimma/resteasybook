OAuth 2.0 and Resteasy Skeleton Key {#oauth2}
===================================

The overall goal of Resteasy Skeleton Key is to provide a unified way
for both Browser and JAX-RS clients to be secured in an integrated and
seemless fashion. We want to support a network of applications and
services so that if one server needs to execute or forward requests to
another, there is a secure and scalable way to do this without hitting a
central authentication server each and every request.

The [OAuth 2.0](http://tools.ietf.org/html/rfc6749) Authorization
Framework enables a third-party to obtain access to an HTTP resource on
behalf of a resource owner without the third-party having to know the
credentials of the resource owner. It does this by issuing access tokens
via a browser redirect protocol, or by a direct grant. The access tokens
can then be transmitted by the [OAuth2 Bearer
Token](http://tools.ietf.org/html/draft-ietf-oauth-v2-bearer-23)
protocol to access the protected resource.

Resteasy Skeleton Key is an OAuth 2.0 implementation that allows you to
use existing JBoss AS7 security infrastructure to secure your web
applications and restful services. You can turn an existing web app into
an OAuth 2.0 Access Token Provider or you can turn a JBoss AS7 Security
Domain into a central authentication and authorization server that a
whole host of applications and services can use. Here are the features
in a nutshell:

-   Turn an existing servlet-form-auth-based web application into an
    OAuth 2.0 provider.

-   Provide Distributed Single-Sign-On (SSO) from a central
    authentication server. Log in once, and you can securely access any
    browser-based app configured to work in the domain.

-   Provide Distributed Logout. Following one link from any application
    can log you out of all your distributed applications configured to
    use SSO.

-   Web apps can interact securely with any remote restful service by
    forwarding access tokens through the standard Authorization header.

-   Access tokens are digitally signed by the oauth2 framework and can
    be used to access any service configured to work in the domain. The
    tokens contain both identity and role mapping information. Because
    they are digitally signed, there's no need to overload the central
    authentication server with each request to verify identity and to
    determine permissions.

> **Important**
>
> The Resteasy distribution comes with an OAuth2 Skeleton key example.
> This is a great way to see OAuth2 in action and how it is configured.
> You may also want to use this as a template for your applications.

System Requirements
===================

-   JBoss AS 7.1.x or higher

-   HTTPS is required. See the JBoss 7 documentation on how to enable
    SSL for web applications

-   Resteasy 3.0 or higher must be installed within your
    AS7 distribution. View how to
    upgrade AS7
    to the latest version of Resteasy.
-   A username/password based JBoss security domain

-   Browser-based apps must be configured to use servlet FORM
    authentication and web.xml security constraints

Generate the Security Domain Key Pair
=====================================

Access tokens are digitally signed and verified by an RSA keypair. You
must generate this keypair using the JDK's keytool command or by
something like openssl.

    $ keytool -genkey -alias mydomain -keyalg rsa -keystore realm.jks

This will ask you a series of questions that will be used to create the
X509 public certificate. Basic PKI stuff that you hopefully are already
familiar with. Move this keystore file into a directory that you can
reference from a configuration file. I suggest the
standalone/configuration directory of your JBoss AS7 distribution.

Setting up the Auth Server
==========================

The next thing you're gonna want to do is set up a web application to be
your OAuth2 provider. This can be an existing web app or you can create
a new WAR to be your central authentication server. An existing web app
must be configured to use servlet FORM authentication. Enabling OAuth2
within this app will not change how normal users interact with it.

Setting up your Security Domain
-------------------------------

You can use any set of JBoss AS7 login modules you want to store your
username, passwords and role mappings. Each security domain will be
comprised of regular users, oauth clients, and admins. Oauth clients
represent either a web application that wants to use the auth-server to
do SSO, or they are traditional oauth clients that want access permision
to act on behalf of another user (the traditional OAuth use case). Every
oauth client must have a username, password, and a specific role mapping
that gives them various permissions to participate in OAuth 2 protocols.
There is a role that grants an oauth client permission to login as a
specific user (default is `login`. This is the SSO case. There is a role
that grants a client permission to request permission to act on behalf
of a user (default is `oauth`). Additional role mappings assigned to the
oauth client define what additional permissions they are allowed to
have. These additional permissions are the role mappings of the
application and are the intersection of the permissions given to the
user the client is acting on behalf of. This is better explained by an
example role mapping file:

    wburke=user,admin
    loginclient=login
    oauthclient1=oauth,*
    oauthclient2=oauth,user

In the above role mapping file with have a simple user `wburke`. He has
application role permissions of `user` and `admin`. One oauth client
user is `loginclient`. It has been given a role mapping of `login`. This
client is allowed to login as the user and is given all roles of the
user. The `oauthclient1` user is not allowed to login as the user, but
is allowed to obtain an OAuth grant to act on behalf of the user. The
`*` role means that `oauthclient1` is granted the same roles as the user
it is acting on behalf of. If `oauthclient1` acts on behalf of `wburke`
then it will have both `user` and `admin` permissions. The
`oauthclient2` is also allowed to use the oauth grant protocol, but it
will only ever be granted `user` permissions.

You are not confined to login, oauth, and \* as role mapping names. You
can configure them to be whatever you want.

*Why have different login and oauth role mappings?* `login` clients are
allowed to bypass entering username and password if the user has already
logged in once and has an existing authenticated session with the
server. `oauth` clients are *always* required to enter username and
password. You probably don't want to grant permission automatically to
an oauth client. A user will want to look at who is requesting
permission. This role distinction gives you this capability.

Auth Server Config File
-----------------------

You must create a configuration file that holds all the configuration
for OAuth2. This is json formatted If you name it `resteasy-oauth.json`
and put it within the `WEB-INF/` directory of your war, that's all you
have to do. Otherwise, you must specify the full path to this
configuration file within a context-param within your web.xml file. The
name of this param is `skeleton.key.config.file`. You can reference
System properties within the value of this context-param by enclosing
them within `${VARIABLE}`. Here's an example configuration:

    {
       "realm" : "mydomain",
       "admin-role" : "admin",
       "login-role" : "login",
       "oauth-client-role" : "oauth",
       "wildcard-role" : "*",
       "realm-keystore" : "${jboss.server.config.dir}/realm.jks",
       "realm-key-alias" : "mydomain",
       "realm-keystore-password" : "password",
       "realm-private-key-password" : "password",
       "access-code-lifetime" : "300",
       "token-lifetime" : "3600",
       "truststore" : "${jboss.server.config.dir}/client-truststore.ts",
       "truststore-password" : "password",
       "resources" : [
          "https://example.com/customer-portal",
          "https://somewhere.com/product-portal"
       ]
    }

Let's go over what each of these config variables represent:

realm

:   Name of the realm representing the users of your distributed
    applications and services

admin-role

:   Admin role mapping used for admins. You must have this defined if
    you want to do distributed logout.

login-role

:   Role mapping for login clients.

oauth-client-role

:   Role mapping for regular oauth clients.

wildcard-role

:   Role mapping for assigning all roles to an oauth client wishing to
    act on behalf of a user.

realm-keystore

:   Absolute path pointing to the keystore that contains the
    realm's keypair. This keypair is used to digitally sign
    access tokens. You may use `${VARIABLE}` to reference
    System properties. The example is referencing the JBoss config dir.

realm-key-alias

:   Key alias for the key pair stored in your realm-keystore file.

realm-keystore-password

:   Password to access the keystore.

realm-private-key-password

:   Password to access the private realm key within the keystore

access-code-lifetime

:   The access code is obtained via a browser redirect after you log
    into the central server. This access code is then transmitted in a
    separate request to the auth server to obtain an access token. This
    variable is the lifetime of this access code. In how many seconds
    will it expire. You want to keep this value short. The default is
    300 seconds.

token-lifetime

:   This is how long in seconds the access token is viable after it was
    first created. The default is one hour. Depending on your security
    requirements you may want to extend or shorten this default.

truststore

:   Used for outgoing client HTTPS communications. This contains one or
    more trusted host certificates or certificate authorities. This is
    OPTIONAL if you are not using distributed logout.

truststore-password

:   Password for the truststore keystore.

resources

:   Root URLs of applications using this auth-server for SSO. This is
    OPTIONAL and only needed if you want to allow distributed logout.

Set up web.xml
--------------

Set up your security constraints however you like. You must though use
FORM authentication.

Set up jboss-web.xml
--------------------

In jboss-web.xml in your WEB-INF directory, point to your security
domain as a normal secured web app does, and also use a specific valve.

    <jboss-web>
        <security-domain>java:/jaas/commerce</security-domain>
        <valve>
            <class-name>org.jboss.resteasy.skeleton.key.as7.OAuthAuthenticationServerValve</class-name>
        </valve>
    </jboss-web>

Set up jboss-deployment-structure.xml
-------------------------------------

You must import the skeleton key modules so that the classes are visible
to this application. Include this file within WEB-INF

    <jboss-deployment-structure>
        <deployment>
            <dependencies>
                <module name="org.jboss.resteasy.resteasy-jaxrs" services="import"/>
                <module name="org.jboss.resteasy.resteasy-jackson-provider" services="import"/>
                <module name="org.jboss.resteasy.skeleton-key"/>
            </dependencies>
        </deployment>
    </jboss-deployment-structure>

Tweak your login page
---------------------

The action url used by your login form is dependent on the oauth
protocol. Skeleton key defines a request attribute called
`OAUTH_FORM_ACTION` which is the URL you should use for the form's
action. Here's an example login.jsp page that uses this attribute:

    <form action="<%= request.getAttribute("OAUTH_FORM_ACTION")%>" method=post>
        <p><strong>Please Enter Your User Name: </strong>
        <input type="text" name="j_username" size="25">
        <p><p><strong>Please Enter Your Password: </strong>
        <input type="password" size="15" name="j_password">
        <p><p>
        <input type="submit" value="Submit">
        <input type="reset" value="Reset">
    </form>

Setting Up An App for SSO
=========================

This section specifies how you can use the central auth-server for SSO.
Following these directions will use the auth-server for browser log in.
The server will also be able to do bearer token authentication as well.

SSO config file
---------------

The best way to create the config file for your application is to ask
the central authentication server you configured in the last section.
So, boot up the auth server and go to
https://*auth-server-context-root*/j\_oauth\_realm\_info.html. For
example: `https://localhost:8443/auth-server/j_oauth_realm_info.html`.
This will show template configurations depending on which valve you are
using. You want the `OAuthManagedResourceValve` config. It will look
something like this.

    {
      "realm" : "mydomain",
      "realm-public-key" : "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCO8XXyi7oAq5ecsYy+tJrl54N2TtKAkxuWEDmzvSPU+mUA2/3qHcxucZakG74Z49410tn5IIu2CXXlk9CuKcpXvKh+cPBzmC1Nmbd+4MelRVVZnvogyPICs8h3sNTAMNdfI6hDc5/MfVQQ9m5OZrKbNR3dY50mTi/ExnJ5IWPqxQIDAQAB",
      "admin-role" : "admin",
      "auth-url" : "https://localhost:8443/auth-server/login.jsp",
      "code-url" : "https://localhost:8443/auth-server/j_oauth_resolve_access_code",
      "truststore" : "REQUIRED",
      "truststore-password" : "REQUIRED",
      "client-id" : "REQUIRED",
      "client-credentials" : {
        "password" : "REQUIRED"
      }
    }

Let's go over what each of these config variables represent:

realm

:   Name of the realm representing the users of your distributed
    applications and services

realm-public-key

:   PEM format of public key.

admin-role

:   Admin role mapping used for admins. You must have this defined if
    you want to do distributed logout.

auth-url

:   URL of the auth server's login page.

code-url

:   URL to turn an access code into an access token. (Part of the
    OAuth2 protocol)

truststore

:   Used for outgoing client HTTPS communications. This contains one or
    more trusted host certificates or certificate authorities. This is
    REQUIRED as you must talk HTTPS to the auth server to turn an access
    code into an access token. You can create this truststore by
    extracting the public certificate of the auth server's SSL keystore.
    The google knows if you want to know how to do this.

truststore-password

:   Password for the truststore keystore.

client-id

:   Username of the login client. This server will send client-id and
    password when turning an access code into an access token.
    Internally, the server will do an HTTPS invocation to the
    auth-server passing this information using Basic AUTH.

client-credentials

:   Must specify the password of the oauth login client.

Set up web.xml
--------------

Set up your security constraints however you like. You must though use
FORM authentication.

Set up jboss-web.xml
--------------------

In jboss-web.xml in your WEB-INF directory you need to use a specific
valve.

    <jboss-web>
        <valve>
            <class-name>org.jboss.resteasy.skeleton.key.as7.OAuthManagedResourceValve</class-name>
        </valve>
    </jboss-web>

Set up jboss-deployment-structure.xml
-------------------------------------

You must import the skeleton key modules so that the classes are visible
to this application. Include this file within WEB-INF

    <jboss-deployment-structure>
        <deployment>
            <dependencies>
                <module name="org.jboss.resteasy.resteasy-jaxrs" services="import"/>
                <module name="org.jboss.resteasy.resteasy-jackson-provider" services="import"/>
                <module name="org.jboss.resteasy.skeleton-key"/>
            </dependencies>
        </deployment>
    </jboss-deployment-structure>

Bearer Token only Setup
=======================

If you have a web app that you want only to allow Bearer token
authentication, i.e. a set of JAX-RS services then follow these
directions.

Bearer token auth config file
-----------------------------

The best way to create the config file for your application is to ask
the central authentication server you configured in the last section.
So, boot up the auth server and go to
https://*auth-server-context-root*/j\_oauth\_realm\_info.html. For
example: `https://localhost:8443/auth-server/j_oauth_realm_info.html`.
This will show template configurations depending on which valve you are
using. You want the `BearerTokenAuthenticatorValve` config. It will look
something like this.

    {
      "realm" : "mydomain",
      "realm-public-key" : "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCO8XXyi7oAq5ecsYy+tJrl54N2TtKAkxuWEDmzvSPU+mUA2/3qHcxucZakG74Z49410tn5IIu2CXXlk9CuKcpXvKh+cPBzmC1Nmbd+4MelRVVZnvogyPICs8h3sNTAMNdfI6hDc5/MfVQQ9m5OZrKbNR3dY50mTi/ExnJ5IWPqxQIDAQAB",
    }

All that is needed is the realm name, and the public key of the realm.
Let's go over what each of these config variables represent:

realm

:   Name of the realm representing the users of your distributed
    applications and services

realm-public-key

:   PEM format of the realm's public key. Used to verify tokens.

Set up web.xml
--------------

Set up your security constraints however you like. You must though use
FORM authentication.

Set up jboss-web.xml
--------------------

In jboss-web.xml in your WEB-INF directory you need to use a specific
valve.

    <jboss-web>
        <valve>
            <class-name>org.jboss.resteasy.skeleton.key.as7.BearerTokenAuthenticatorValve</class-name>
        </valve>
    </jboss-web>

Set up jboss-deployment-structure.xml
-------------------------------------

You must import the skeleton key modules so that the classes are visible
to this application. Include this file within WEB-INF

    <jboss-deployment-structure>
        <deployment>
            <dependencies>
                <module name="org.jboss.resteasy.resteasy-jaxrs" services="import"/>
                <module name="org.jboss.resteasy.resteasy-jackson-provider" services="import"/>
                <module name="org.jboss.resteasy.skeleton-key"/>
            </dependencies>
        </deployment>
    </jboss-deployment-structure>

Obtaining an access token programmatically.
===========================================

You can request an access token from the auth-server by doing a simple
HTTPS invocation. You must use BASIC authentication to identify your
user, and you will get back a signed access token for that user. Here's
an example using a JAX-RS 2.0 client:

        ResteasyClient client = new ResteasyClientBuilder()
                                    .truststore(truststore)
                                    .build();

        Form form = new Form().param("grant_type", "client_credentials");
        ResteasyWebTarget target = client.target("https://localhost:8443/auth-server/j_oauth_token_grant");
        target.configuration().register(new BasicAuthentication("bburke@redhat.com", "password"));
        AccessTokenResponse res = target.request()
                               .post(Entity.form(form), AccessTokenResponse.class);

The above makes a simple POST to the context root of the auth server
with `j_oauth_token_grant` at the end of the target URL. This resource
is responsible for creating access tokens.

        try
        {
           Response response = client.target("https://localhost:8443/database/products").request()
                                   .header(HttpHeaders.AUTHORIZATION, "Bearer " + res.getToken()).get();
           String xml = response.readEntity(String.class);
        }
        finally
        {
           client.close();
        }

The access token is a simple string. To invoke on a service protected by
bearer token auth, just set the `Authorization` header of your HTTPS
request with a value of `Bearer` and then the access token string.

Access remote services securely in a secure web session
=======================================================

If you have an application secured by one of the methods described in
this chapter, you can obtain the access token of the current web session
so that you can use it to invoke on other remote services securely using
bearer token authentication. Each HttpServletRequest in a secure web
session has an attribute called
`org.jboss.resteasy.skeleton.key.SkeletonKeySession` which points to an
instance of a class with the same name. This class contains the access
token and also points to the truststore you configured. You can then
extract this info and make secure remote invocations. Here's an example
of that.

       public List<String> getCustomers(HttpServletRequest request)
       {
          SkeletonKeySession session = (SkeletonKeySession)request.getAttribute(SkeletonKeySession.class.getName());
          ResteasyClient client = new ResteasyClientBuilder()
                     .truststore(session.getMetadata().getTruststore())
                     .build();
          try
          {
             Response response = client.target("https://localhost:8443/database/customers").request()
                     .header(HttpHeaders.AUTHORIZATION, "Bearer " + session.getToken()).get();
             return response.readEntity(new GenericType<List<String>>(){});
          }
          finally
          {
             client.close();
          }
       }
            

If you are within a JAX-RS environment you can inject a
`SkeletonKeySession` using the `@Context` annotation.

Check Out the OAuth2 Example!
=============================

> **Important**
>
> The Resteasy distribution comes with an example project that shows all
> of these different features in action! Check it out!

Auth Server Action URLs
=======================

For reference, here is the set of relative URL actions that the auth
server will publish.

*login page*

:   The is the url of your login page. OAuth clients will redirect
    to it. This is application specific.

j\_oauth\_resolve\_access\_code

:   Used by oauth clients to turn an access code into an access token.

j\_oauth\_logout

:   Do a GET request to this URL and it will perform a
    distributed logout.

j\_oauth\_token\_grant

:   Do a POST with BASIC Auth to obtain an access token for a
    specific user.

j\_oauth\_realm\_info.html

:   Displays an HTML page with template configurations for using
    this realm.



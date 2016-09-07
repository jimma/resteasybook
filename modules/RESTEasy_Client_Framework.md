Resteasy Client API {#RESTEasy_Client_Framework}
===================

JAX-RS 2.0 Client API {#client_api}
=====================

JAX-RS 2.0 introduces a new client API so that you can make http
requests to your remote RESTful web services. It is a 'fluent' request
building API with really 3 main classes: Client, WebTarget, and
Response. The Client interface is a builder of WebTarget instances.
WebTarget represents a distinct URL or URL template from which you can
build more sub-resource WebTargets or invoke requests on.

There are really two ways to create a Client. Standard way, or you can
use the ResteasyClientBuilder class. The advantage of the latter is that
it gives you a few more helper methods to configure your client.

                Client client = ClientBuilder.newClient();
                ... or...
                Client client = ClientBuilder.newBuilder().build();
                WebTarget target = client.target("http://foo.com/resource");
                Response response = target.request().get();
                String value = response.readEntity(String.class);
                response.close();  // You should close connections!

                ResteasyClient client = new ResteasyClientBuilder().build();
                ResteasyWebTarget target = client.target("http://foo.com/resource");
            

Resteasy will automatically load a set of default providers. (Basically
all classes listed in all META-INF/services/javax.ws.rs.ext.Providers
files). Additionally, you can manually register other providers,
filters, and interceptors through the Configuration object provided by
the method call Client.configuration(). Configuration also lets you set
various configuration properties that may be needed.

Each WebTarget has its own Configuration instance which inherits the
components and properties registered with its parent. This allows you to
set specific configuration options per target resource. For example,
username and password.

Resteasy Proxy Framework {#proxy_framework}
========================

The Resteasy Proxy Framework is the mirror opposite of the JAX-RS
server-side specification. Instead of using JAX-RS annotations to map an
incoming request to your RESTFul Web Service method, the client
framework builds an HTTP request that it uses to invoke on a remote
RESTful Web Service. This remote service does not have to be a JAX-RS
service and can be any web resource that accepts HTTP requests.

Resteasy has a client proxy framework that allows you to use JAX-RS
annotations to invoke on a remote HTTP resource. The way it works is
that you write a Java interface and use JAX-RS annotations on methods
and the interface. For example:

    public interface SimpleClient
    {
       @GET
       @Path("basic")
       @Produces("text/plain")
       String getBasic();

       @PUT
       @Path("basic")
       @Consumes("text/plain")
       void putBasic(String body);

       @GET
       @Path("queryParam")
       @Produces("text/plain")
       String getQueryParam(@QueryParam("param")String param);

       @GET
       @Path("matrixParam")
       @Produces("text/plain")
       String getMatrixParam(@MatrixParam("param")String param);

       @GET
       @Path("uriParam/{param}")
       @Produces("text/plain")
       int getUriParam(@PathParam("param")int param);
    }

Resteasy has a simple API based on Apache HttpClient. You generate a
proxy then you can invoke methods on the proxy. The invoked method gets
translated to an HTTP request based on how you annotated the method and
posted to the server. Here's how you would set this up:

                Client client = ClientFactory.newClient();
                WebTarget target = client.target("http://example.com/base/uri");
                ResteasyWebTarget rtarget = (ResteasyWebTarget)target;

                SimpleClient simple = rtarget.proxy(SimpleClient.class);
                client.putBasic("hello world");
            

Alternatively you can use the Resteasy client extension interfaces
directly:

                ResteasyClient client = new ResteasyClientBuilder().build();
                ResteasyWebTarget target = client.target("http://example.com/base/uri");

                SimpleClient simple = target.proxy(SimpleClient.class);
                client.putBasic("hello world");
            

@CookieParam works the mirror opposite of its server-side counterpart
and creates a cookie header to send to the server. You do not need to
use @CookieParam if you allocate your own javax.ws.rs.core.Cookie object
and pass it as a parameter to a client proxy method. The client
framework understands that you are passing a cookie to the server so no
extra metadata is needed.

The framework also supports the JAX-RS locator pattern, but on the
client side. So, if you have a method annotated only with @Path, that
proxy method will return a new proxy of the interface returned by that
method.

Abstract Responses {#Custom_client-side_responses}
==================

Sometimes you are interested not only in the response body of a client
request, but also either the response code and/or response headers. The
Client-Proxy framework has two ways to get at this information

You may return a javax.ws.rs.core.Response.Status enumeration from your
method calls:

    @Path("/")
    public interface MyProxy {
       @POST
       Response.Status updateSite(MyPojo pojo);
    }
                

Internally, after invoking on the server, the client proxy internals
will convert the HTTP response code into a Response.Status enum.

If you are interested in everything, you can get it with the
javax.ws.rs.core.Response class:

    @Path("/")
    public interface LibraryService {

       @GET
       @Produces("application/xml")
       Response getAllBooks();
    }

Sharing an interface between client and server {#Sharing_interfaces}
==============================================

It is generally possible to share an interface between the client and
server. In this scenario, you just have your JAX-RS services implement
an annotated interface and then reuse that same interface to create
client proxies to invoke on the client-side.

Apache HTTP Client 4.x and other backends {#transport_layer}
=========================================

Network communication between the client and server is handled in
Resteasy, by default, by HttpClient (4.x) from the Apache HttpComponents
project. In general, the interface between the Resteasy Client Framework
and the network is found in an implementation of
`org.jboss.resteasy.client.jaxrs.ClientHttpEngine`, and
`org.jboss.resteasy.client.jaxrs.engines.ApacheHttpClient4Engine`, which
uses HttpClient (4.x), is the default implementation. Resteasy also
ships with the following client engines, all found in the
`org.jboss.resteasy.client.jaxrs.engines` package:

-   URLConnectionClientExecutor: uses
    java.net.HttpURLConnection
    ;
-   InMemoryClientExecutor: dispatches requests to a server in the
    same JVM.

and a client executor may be passed to a specific `ClientRequest`:

    ResteasyClient client = new ResteasyClientBuilder().httpEngine(engine).build();
         

Resteasy and HttpClient make reasonable default decisions so that it is
possible to use the client framework without ever referencing
HttpClient, but for some applications it may be necessary to drill down
into the HttpClient details. `ApacheHttpClient4Engine` can be supplied
with an instance of `org.apache.http.client.HttpClient` and an instance
of `org.apache.http.protocol.HttpContext`, which can carry additional
configuration details into the HttpClient layer. For example,
authentication may be configured as follows:

    // Configure HttpClient to authenticate preemptively
    // by prepopulating the authentication data cache.
     
    // 1. Create AuthCache instance
    AuthCache authCache = new BasicAuthCache();
     
    // 2. Generate BASIC scheme object and add it to the local auth cache
    AuthScheme basicAuth = new BasicScheme();
    authCache.put(new HttpHost("sippycups.bluemonkeydiamond.com"), basicAuth);
     
    // 3. Add AuthCache to the execution context
    BasicHttpContext localContext = new BasicHttpContext();
    localContext.setAttribute(ClientContext.AUTH_CACHE, authCache);
     
    // 4. Create client executor and proxy
    DefaultHttpClient httpClient = new DefaultHttpClient();
    ApacheHttpClient4Engine engine = new ApacheHttpClient4Engine(httpClient, localContext);
    ResteasyClient client = new ResteasyClientBuilder().httpEngine(engine).build();
         

One default decision made by HttpClient and adopted by Resteasy is the
use of `org.apache.http.impl.conn.SingleClientConnManager`, which
manages a single socket at any given time and which supports the use
case in which one or more invocations are made serially from a single
thread. For multithreaded applications, `SingleClientConnManager` may be
replaced by
`org.apache.http.impl.conn.tsccm.ThreadSafeClientConnManager`:

    ClientConnectionManager cm = new ThreadSafeClientConnManager();
    HttpClient httpClient = new DefaultHttpClient(cm);
    ApacheHttpClient4Engine engine = new ApacheHttpClient4Engine(httpClient);
         

For more information about HttpClient (4.x), see the documentation at
<http://hc.apache.org/httpcomponents-client-ga/tutorial/html/>.

**Note.** It is important to understand the difference between
"releasing" a connection and "closing" a connection. **Releasing** a
connection makes it available for reuse. **Closing** a connection frees
its resources and makes it unusable.

`SingleClientConnManager` manages a single socket, which it allocates to
at most a single invocation at any given time. Before that socket can be
reused, it has to be released from its current use, which can occur in
one of two ways. If an execution of a request or a call on a proxy
returns a class other than `Response`, then Resteasy will take care of
releasing the connection. For example, in the fragments

    WebTarget target = client.target("http://localhost:8081/customer/123");
    String answer = target.request().get(String.class);
         

or

    ResteasyWebTarget target = client.target("http://localhost:8081/customer/123");
    RegistryStats stats = target.proxy(RegistryStats.class);
    RegistryData data = stats.get();
         

Resteasy will release the connection under the covers. The only
counterexample is the case in which the response is an instance of
`InputStream`, which must be closed explicitly.

On the other hand, if the result of an invocation is an instance of
`Response`, then Response.close() method must be used to released the
connection.

    WebTarget target = client.target("http://localhost:8081/customer/123");
    Response response = target.request().get();
    System.out.println(response.getStatus());
    response.close();
         

You should probably execute this in a try/finally block. Again,
releasing a connection only makes it available for another use. **It
does not normally close the socket.**

On the other hand, ApacheHttpClient4Engine.finalize() will close any
open sockets, but only if it created the `HttpClient` it has been using.
If an `HttpClient` has been passed into the `ApacheHttpClient4Executor`,
then the user is responsible for closing the connections:

    HttpClient httpClient = new DefaultHttpClient();
    ApacheHttpClient4Engine executor = new ApacheHttpClient4Engine(httpClient);
    ...
    httpClient.getConnectionManager().shutdown();
         

Note that if `ApacheHttpClient4Engine` has created its own instance of
`HttpClient`, it is not necessary to wait for finalize() to close open
sockets. The `ClientHttpEngine` interface has a close() method for this
purpose.

Finally, if your javax.ws.rs.client.Client class has created the engine
automatically for you, you should call Client.close() and this will
clean up any socket connections.

Resteasy Caching Features {#Cache_NoCache_CacheControl}
=========================

Resteasy provides numerous annotations and facilities to support HTTP
caching semantics. Annotations to make setting Cache-Control headers
easier and both server-side and client-side in-memory caches are
available.

@Cache and @NoCache Annotations {#Cache_Annotation}
===============================

Resteasy provides an extension to JAX-RS that allows you to
automatically set Cache-Control headers on a successful GET request. It
can only be used on @GET annotated methods. A successful @GET request is
any request that returns 200 OK response.

    package org.jboss.resteasy.annotations.cache;

    public @interface Cache
    {
       int maxAge() default -1;
       int sMaxAge() default -1;
       boolean noStore() default false;
       boolean noTransform() default false;
       boolean mustRevalidate() default false;
       boolean proxyRevalidate() default false;
       boolean isPrivate() default false;
    }

    public @interface NoCache
    {
       String[] fields() default {};
    }

       

While @Cache builds a complex Cache-Control header, @NoCache is a
simplified notation to say that you don't want anything cached; i.e.
Cache-Control: nocache.

These annotations can be put on the resource class or interface and
specifies a default cache value for each @GET resource method. Or they
can be put individually on each @GET resource method.

Client "Browser" Cache {#client_cache}
======================

Resteasy has the ability to set up a client-side, browser-like, cache.
You can use it with the Client Proxy Framework, or with ordinary
requests. This cache looks for Cache-Control headers sent back with a
server response. If the Cache-Control headers specify that the client is
allowed to cache the response, Resteasy caches it within local memory.
The cache obeys max-age requirements and will also automatically do HTTP
1.1 cache revalidation if either or both the Last-Modified and/or ETag
headers are sent back with the original response. See the HTTP 1.1
specification for details on how Cache-Control or cache revalidation
works.

It is very simple to enable caching. Here's an example of using the
client cache with the Client Proxy Framework

    @Path("/orders")
    public interface OrderServiceClient {

       @Path("{id}")
       @GET
       @Produces("application/xml")
       public Order getOrder(@PathParam("id") String id);
    }

To create a proxy for this interface and enable caching for that proxy
requires only a few simple steps in which the `BrowserCacheFeature` is
registered:

    ResteasyWebTarget target = (ResteasyWebTarget) ClientBuilder.newClient().target("http://localhost:8081");
    BrowserCacheFeature cacheFeature = new BrowserCacheFeature();
    OrderServiceClient orderService = target.register(cacheFeature).proxy(OrderServiceClient.class);

`BrowserCacheFeature` will create a Resteasy `LightweightBrowserCache`
by default. It is also possible to configure the cache, or install a
completely different cache implementation:

    ResteasyWebTarget target = (ResteasyWebTarget) ClientBuilder.newClient().target("http://localhost:8081");
    LightweightBrowserCache cache = new LightweightBrowserCache();
    cache.setMaxBytes(20);
    BrowserCacheFeature cacheFeature = new BrowserCacheFeature();
    cacheFeature.setCache(cache);
    OrderServiceClient orderService = target.register(cacheFeature).proxy(OrderServiceClient.class); 

If you are using the standard JAX-RS client framework to make
invocations rather than the proxy framework, it is just as easy:

    ResteasyWebTarget target = (ResteasyWebTarget) ClientBuilder.newClient().target("http://localhost:8081/orders/{id}");
    BrowserCacheFeature cacheFeature = new BrowserCacheFeature();
    target.register(cacheFeature);
    String rtn = target.resolveTemplate("id", "1").request().get(String.class);

The LightweightBrowserCache, by default, has a maximum 2 megabytes of
caching space. You can change this programmatically by callings its
setMaxBytes() method. If the cache gets full, the cache completely wipes
itself of all cached data. This may seem a bit draconian, but the cache
was written to avoid unnecessary synchronizations in a concurrent
environment where the cache is shared between multiple threads. If you
desire a more complex caching solution or if you want to plug in a
thirdparty cache please contact our resteasy-developers list and discuss
it with the community.

Local Server-Side Response Cache {#server_cache}
================================

Resteasy has a server-side cache that can sit in front of your JAX-RS
services. It automatically caches marshalled responses from HTTP GET
JAX-RS invocations if, and only if your JAX-RS resource method sets a
Cache-Control header. When a GET comes in, the Resteasy Server Cache
checks to see if the URI is stored in the cache. If it does, it returns
the already marshalled response without invoking your JAX-RS method.
Each cache entry has a max age to whatever is specified in the
Cache-Control header of the initial request. The cache also will
automatically generate an ETag using an MD5 hash on the response body.
This allows the client to do HTTP 1.1 cache revalidation with the
IF-NONE-MATCH header. The cache is also smart enough to perform
revalidation if there is no initial cache hit, but the jax-rs method
still returns a body that has the same ETag.

The cache is also automatically invalidated for a particular URI that
has PUT, POST, or DELETE invoked on it. You can also obtain a reference
to the cache by injecting a org.jboss.resteasy.plugins.cache.ServerCache
via the @Context annotation


        @Context
        ServerCache cache;

        @GET
        public String get(@Context ServerCache cache) {...}

To set up the server-side cache you must register an instance of
org.jboss.resteasy.plugins.cache.server.ServerCacheFeature via your
Application getSingletons() or getClasses() methods. The underlying
cache is Infinispan. By default, Resteasy will create an Infinispan
cache for you. Alternatively, you can create and pass in an instance of
your cache to the ServerCacheFeature constructor. You can also configure
Infinispan by specifying various context-param variables in your
web.xml. First, if you are using Maven you must depend on the
resteasy-cache-core artifact:


    <dependency>
       <groupId>org.jboss.resteasy</groupId>
       <artifactId>resteasy-cache-core</artifactId>
       <version>3.1.0-SNAPSHOT</version>
    </dependency>

The next thing you should probably do is set up the Infinispan
configuration in your web.xml.


    <web-app>
        <context-param>
            <param-name>server.request.cache.infinispan.config.file</param-name>
            <param-value>infinispan.xml</param-value>
        </context-param>

        <context-param>
            <param-name>server.request.cache.infinispan.cache.name</param-name>
            <param-value>MyCache</param-value>
        </context-param>

    </web-app>

server.request.cache.infinispan.config.file can either be a classpath or
a file path. server.request.cache.infinispan.cache.name is the name of
the cache you want to reference that is declared in the config file.

HTTP preconditions {#Http_Precondition}
==================

JAX-RS provides an API for evaluating HTTP preconditions based on
`"If-Match"`, `"If-None-Match"`, `"If-Modified-Since"` and
`"If-Unmodified-Since"` headers.

                Response.ResponseBuilder rb = request.evaluatePreconditions(lastModified, etag);
            

By default Resteasy will return status code 304 (Not modified) or 412
(Precondition failed) if any of conditions fails. However it is not
compliant with RFC 7232 which states that headers `"If-Match"`,
`"If-None-Match"` MUST have higher precedence. You can enable RFC 7232
compatible mode by setting `resteasy.rfc7232preconditions` context
parameter to `true`
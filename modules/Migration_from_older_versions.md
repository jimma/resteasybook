Migration from older versions {#Migration_from_older_versions}
=============================

Migrating from 3.0.7 to 3.0.9 {#307_to_309}
=============================

-   You may need to upgrade your JDK to the latest 1.7.x or 1.8 releases
    if you are using JAXB. There's some entity expansion security
    vulnerabilities that we had to patch and it seems the fix doesn't
    work on earlier versions of JDK 1.7.
-   ContainerRequestContext.setRequestUri() method behavior has been
    changed to match the behavior of the JAXRS RI. The relative URI must
    be equal to are an extension of the base URI or it will not work.
    i.e.
            context.setRequestUri(URI.create("https://foo.com/base"),
                                  URI.create("https://foo.com/base/path"));  // legal
            context.setRequestUri(URI.create("https://foo.com/base"),
                                  URI.create("/path"));  // "path" is ignored
            // if base uri is "http://foo.com/base"
            context.setRequestUri(URI.create("http://foo.com/base/path")); // legal
            context.setRequestUri(URI.create("/path")); // ignored

Migrating from 3.0.6 to 3.0.7 {#306_to_307}
=============================

-   Scannotation has been removed. If you are not running within an
    application server you must use the ResteasyServletInitializer. See
    docs for more detail.

Migrating from 3.0 to 3.0.4 {#30_to_304}
===========================

-   Servlet 3.0 deployments within standalone Tomcat or Jetty can now
    use the Resteasy
    ServletContainerInitializer
    . This allows tighter integration with Resteasy much like you have
    within JBoss/Wildfly. Check out the Installation/Configuration
    section of this document for more information.

Migrating from 3.0-beta-6 and 3.0-rc-1 {#30beta6_to_30rc1}
======================================

-   Form parameters are now read via a provider where earlier they were
    read from HttpServletRequest.getParameterMap(). This may break
    deployments that depend on that behavior, i.e. if you have a servlet
    filter that calls that very method. For those situations I added the
    switch resteasy.use.container.form.params
-   The JAX-RS TCK has become very strict with a ton more tests. I can't
    remember them all, but there are a number of edge cases which
    earlier Resteasy releases misinterpreted.
-   Any Failure exceptions in the SPI now have a corresponding JAX-RS
    2.0 exception, so they have been deprecated Resteasy no longer uses
    these old SPI exceptions internally. It now uses the JAX-RS
    2.0 ones.
-   A number of SPIs have changed. Shouldn't be an issue for those of
    you who use Restasy as-is. Specifically InjectorFactory and Registry
    have changed.

Migrating from 3.0-beta-5 and 3.0-beta-6 {#30beat5_to_30beta6}
========================================

-   The JAX-RS 2.0 TCK has become very very strict in terms of the
    matching algorithm. Unfortunately, the matching algorithm is quite
    poor so there are a number of resource schemes that will no longer
    match. For example, resource classes are scanned for a best match,
    other are ignored in the match. Resource locators are not visited
    unless they are a best match over resource methods. There is one
    config switch I added so that Resteasy will ignore the Spec defined
    class expression filtering step and instead match base on the full
    expressions of each JAX-RS method. resteasy.wider.request.matching.
    Set that to true and you will at least be able to avoid that.
-   The JAX-RS TCK has become very strict with a ton more tests. I can't
    remember them all, but there are a number of edge cases which
    earlier Resteasy releases misinterpreted.
-   Any Failure exceptions in the SPI now have a corresponding JAX-RS
    2.0 exception, so they have been deprecated Resteasy no longer uses
    these old SPI exceptions internally. It now uses the JAX-RS
    2.0 ones.
-   A number of SPIs have changed. Shouldn't be an issue for those of
    you who use Restasy as-is. Specifically InjectorFactory and Registry
    have changed.

Migrating from 3.0-beta-4 and 3.0-beta-5 {#30beta4_to_30beta5}
========================================

-   JSONP support is no longer on by default. A few users have
    complained that it is a security hole for their applications.
-   A number of SPIs have changed. Shouldn't be an issue for those of
    you who use Restasy as-is. Specifically InjectorFactory and Registry
    have changed.

Migrating from 3.0-beta-2 and 3.0-beta-4 {#30beta2_to_30beta4}
======================================== 

-   The JAX-RS 2.0 class ClientFactory no longer exists. It has been
    replaced with ClientBuilder. You can still call newClient(), but
    there is now an additional builder interface. Likewise, there is
    no ResteasyClientFactory. We also have an extension to ClientBuilder
    called ResteasyClientBuilder. So docs for more details.
-   Filter execution and exception handling now matches the JAX-RS
    2.0 spec. Exceptions thrown from filters/interceptors can now be
    mapped if possible. Responses returned from ExceptionMappers are
    now filtered.

Migrating from 3.0-beta-1 and 3.0-beta-2 {#30beta1_to_30beta2}
========================================

-   The constructors for ResteasyClient class are no longer public. You
    need to use the new ResteasyClientBuilder class. One thing to note
    is that when a ResteasyClient is created by a builder, it no longer
    uses ResteasyProviderFactory.getInstance(), but instead instantiates
    a new one. This will probably not effect most uses.

Migrating from 2.x to 3.0-beta-1 {#2x_to_30beta1}
================================

-   Resteasy manual client API, interceptors, StringConverters,
    StringParamterConverters, and Async HTTP APIs have all been
    deprecated and will be removed possibly in a later release. There is
    now a JAX-RS 2.0 equivalent for each of these things.
-   resteasy-crypto: SignedInput and SignedOutput must have a
    multipart/signed content type set either through the request or
    response object, or by annotation @Consumes/@Produces
-   Server-side cache setup has been changed. Please see documentation
    for more details.
-   The security filters for @RolesAllowed, etc. now return 403,
    Forbidden instead of 401.
-   Most add() methods have been removed or made protected
    in ResteasyProviderFactory. Use registerProvider()
    and registerProviderInstance() methods.
-   The new JAX-RS 2.0 client-side filters will not be bound and run
    when you are using Resteasy's old client api.
-   On server-side, all old Resteasy interceptors can run in parallel
    with the new JAX-RS 2.0 filter and interceptor interfaces.
-   Some SPIs have changed. This should not effect applications unless
    you are doing something you aren't supposed to do.
-   The async tomcat and async jboss web modules have been removed. If
    you are not running under Servlet 3.0, async HTTP server-side, will
    be faked and run synchronously in same request thread.

Migrating from 2.3.2 to 2.3.3 {#232_to_233}
=============================

-   MultipartInput has a new close() method. If you have a read body
    that is MultipartInput or one of its subinterfaces, then you must
    call this method to clean up any temporary files created. Otherwise,
    these possible temporary files are deleted on GC or JDK shutdown.
    Other multipart providers clean up automatically.

Migrating from 2.3.0 to 2.3.1 {#230_to_231}
=============================

-   sjsxp has been removed as a dependency for the Resteasy JAXB
    provider

Migrating from 2.2.x to 2.3 {#22x_to_23}
===========================

-   The Apache Abdera integration has been removed as a project. If you
    want the integration back, please ping our dev lists or open a JIRA.
-   Apache Http Client 4.x is now the default underlying client
    HTTP mechanism. If there are problems, you can change the default
    mechanism by calling ClientRequest.setDefaultExecutorClass.
-   ClientRequest no longer supports a shared default executor. The
    createPerRequestInstance
    parameter has been removed from
    ClientRequest.setDefaultExecutorClass()
    .
-   resteasy-doseta module no longer exists. It is now renamed to the
    resteasy-crypto module and also includes other things beyond doseta.
-   Doseta work has be refactored a bit and may have broken
    backward compatibility.
-   Jackson has been upgraded from 1.6.3 to 1.8.5. Let me know if there
    are any issues.
-   Form parameter processing behavior was modified because
    of RESTEASY-574. If you are having problems with form paramater
    processing on Tomcat after this fix, please log a JIRA or contact
    the resteasy-developers email list.
-   Some subtle changes were made to ExceptionMapper handling so that
    you can write ExceptionMappers for any exception thrown internally
    or within your application. See JIRA Issue RESTEASY-595 for
    more details. This may have an effect on existing applications that
    have an ExceptionMapper for RuntimeException in that you will start
    to see Resteasy internal exceptions being caught by this kind
    of ExceptionMapper.
-   The resteasy-cache (Server-side cache) will now invalidate the cache
    when a PUT, POST, or DELETE is done on a particular URI.

Migrating from 2.2.0 to 2.2.1 {#22_221}
=============================

-   Had to upgrade JAXB libs from 2.1.x to 2.2.4 as there was a
    concurrency bug in JAXB impl.

Migrating from 2.1.x to 2.2 {#21_22}
===========================

-   ClientRequest.getHeaders() always returns a copy. It also converts
    the values within ClientRequest.getHeadersAsObjects() to string. If
    you add values to the map returned by getHeaders() nothing happen.
    Instead add values to the getHeadersAsObjects() map. This allows
    non-string header objects to propagate through the MessageBodyWriter
    interceptor and ClientExecutor interceptor chains.

Migrating from 2.0.x to 2.1 {#20_21}
===========================

-   Slf4j is no longer the default logging mechanism for resteasy.
    Resteasy also no longer ships with SLF4J libraries. Please read the
    logging section in the Installation and Configuration chapter for
    more details.
-   The constructor used to instantiate resource and provider classes is
    now picked based on the requirements of the JAX-RS specification.
    Specifically, the public constructor with the most arguments
    is picked. This behavior varies from previous versions where a
    no-arg constructor is preferred.

Migrating from 1.2.x to 2.0 {#Migrating_to_Resteasy_12_20}
===========================

-   TJWS has been forked to fix some bugs. The new groupId is
    org.jboss.resteasy, the artifactId is tjws. It will match the
    resteasy distribution version
-   Please check out the JBoss 6 integration. It makes things a lot
    easier if you are deploying in that environment
-   There is a new Filter implementation that is the preferred
    deployment mechanism. Servlet-based deployments are still supported,
    but it is suggested you use to using a FilterDispatcher. See
    documentation for more details.
-   As per required by the spec List or array injection of empty values
    will return an empty collection or array, not null.
    I.e. (@QueryParam("name") List&lt;String&gt; param) param will be an
    empty List. Resteasy 1.2.x and earlier would return null.
-   We have forked TJWS, the servlet container used for embedded testing
    into the group org.jboss.resteasy, with the artifact id of tjws. You
    will need to remove these dependencies from your maven builds if you
    are using any part of the resteasy embeddable server. TJWS has a
    number of startup/shutdown race conditions we had to fix in order to
    make unit testing viable.
-   Spring integration compiled against Spring 3.0.3. It may or may not
    still work with 2.5.6 and lower

Migrating from 1.2.GA to 1.2.1.GA {#migrating_12_121}
=================================

Methods @Deprecated within 1.2.GA have been removed. This is in the
Client Framework and has to do with all references to Apache HTTP
Client. You must now create an ClientExecutor if you want to manage your
Apache HTTP Client sessions.

Migrating from 1.1 to 1.2 {#Migrating_to_Resteasy_1_1_1_2}
=========================

-   The resteasy-maven-import artifact has been renamed to resteasy-bom
-   Jettison and Fastinfoset have been broken out of the
    resteasy-jaxb-provider maven module. You will now need to include
    resteasy-jettison-provider or resteasy-fastinfoset-provider if you
    use either of these libraries.
-   The constructors for ClientRequest that have a HttpClient parameter
    (Apache Http Client 3.1 API) are now deprecated. They will be
    removed in the final release of 1.2. You must create a Apache hTTP
    Client Executor and pass it in as a parameter if you want to re-use
    existing Apache HttpClient sessions or do any special configuration.
    The same is true for the ProxyFactoyr methods.
-   Apache HttpClient 4.0 support is available if you want to use it.
    I've had some trouble with it so it is not the default
    implementation yet for the client framework.
-   It is no longer required to call RegisterBuiltin.register() to
    initialize the set of providers. Too many users forgot to do this
    (include myself!). You can turn this off by calling the static
    method ResteasyProviderFactory.setRegisterBuiltinByDefault(false)
-   The Embedded Container's API has changed to
    use org.jboss.resteasy.spi.ResteasyDeployment. Please see embedded
    documentation for more details.


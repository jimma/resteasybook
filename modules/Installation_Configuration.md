Installation/Configuration {#Installation_Configuration}
==========================

RESTEasy is installed and configured in different ways depending on
which environment you are running in. If you are running in JBoss AS
6-M4 (milestone 4) or higher, resteasy is already bundled and integrated
completely so there is very little you have to do. If you are running in
a different distribution, there is some manual installation and
configuration you will have to do.

Upgrading Resteasy Within JBoss AS 7 {#upgrading-as7}
====================================

Resteasy is bundled with JBoss AS 7. You may have the need to upgrade
Resteasy in AS7. The Resteasy distribution comes with a zip file called
resteasy-jboss-modules-3.1.0-SNAPSHOT.zip. Unzip this file within the
modules/ directory of the JBoss AS7 distribution. This will overwrite
some of the existing files there.

Upgrading Resteasy Within JBoss EAP 6.1 {#upgrading-eap61}
=======================================

Resteasy is bundled with JBoss EAP 6.1. You may have the need to upgrade
Resteasy in JBoss EAP 6.1. The Resteasy distribution comes with a zip
file called resteasy-jboss-modules-3.1.0-SNAPSHOT.zip. Unzip this file
within the modules/system/layers/base/ directory of the JBoss EAP 6.1
distribution. This will overwrite some of the existing files there.

> **Warning**
>
> Overwriting Resteasy modules in EAP 6 will place your installation in
> a less supportable state because Red Hat does not provide direct
> support for Resteasy 3.x or JAX-RS 2.x in EAP 6; this will only be
> supported by the community.

Upgrading Resteasy Within Wildfly {#upgrading-wildfly}
=================================

Resteasy is bundled with Wildfly. You may have the need to upgrade
Resteasy in Wildfly. The Resteasy distribution comes with a zip file
called resteasy-jboss-modules-wf8-3.1.0-SNAPSHOT.zip. Unzip this file
within the modules/system/layers/base/ directory of the Wildfly
distribution. This will overwrite some of the existing files there.

Configuring in JBoss AS 7, EAP, and Wildfly
===========================================

RESTEasy is bundled with JBoss/Wildfly and completely integrated as per
the requirements of Java EE 6. First you must at least provide an empty
web.xml file. You can of course deploy any custom servlet, filter or
security constraint you want to within your web.xml, but the least
amount of work is to create an empty web.xml file. Also, resteasy
context-params are available if you want to tweak turn on/off any
specific resteasy feature.

    <web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
    </web-app>

Since we're not using a jax-rs servlet mapping, we must define an
Application class that is annotated with the @ApplicationPath
annotation. If you return any empty set for by classes and singletons,
your WAR will be scanned for JAX-RS annotation resource and provider
classes.

    import javax.ws.rs.ApplicationPath;
    import javax.ws.rs.core.Application;

    @ApplicationPath("/root-path")
    public class MyApplication extends Application
    {
    }       

The Resteasy distribution has ported the "Restful Java" O'Reilly
workbook examples to AS7. You can find these under the directory
examples/oreilly-workbook-as7.

Resteasy Modules in AS7, EAP6.1, Wildfly
----------------------------------------

Resteasy and JAX-RS are automically loaded into your deployment's
classpath, if and only if you are deploying a JAX-RS Application. If you
only want to use the client library, you will have to create a
dependency for it within your deployment. Also, only some resteasy
features are automatically loaded. To bring in these libraries, you'll
have to create a jboss-deployment-structure.xml file within your WEB-INF
directory of your WAR file. Here's an example:

    <jboss-deployment-structure>
        <deployment>
            <dependencies>
                <module name="org.jboss.resteasy.resteasy-yaml-provider" services="import"/>
            </dependencies>
        </deployment>
    </jboss-deployment-structure>

The `services` attribute must be set to import for modules that have
default providers that must be registered. The following table specifies
which modules are loaded by default when JAX-RS services are deployed
and which aren't.

  Module Name                                      Loaded by Default   Description
  ------------------------------------------------ ------------------- -------------------------------------------------------------------------------------------------------------------------------------
  org.jboss.resteasy.resteasy-jaxrs                yes                 Core resteasy libraries for server and client. You will need to include this in your deployment if you are only using JAX-RS client
  org.jboss.resteasy.resteasy-cdi                  yes                 Resteasy CDI integration
  org.jboss.resteasy.resteasy-jaxb-provider        yes                 XML JAXB integration.
  org.jboss.resteasy.resteasy-atom-provider        yes                 Resteasy's atom library.
  org.jboss.resteasy.resteasy-jackson-provider     yes                 Integration with the JSON parser and object mapper, Jackson.
  org.jboss.resteasy.resteasy-jsapi                yes                 Resteasy's Javascript API.
  org.jboss.resteasy.resteasy-multipart-provider   yes                 Features for dealing with multipart formats.
  org.jboss.resteasy.resteasy-crypto               no                  S/MIME, DKIM, and support for other security formats.
  org.jboss.resteasy.jose-jwt                      no                  JOSE-JWT library. JSON Web Token.
  org.jboss.resteasy.resteasy-jettison-provider    no                  Alternative JAXB-like parser for JSON.
  org.jboss.resteasy.resteasy-yaml-provider        no                  YAML marshalling.
  org.jboss.resteasy.skeleton-key                  no                  OAuth2 support for AS7.

Standalone Resteasy in Servlet 3.0 Containers
=============================================

If you are using resteasy outside of JBoss/Wildfly, in a standalone
servlet container like Tomcat or Jetty you will need to include the core
Resteasy jars in your WAR file. Resteasy provides integration with
standalone Servlet 3.0 containers via the `ServletContainerInitializer`
integration interface. To use this, you must also include the
`resteasy-servlet-initializer` artifact in your WAR file as well.

    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-servlet-initializer</artifactId>
        <version>3.1.0-SNAPSHOT</version>
    </dependency>

We strongly suggest that you use Maven to build your WAR files as
RESTEasy is split into a bunch of different modules. You can see an
example Maven project in one of the examples in the examples/ directory.
If you are not using Maven,when you download RESTeasy and unzip it you
will see a lib/ directory that contains the libraries needed by
resteasy. Copy these into your /WEB-INF/lib directory. Place your JAX-RS
annotated class resources and providers within one or more jars within
/WEB-INF/lib or your raw class files within /WEB-INF/classes.

Standalone Resteasy in Older Servlet Containers
===============================================

The `resteasy-servlet-initializer` artifact will not work in Servlet
versions older than 3.0. You'll then have to manually declare the
Resteasy servlet in your WEB-INF/web.xml file of your WAR project. For
example:

    <web-app>
        <display-name>Archetype Created Web Application</display-name>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>
                org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
            </servlet-class>
            <init-param>
                <param-name>javax.ws.rs.Application</param-name>
                <param-value>com.restfully.shop.services.ShoppingApplication</param-value>
            </init-param>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>

    </web-app>

The Resteasy servlet is responsible for initializing some basic
components of RESTeasy.

Configuration Switches
======================

Resteasy receives configuration options from &lt;context-param&gt;
elements.

  Option Name                                   Default Value   Description
  --------------------------------------------- --------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  resteasy.servlet.mapping.prefix               no default      If the url-pattern for the Resteasy servlet-mapping is not /\*
  resteasy.providers                            no default      A comma delimited list of fully qualified @Provider class names you want to register
  resteasy.use.builtin.providers                true            Whether or not to register default, built-in @Provider classes. (Only available in 1.0-beta-5 and later)
  resteasy.resources                            no default      A comma delimited list of fully qualified JAX-RS resource class names you want to register
  resteasy.jndi.resources                       no default      A comma delimited list of JNDI names which reference objects you want to register as JAX-RS resources
  javax.ws.rs.Application                       no default      Fully qualified name of Application class to bootstrap in a spec portable way
  resteasy.media.type.mappings                  no default      Replaces the need for an Accept header by mapping file name extensions (like .xml or .txt) to a media type. Used when the client is unable to use a Accept header to choose a representation (i.e. a browser). See JAX-RS Content Negotiation chapter for more details.
  resteasy.language.mappings                    no default      Replaces the need for an Accept-Language header by mapping file name extensions (like .en or .fr) to a language. Used when the client is unable to use a Accept-Language header to choose a language (i.e. a browser). See JAX-RS Content Negotiation chapter for more details
  resteasy.document.expand.entity.references    false           Expand external entities in org.w3c.dom.Document documents and JAXB object representations
  resteasy.document.secure.processing.feature   true            Impose security constraints in processing org.w3c.dom.Document documents and JAXB object representations
  resteasy.document.secure.disableDTDs          true            Prohibit DTDs in org.w3c.dom.Document documents and JAXB object representations
  resteasy.wider.request.matching               true            Turns off the JAX-RS spec defined class-level expression filtering and instead tries to match version every method's full path.
  resteasy.use.container.form.params            true            Will use the HttpServletRequest.getParameterMap() method to obtain form parameters. Use this switch if you are calling this method within a servlet filter or eating the input stream within the filter.
  resteasy.rfc7232preconditions                 false           Enables [RFC7232 compliant HTTP preconditions handling](#Http_Precondition).

The resteasy.servlet.mapping.prefix &lt;context param&gt; variable must
be set if your servlet-mapping for the Resteasy servlet has a
url-pattern other than /\*. For example, if the url-pattern is

    <servlet-mapping>
    <servlet-name>Resteasy</servlet-name>
    <url-pattern>/restful-services/*</url-pattern>
    </servlet-mapping>

Then the value of resteasy-servlet.mapping.prefix must be:

    <context-param>
    <param-name>resteasy.servlet.mapping.prefix</param-name>
    <param-value>/restful-services</param-value>
    </context-param>

javax.ws.rs.core.Application {#javax.ws.rs.core.Application}
============================

The javax.ws.rs.core.Application class is a standard JAX-RS class that
you may implement to provide information on your deployment. It is
simply a class the lists all JAX-RS root resources and providers.

    /**
    * Defines the components of a JAX-RS application and supplies additional
    * metadata. A JAX-RS application or implementation supplies a concrete
    * subclass of this abstract class.
    */
    public abstract class Application
    {
        private static final Set<Object> emptySet = Collections.emptySet();

        /**
        * Get a set of root resource and provider classes. The default lifecycle
        * for resource class instances is per-request. The default lifecycle for
        * providers is singleton.
        * <p/>
        * <p>Implementations should warn about and ignore classes that do not
        * conform to the requirements of root resource or provider classes.
        * Implementations should warn about and ignore classes for which
        * {@link #getSingletons()} returns an instance. Implementations MUST
        * NOT modify the returned set.</p>
        *
        * @return a set of root resource and provider classes. Returning null
        * is equivalent to returning an empty set.
        */
        public abstract Set<Class<?>> getClasses();

        /**
        * Get a set of root resource and provider instances. Fields and properties
        * of returned instances are injected with their declared dependencies
        * (see {@link Context}) by the runtime prior to use.
        * <p/>
        * <p>Implementations should warn about and ignore classes that do not
        * conform to the requirements of root resource or provider classes.
        * Implementations should flag an error if the returned set includes
        * more than one instance of the same class. Implementations MUST
        * NOT modify the returned set.</p>
        * <p/>
        * <p>The default implementation returns an empty set.</p>
        *
        * @return a set of root resource and provider instances. Returning null
        * is equivalent to returning an empty set.
        */
        public Set<Object> getSingletons()
        {
            return emptySet;
        }

    }            

To use Application you must set a servlet init-param,
javax.ws.rs.Application with a fully qualified class that implements
Application. For example:

    <servlet>
        <servlet-name>Resteasy</servlet-name>
        <servlet-class>
            org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
        </servlet-class>
        <init-param>
            <param-name>javax.ws.rs.Application</param-name>
            <param-value>com.restfully.shop.services.ShoppingApplication</param-value>
        </init-param>
    </servlet>

If you have this set, you should probably turn off automatic scanning as
this will probably result in duplicate classes being registered.

RESTEasy as a ServletContextListener {#listener}
====================================

This section is pretty much deprecated if you are using a Servlet 3.0
container or higher. Skip it if you are and read the configuration
section above on installing in Servlet 3.0. The initialization of
RESTEasy can be performed within a ServletContextListener instead of
within the Servlet. You may need this if you are writing custom
Listeners that need to interact with RESTEasy at boot time. An example
of this is the RESTEasy Spring integration that requires a Spring
ServletContextListener. The
org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap class is a
ServletContextListener that configures an instance of an
ResteasyProviderFactory and Registry. You can obtain instances of a
ResteasyProviderFactory and Registry from the ServletContext attributes
org.jboss.resteasy.spi.ResteasyProviderFactory and
org.jboss.resteasy.spi.Registry. From these instances you can
programmatically interact with RESTEasy registration interfaces.

    <web-app>
       <listener>
          <listener-class>
             org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap
          </listener-class>
       </listener>

      <!-- ** INSERT YOUR LISTENERS HERE!!!! -->

       <servlet>
          <servlet-name>Resteasy</servlet-name>
          <servlet-class>
             org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
          </servlet-class>
       </servlet>

       <servlet-mapping>
          <servlet-name>Resteasy</servlet-name>
          <url-pattern>/resteasy/*</url-pattern>
       </servlet-mapping>

    </web-app>

RESTEasy as a servlet Filter {#filter}
============================

This section is pretty much deprecated if you are using a Servlet 3.0
container or higher. Skip it if you are and read the configuration
section above on installing in Servlet 3.0. The downside of running
Resteasy as a Servlet is that you cannot have static resources like
.html and .jpeg files in the same path as your JAX-RS services. Resteasy
allows you to run as a Filter instead. If a JAX-RS resource is not found
under the URL requested, Resteasy will delegate back to the base servlet
container to resolve URLs.

    <web-app>
        <filter>
            <filter-name>Resteasy</filter-name>
            <filter-class>
                org.jboss.resteasy.plugins.server.servlet.FilterDispatcher
            </filter-class>
            <init-param>
                <param-name>javax.ws.rs.Application</param-name>
                <param-value>com.restfully.shop.services.ShoppingApplication</param-value>
            </init-param>
        </filter>

        <filter-mapping>
            <filter-name>Resteasy</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

    </web-app>

RESTEasyLogging {#RESTEasyLogging}
===============

RESTEasy supports logging via java.util.logging, Log4j, or Slf4j. How it
picks which framework to delegate to is expressed in the following
algorithm:

-   If log4j is in the application's classpath, log4j will be used

-   If slf4j is in the application's classpath, slf4j will be used

-   java.util.logging is the default if neither log4j or slf4j is in the
    classpath

-   If the servlet context param resteasy.logger.type is set to JUL,
    LOG4J, or SLF4J will override this default behavior

The logging categories are still a work in progress, but the initial set
should make it easier to troubleshoot issues. Currently, the framework
has defined the following log categories:

  Category                               Function
  -------------------------------------- ----------------------------------------------------------
  org.jboss.resteasy.core                Logs all activity by the core RESTEasy implementation
  org.jboss.resteasy.plugins.providers   Logs all activity by RESTEasy entity providers
  org.jboss.resteasy.plugins.server      Logs all activity by the RESTEasy server implementation.
  org.jboss.resteasy.specimpl            Logs all activity by JAX-RS implementing classes
  org.jboss.resteasy.mock                Logs all activity by the RESTEasy mock framework



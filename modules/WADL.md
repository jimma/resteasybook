RESTEasy WADL Support {#WADL}
=====================

RESTEasy has its own support to generate WADL for its resources, and it
supports several different containers. The following text will show you
how to use this feature in different containers.

RESTEasy WADL Support for Servlet Container {#servlet_container}
===========================================

RESTEasy WADL uses ResteasyWadlServlet to support servlet container. It
can be registered into web.xml to enable WADL feature. Here is an
example to show the usages of ResteasyWadlServlet in web.xml:

    <servlet>
        <servlet-name>RESTEasy WADL</servlet-name>
        <servlet-class>org.jboss.resteasy.wadl.ResteasyWadlServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>RESTEasy WADL</servlet-name>
        <url-pattern>/application.xml</url-pattern>
    </servlet-mapping>

The preceding configuration in web.xml shows how to enable
ResteasyWadlServlet and mapped it to /application.xml. And then the WADL
can be accessed from the configured URL:

    /application.xml

RESTEasy WADL support for Sun JDK HTTP Server {#http_server}
=============================================

RESTEasy has provided a ResteasyWadlDefaultResource to generate WADL
info for its embedded containers. Here is and example to show how to use
it with RESTEasy's Sun JDK HTTP Server container:

    com.sun.net.httpserver.HttpServer httpServer =
        com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(port), 10);

    org.jboss.resteasy.plugins.server.sun.http.HttpContextBuilder contextBuilder = 
        new org.jboss.resteasy.plugins.server.sun.http.HttpContextBuilder();

    contextBuilder.getDeployment().getActualResourceClasses()
        .add(ResteasyWadlDefaultResource.class);
    contextBuilder.bind(httpServer);

    ResteasyWadlDefaultResource.getServices()
        .put("/",
            ResteasyWadlGenerator
                .generateServiceRegistry(contextBuilder.getDeployment()));

    httpServer.start();

From the above code example, we can see how ResteasyWadlDefaultResource
is registered into deployment:

    contextBuilder.getDeployment().getActualResourceClasses()
        .add(ResteasyWadlDefaultResource.class);

Another important thing is to use ResteasyWadlGenerator to generate the
WADL info for the resources in deployment at last:

    ResteasyWadlDefaultResource.getServices()
        .put("/",
            ResteasyWadlGenerator
                .generateServiceRegistry(contextBuilder.getDeployment()));

After the above configuration is set, then users can access
"/application.xml" to fetch the WADL info, because
ResteasyWadlDefaultResource has @PATH set to "/application.xml" as
default:

    @Path("/application.xml")
    public class ResteasyWadlDefaultResource

RESTEasy WADL support for Netty Container {#netty}
=========================================

RESTEasy WADL support for Netty Container is simliar to the support for
JDK HTTP Server. It also uses ResteasyWadlDefaultResource to serve
'/application.xml' and ResteasyWadlGenerator to generate WADL info for
resources. Here is the sample code:

    ResteasyDeployment deployment = new ResteasyDeployment();

    netty = new NettyJaxrsServer();
    netty.setDeployment(deployment);
    netty.setPort(port);
    netty.setRootResourcePath("");
    netty.setSecurityDomain(null);
    netty.start();

    deployment.getRegistry()
        .addPerRequestResource(ResteasyWadlDefaultResource.class);        
    ResteasyWadlDefaultResource.getServices()
        .put("/", ResteasyWadlGenerator.generateServiceRegistry(deployment));

Please note for all the embedded containers like JDK HTTP Server and
Netty Container, if the resources in the deployment changes at runtime,
the ResteasyWadlGenerator.generateServiceRegistry() need to be re-run to
refresh the WADL info.

RESTEasy WADL Support for Undertow Container {#undertow}
============================================

The RESTEasy Undertow Container is a embedded Servlet Container, and
RESTEasy WADL provides a connector to it. To use RESTEasy Undertow
Container together with WADL support, you need to add these three
components into your maven dependencies:

    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-wadl</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-wadl-undertow-connector</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-undertow</artifactId>
        <version>${project.version}</version>
    </dependency>

The resteasy-wadl-undertow-connector provides a WadlUndertowConnector to
help you to use WADL in RESTEasy Undertow Container. Here is the code
example:

    UndertowJaxrsServer server = new UndertowJaxrsServer().start();
    WadlUndertowConnector connector = new WadlUndertowConnector();
    connector.deployToServer(server, MyApp.class);

The MyApp class shown in above code is a standard JAX-RS 2.0 Application
class in your project:

                
    @ApplicationPath("/base")
    public static class MyApp extends Application {
        @Override
        public Set<Class<?>> getClasses() {
            HashSet<Class<?>> classes = new HashSet<Class<?>>();
            classes.add(YourResource.class);
            return classes;
        }
    }

After the Application is deployed to the UndertowJaxrsServer via
WadlUndertowConnector, you can access the WADL info at
"/application.xml" prefixed by the @ApplicationPath in your Application
class. If you want to override the @ApplicationPath, you can use the
other method in WadlUndertowConnector:

                
    public UndertowJaxrsServer deployToServer(UndertowJaxrsServer server, Class<? extends Application> application, String contextPath)
                
            

The "deployToServer" method shown above accepts a "contextPath"
parameter, which you can use to override the @ApplicationPath value in
the Application class.

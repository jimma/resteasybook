Embedded Containers {#RESTEasy_Embedded_Container}
===================

Resteasy has a few different plugins for different embedabble HTTP
and/or Servlet containers if use Resteasy in a test environment, or
within an environment where you do not want a Servlet engine dependency.

Undertow {#undertow}
========

Undertow is a new Servlet Container that is used by Wildfly (JBoss
Community Server). You can embed Undertow as you wish. Here's a a test
that shows it in action.

    import io.undertow.servlet.api.DeploymentInfo;
    import org.jboss.resteasy.plugins.server.undertow.UndertowJaxrsServer;
    import org.jboss.resteasy.test.TestPortProvider;
    import org.junit.AfterClass;
    import org.junit.Assert;
    import org.junit.BeforeClass;
    import org.junit.Test;

    import javax.ws.rs.ApplicationPath;
    import javax.ws.rs.GET;
    import javax.ws.rs.Path;
    import javax.ws.rs.Produces;
    import javax.ws.rs.client.Client;
    import javax.ws.rs.client.ClientBuilder;
    import javax.ws.rs.core.Application;
    import java.util.HashSet;
    import java.util.Set;

    /**
     * @author <a href="mailto:bill@burkecentral.com">Bill Burke</a>
     * @version $Revision: 1 $
     */
    public class UndertowTest
    {
       private static UndertowJaxrsServer server;

       @Path("/test")
       public static class Resource
       {
          @GET
          @Produces("text/plain")
          public String get()
          {
             return "hello world";
          }
       }

       @ApplicationPath("/base")
       public static class MyApp extends Application
       {
          @Override
          public Set<Class<?>> getClasses()
          {
             HashSet<Class<?>> classes = new HashSet<Class<?>>();
             classes.add(Resource.class);
             return classes;
          }
       }

       @BeforeClass
       public static void init() throws Exception
       {
          server = new UndertowJaxrsServer().start();
       }

       @AfterClass
       public static void stop() throws Exception
       {
          server.stop();
       }

       @Test
       public void testApplicationPath() throws Exception
       {
          server.deploy(MyApp.class);
          Client client = ClientBuilder.newClient();
          String val = client.target(TestPortProvider.generateURL("/base/test"))
                             .request().get(String.class);
          Assert.assertEquals("hello world", val);
          client.close();
       }

       @Test
       public void testApplicationContext() throws Exception
       {
          server.deploy(MyApp.class, "/root");
          Client client = ClientBuilder.newClient();
          String val = client.target(TestPortProvider.generateURL("/root/test"))
                             .request().get(String.class);
          Assert.assertEquals("hello world", val);
          client.close();
       }

       @Test
       public void testDeploymentInfo() throws Exception
       {
          DeploymentInfo di = server.undertowDeployment(MyApp.class);
          di.setContextPath("/di");
          di.setDeploymentName("DI");
          server.deploy(di);
          Client client = ClientBuilder.newClient();
          String val = client.target(TestPortProvider.generateURL("/di/base/test"))
                             .request().get(String.class);
          Assert.assertEquals("hello world", val);
          client.close();
       }
    }

Sun JDK HTTP Server {#http-server}
===================

The Sun JDK comes with a simple HTTP server implementation
(com.sun.net.httpserver.HttpServer) which you can run Resteasy on top
of.

     
          HttpServer httpServer = HttpServer.create(new InetSocketAddress(port), 10);
          contextBuilder = new HttpContextBuilder();
          contextBuilder.getDeployment().getActualResourceClasses().add(SimpleResource.class);
          HttpContext context = contextBuilder.bind(httpServer);
          context.getAttributes().put("some.config.info", "42");
          httpServer.start();

          contextBuilder.cleanup();
          httpServer.stop(0);
        

Create your HttpServer the way you want then use the
org.jboss.resteasy.plugins.server.sun.http.HttpContextBuilder to
initialize Resteasy and bind it to an HttpContext. The HttpContext
attributes are available by injecting in a
org.jboss.resteasy.spi.ResteasyConfiguration interface using @Context
within your provider and resource classes.

Maven project you must include is:

     
      <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-jdk-http</artifactId>
          <version>3.1.0-SNAPSHOT</version>
      </dependency>

TJWS Embeddable Servlet Container {#TJES}
=================================

RESTeasy integrates with the TJWS Embeddable Servlet container. It comes
with this distribution, or you can reference the Maven artifact. You
must also provide a servlet API dependency as well.

     
      <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>tjws</artifactId>
          <version>3.1.0-SNAPSHOT</version>
      </dependency>

      <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>servlet-api</artifactId>
          <version>2.5</version>
      </dependency>

From the distribution, move the jars in resteasy-jaxrs.war/WEB-INF/lib
into your classpath. You must both programmatically register your JAX-RS
beans using the embedded server's Registry. Here's an example:


    @Path("/")
    public class MyResource {

       @GET
       public String get() { return "hello world"; }
     

       public static void main(String[] args) throws Exception 
       {
          TJWSEmbeddedJaxrsServer tjws = new TJWSEmbeddedJaxrsServer();
          tjws.setPort(8080);
          tjws.start();
          tjws.getRegistry().addPerRequestResource(RestEasy485Resource.class);
       }
    }

The server can either host non-encrypted or SSL based resources, but not
both. See the Javadoc for TJWSEmbeddedJaxrsServer as well as its
superclass TJWSServletServer. The TJWS website is also a good place for
information.

If you want to use Spring, see the SpringBeanProcessor. Here's a
pseudo-code example


       public static void main(String[] args) throws Exception 
       {
          final TJWSEmbeddedJaxrsServer tjws = new TJWSEmbeddedJaxrsServer();
          tjws.setPort(8081);

          tjws.start();
          org.jboss.resteasy.plugins.server.servlet.SpringBeanProcessor processor = new SpringBeanProcessor(tjws.getDeployment().getRegistry(), tjws.getDeployment().getFactory();
          ConfigurableBeanFactory factory = new XmlBeanFactory(...);
          factory.addBeanPostProcessor(processor);
       }

**NOTE:** TJWS is now deprecated. Consider using the more modern
Undertow.

Netty {#Netty}
=====

Resteasy has integration with the popular Netty project as well..

     
       public static void start(ResteasyDeployment deployment) throws Exception
       {
          netty = new NettyJaxrsServer();
          netty.setDeployment(deployment);
          netty.setPort(TestPortProvider.getPort());
          netty.setRootResourcePath("");
          netty.setSecurityDomain(null);
          netty.start();
       }
        

Maven project you must include is:

     
      <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-netty</artifactId>
          <version>3.1.0-SNAPSHOT</version>
      </dependency>

Vert.x {#Vert}
======

Resteasy has integration with the popular Vert.x project as well..

     
       public static void start(VertxResteasyDeployment deployment) throws Exception
       {
          VertxJaxrsServer server = new VertxJaxrsServer();
          server.setDeployment(deployment);
          server.setPort(TestPortProvider.getPort());
          server.setRootResourcePath("");
          server.setSecurityDomain(null);
          server.start();
       }
        

Maven project you must include is:

     
      <dependency>
          <groupId>org.jboss.resteasy</groupId>
          <artifactId>resteasy-vertx</artifactId>
          <version>3.1.0-SNAPSHOT</version>
      </dependency>
        

The server will bootstrap its own Vert.x instance and Http server.

When a resource is called, it is done with the Vert.x Event Loop thread,
keep in mind to not block this thread and respect the Vert.x programming
model, see the related Vert.x [manual
page](http://vertx.io/docs/vertx-core/java/#_don_t_block_me).

Vert.x extends the Resteasy registry to provide a new binding scope that
creates resources per Event Loop:

     
      VertxResteasyDeployment deployment = new VertxResteasyDeployment();
      // Create an instance of resource per Event Loop
      deployment.getRegistry().addPerInstanceResource(Resource.class);
        

The per instance binding scope caches the same resource instance for
each event loop providing the same concurrency model than a verticle
deployed multiple times.

Vert.x can also embed a Resteasy deployment, making easy to use Jax-RS
annotated controller in Vert.x applications:

     
      Vertx vertx = Vertx.vertx();
      HttpServer server = vertx.createHttpServer();

      // Set an handler calling Resteasy
      server.requestHandler(new VertxRequestHandler(vertx, deployment));

      // Start the server
      server.listen(8080, "localhost");
        

Vert.x objects can be injected in annotated resources:

     
      @GET
      @Path("/somepath")
      @Produces("text/plain")
      public String context(
          @Context io.vertx.core.Context context,
          @Context io.vertx.core.Vertx vertx,
          @Context io.vertx.core.http.HttpServerRequest req,
          @Context io.vertx.core.http.HttpServerResponse resp) {
        return "the-response";
      }
        

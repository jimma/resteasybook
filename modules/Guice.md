Guice 3.0 Integration {#Guice1}
=====================

RESTEasy has some simple integration with Guice 3.0. RESTEasy will scan
the binding types for a Guice Module for @Path and @Provider
annotations. It will register these bindings with RESTEasy. The
guice-hello project that comes in the RESTEasy examples/ directory gives
a nice example of this.

    @Path("hello")
    public class HelloResource
    {
       @GET
       @Path("{name}")
       public String hello(@PathParam("name") final String name) {
          return "Hello " + name;
       }
    }

First you start off by specifying a JAX-RS resource class. The
HelloResource is just that. Next you create a Guice Module class that
defines all your bindings:

    import com.google.inject.Module;
    import com.google.inject.Binder;

    public class HelloModule implements Module
    {
        public void configure(final Binder binder)
        {
           binder.bind(HelloResource.class);
        }
    }

You put all these classes somewhere within your WAR WEB-INF/classes or
in a JAR within WEB-INF/lib. Then you need to create your web.xml file.
You need to use the GuiceResteasyBootstrapServletContextListener as
follows


    <web-app>
        <display-name>Guice Hello</display-name>

        <context-param>
            <param-name>resteasy.guice.modules</param-name>
            <param-value>org.jboss.resteasy.examples.guice.hello.HelloModule</param-value>
        </context-param>

        <listener>
            <listener-class>
                org.jboss.resteasy.plugins.guice.GuiceResteasyBootstrapServletContextListener
            </listener-class>
        </listener>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>
                org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
            </servlet-class>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>

    </web-app>

GuiceResteasyBootstrapServletContextListener is a subclass of
ResteasyBootstrap, so you can use any other RESTEasy configuration
option within your web.xml file. Also notice that there is a
resteasy.guice.modules context-param. This can take a comma delimited
list of class names that are Guice Modules.

Request Scope {#request_scope}
=============

Add the RequestScopeModule to your modules to allow objects to be scoped
to the HTTP request by adding the @RequestScoped annotation to your
class. All the objects injectable via the @Context annotation are also
injectable, except ServletConfig and ServletContext.


    import javax.inject.Inject;
    import javax.servlet.http.HttpServletRequest;
    import javax.ws.rs.core.Context;

    import org.jboss.resteasy.plugins.guice.RequestScoped;

    @RequestScoped
    public class MyClass
    {
        @Inject @Context
        private HttpRequest request;
    }

Binding JAX-RS utilities {#binding_jax-rs}
========================

Add the JaxrsModule to bind javax.ws.rs.ext.RuntimeDelegate,
javax.ws.rs.core.Response.ResponseBuilder, javax.ws.rs.core.UriBuilder,
javax.ws.rs.core.Variant.VariantListBuilder and
org.jboss.resteasy.client.ClientExecutor.

Configuring Stage {#config_stage}
=================

You can configure the stage Guice uses to deploy your modules by
specific a context param, resteasy.guice.stage. If this value is not
specified, Resteasy uses whatever Guice's default is.


    <web-app>
        <display-name>Guice Hello</display-name>

        <context-param>
            <param-name>resteasy.guice.modules</param-name>
            <param-value>org.jboss.resteasy.examples.guice.hello.HelloModule</param-value>
        </context-param>

        <context-param>
            <param-name>resteasy.guice.stage</param-name>
            <param-value>PRODUCTION</param-value>
        </context-param>

        <listener>
            <listener-class>
                org.jboss.resteasy.plugins.guice.GuiceResteasyBootstrapServletContextListener
            </listener-class>
        </listener>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>
                org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
            </servlet-class>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>

    </web-app>

Custom Injector creation {#custom_injector}
========================

GuiceResteasyBootstrapServletContextListener can be extended to allow
more flexibility in the way the Injector and Modules are created. Three
methods can be overridden: getModules(), withInjector() and getStage().
Register your subclass as the listener in the web.xml.

Override getModules() when you need to pass arguments to your modules'
constructor or perform more complex operations.

Override withInjector(Injector) when you need to interact with the
Injector after it has been created.

Override getStage(ServletContext) to set the Stage yourself.


    <web-app>
        <!-- other tags omitted -->
        <listener>
          <listener-class>
             org.jboss.resteasy.plugins.guice.GuiceResteasyBootstrapServletContextListener
          </listener-class>
        </listener>
    </web-app>

    public class MyServletContextListener extends GuiceResteasyBootstrapServletContextListener
    {

        @Override
        protected List<? extends Module> getModules(ServletContext context)
        {
            return Arrays.asList(new JpaPersistModule("consulting_hours"), new MyModule());
        }
        
        @Override
        public void withInjector(Injector injector)
        {
            injector.getInstance(PersistService.class).start();
        }
    }


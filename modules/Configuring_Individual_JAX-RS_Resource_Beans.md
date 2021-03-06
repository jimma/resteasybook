Configuring Individual JAX-RS Resource Beans {#Configuring_Individual_JAX-RS_Resource_Beans}
============================================

If you are scanning your path for JAX-RS annotated resource beans, your
beans will be registered in per-request mode. This means an instance
will be created per HTTP request served. Generally, you will need
information from your environment. If you are running within a servlet
container using the WAR-file distribution, in Beta-2 and lower, you can
only use the JNDI lookups to obtain references to Java EE resources and
configuration information. In this case, define your EE configuration
(i.e. ejb-ref, env-entry, persistence-context-ref, etc...) within
web.xml of the resteasy WAR file. Then within your code do jndi lookups
in the java:comp namespace. For example:

web.xml


    <ejb-ref>
      <ejb-ref-name>ejb/foo</ejb-ref-name>
      ...
    </ejb-ref>

resource code:

    @Path("/")
    public class MyBean {

       public Object getSomethingFromJndi() {
          new InitialContext.lookup("java:comp/ejb/foo");
       }
    ...
    }

You can also manually configure and register your beans through the
Registry. To do this in a WAR-based deployment, you need to write a
specific ServletContextListener to do this. Within the listener, you can
obtain a reference to the registry as follows:


    public class MyManualConfig implements ServletContextListener
    {
       public void contextInitialized(ServletContextEvent event)
       {

          Registry registry = (Registry) event.getServletContext().getAttribute(Registry.class.getName());

       }
    ...
    }

Please also take a look at our Spring Integration as well as the
Embedded Container's Spring Integration

EJB Integration {#RESTEasy_EJB_Integration}
===============

To integrate with EJB you must first modify your EJB's published
interfaces. Resteasy currently only has simple portable integration with
EJBs so you must also manually configure your Resteasy WAR.

Resteasy currently only has simple integration with EJBs. To make an EJB
a JAX-RS resource, you must annotate an SLSB's @Remote or @Local
interface with JAX-RS annotations:

    @Local
    @Path("/Library")
    public interface Library {
       
       @GET
       @Path("/books/{isbn}")
       public String getBook(@PathParam("isbn") String isbn);
    }

    @Stateless
    public class LibraryBean implements Library {

    ...

    }

Next, in RESTeasy's web.xml file you must manually register the EJB with
RESTeasy using the resteasy.jndi.resources &lt;context-param&gt;

    <web-app>
       <display-name>Archetype Created Web Application</display-name>
       <context-param>
          <param-name>resteasy.jndi.resources</param-name>
          <param-value>LibraryBean/local</param-value>
       </context-param>

       <listener>
          <listener-class>org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap</listener-class>
       </listener>

       <servlet>
          <servlet-name>Resteasy</servlet-name>
          <servlet-class>org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher</servlet-class>
       </servlet>

       <servlet-mapping>
          <servlet-name>Resteasy</servlet-name>
          <url-pattern>/*</url-pattern>
       </servlet-mapping>

    </web-app>

This is the only portable way we can offer EJB integration. Future
versions of RESTeasy will have tighter integration with JBoss AS so you
do not have to do any manual registrations or modifications to web.xml.
For right now though, we're focusing on portability.

If you're using Resteasy with an EAR and EJB, a good structure to have
is:

    my-ear.ear
    |------myejb.jar
    |------resteasy-jaxrs.war
           |
           ----WEB-INF/web.xml
           ----WEB-INF/lib (nothing)
    |------lib/
           |
           ----All Resteasy jar files

From the distribution, remove all libraries from WEB-INF/lib and place
them in a common EAR lib. OR. Just place the Resteasy jar dependencies
in your application server's system classpath. (i.e. In JBoss put them
in server/default/lib)

An example EAR project is available from our testsuite here.

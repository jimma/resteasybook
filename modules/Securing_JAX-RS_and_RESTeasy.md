Securing JAX-RS and RESTeasy {#Securing_JAX-RS_and_RESTeasy}
============================

Because Resteasy is deployed as a servlet, you must use standard web.xml
constraints to enable authentication and authorization.

Unfortunately, web.xml constraints do not mesh very well with JAX-RS in
some situations. The problem is that web.xml URL pattern matching is
very very limited. URL patterns in web.xml only support simple
wildcards, so JAX-RS resources like:

    /{pathparam1}/foo/bar/{pathparam2} 

Cannot be mapped as a web.xml URL pattern like:

    /*/foo/bar/*

To get around this problem you will need to use the security annotations
defined below on your JAX-RS methods. You will still need to set up some
general security constraint elements in web.xml to turn on
authentication.

Resteasy JAX-RS supports the @RolesAllowed, @PermitAll and @DenyAll
annotations on JAX-RS methods. By default though, Resteasy does not
recognize these annotations. You have to configure Resteasy to turn on
role-based security by setting a context parameter. NOTE!!! Do not turn
on this switch if you are using EJBs. The EJB container will provide
this functionality instead of Resteasy.


    <web-app>
    ...
       <context-param>
          <param-name>resteasy.role.based.security</param-name>
          <param-value>true</param-value>
       </context-param>
    </web-app>

There is a bit of quirkiness with this approach. You will have to
declare all roles used within the Resteasy JAX-RS war file that you are
using in your JAX-RS classes and set up a security constraint that
permits all of these roles access to every URL handled by the JAX-RS
runtime. You'll just have to trust that Resteasy JAX-RS authorizes
properly.

How does Resteasy do authorization? Well, its really simple. It just
sees if a method is annotated with @RolesAllowed and then just does
HttpServletRequest.isUserInRole. If one of the @RolesAllowed passes,
then allow the request, otherwise, a response is sent back with a 401
(Unauthorized) response code.

So, here's an example of a modified RESTEasy WAR file. You'll notice
that every role declared is allowed access to every URL controlled by
the Resteasy servlet.


    <web-app>

       <context-param>
          <param-name>resteasy.role.based.security</param-name>
          <param-value>true</param-value>
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

       <security-constraint>
          <web-resource-collection>
             <web-resource-name>Resteasy</web-resource-name>
             <url-pattern>/security</url-pattern>
          </web-resource-collection>
           <auth-constraint>
             <role-name>admin</role-name>
             <role-name>user</role-name>
          </auth-constraint>
      </security-constraint>

       <login-config>
          <auth-method>BASIC</auth-method>
          <realm-name>Test</realm-name>
       </login-config>

       <security-role>
          <role-name>admin</role-name>
       </security-role>
       <security-role>
          <role-name>user</role-name>
       </security-role>


    </web-app>



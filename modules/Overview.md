Overview {#Overview}
========

JAX-RS, JSR-311, is a new JCP specification that provides a Java API for
RESTful Web Services over the HTTP protocol. Resteasy is an portable
implementation of this specification which can run in any Servlet
container. Tighter integration with JBoss Application Server is also
available to make the user experience nicer in that environment. While
JAX-RS is only a server-side specification, Resteasy has innovated to
bring JAX-RS to the client through the RESTEasy JAX-RS Client Framework.
This client-side framework allows you to map outgoing HTTP requests to
remote servers using JAX-RS annotations and interface proxies.

-   JAX-RS implementation
-   Portable to any app-server/Tomcat that runs on JDK 5 or higher
-   Embeddable server implementation for junit testing
-   EJB and Spring integration
-   Client framework to make writing HTTP clients easy (JAX-RS only
    define server bindings)


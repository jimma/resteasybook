CDI Integration {#CDI}
===============

This module provides integration with JSR-299 (Contexts and Dependency
Injection for the Java EE platform)

Using CDI beans as JAX-RS components {#cdi_beans}
====================================

Both the JAX-RS and CDI specifications introduce their own component
model. On the one hand, every class placed in a CDI archive that
fulfills a set of basic constraints is implicitly a CDI bean. On the
other hand, explicit decoration of your Java class with `@Path` or
`@Provider` is required for it to become a JAX-RS component. Without the
integration code, annotating a class suitable for being a CDI bean with
JAX-RS annotations leads into a faulty result (JAX-RS component not
managed by CDI) The resteasy-cdi module is a bridge that allows RESTEasy
to work with class instances obtained from the CDI container.

During a web service invocation, resteasy-cdi asks the CDI container for
the managed instance of a JAX-RS component. Then, this instance is
passed to RESTEasy. If a managed instance is not available for some
reason (the class is placed in a jar which is not a bean deployment
archive), RESTEasy falls back to instantiating the class itself.

As a result, CDI services like injection, lifecycle management, events,
decoration and interceptor bindings can be used in JAX-RS components.

Default scopes {#default_scopes}
==============

A CDI bean that does not explicitly define a scope is `@Dependent`
scoped by default. This pseudo scope means that the bean adapts to the
lifecycle of the bean it is injected into. Normal scopes (request,
session, application) are more suitable for JAX-RS components as they
designate component's lifecycle boundaries explicitly. Therefore, the
resteasy-cdi module alters the default scoping in the following way:

-   If a JAX-RS root resource does not define a scope explicitly, it is
    bound to the Request scope.

-   If a JAX-RS Provider or `javax.ws.rs.Application` subclass does not
    define a scope explicitly, it is bound to the Application scope.

> **Warning**
>
> Since the scope of all beans that do not declare a scope is modified
> by resteasy-cdi, this affects session beans as well. As a result, a
> conflict occurs if the scope of a stateless session bean or singleton
> is changed automatically as the spec prohibits these components to be
> @RequestScoped. Therefore, you need to explicitly define a scope when
> using stateless session beans or singletons. This requirement is
> likely to be removed in future releases.

Configuration within JBoss 6 M4 and Higher {#config_within_jboss}
==========================================

CDI integration is provided with no additional configuration with JBoss
AS 6-M4 and higher.

Configuration with different distributions {#config_diff_distributions}
==========================================

Provided you have an existing RESTEasy application, all that needs to be
done is to add the resteasy-cdi jar into your project's `WEB-INF/lib`
directory. When using maven, this can be achieve by defining the
following dependency.

``` {.xml}
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-cdi</artifactId>
    <version>${project.version}</version>
</dependency>
```

Furthermore, when running a pre-Servlet 3 container, the following
context parameter needs to be specified in web.xml. (This is done
automatically via web-fragment in a Servlet 3 environment)

``` {.xml}
<context-param>
    <param-name>resteasy.injector.factory</param-name>
    <param-value>org.jboss.resteasy.cdi.CdiInjectorFactory</param-value>
</context-param>
```

When deploying an application to a Servlet container that does not
support CDI out of the box (Tomcat, Jetty, Google App Engine), a CDI
implementation needs to be added first. [Weld-servlet
module](http://docs.jboss.org/weld/reference/latest/en-US/html/environments.html)
can be used for this purpose.

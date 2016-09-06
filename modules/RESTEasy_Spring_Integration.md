Spring Integration {#RESTEasy_Spring_Integration}
==================

RESTEasy integrates with Spring 3.0.x. We are interested in other forms
of Spring integration, so please help contribute.

Basic Integration {#SpringBeanProcessor}
-----------------

For Maven users, you must use the resteasy-spring artifact. Otherwise,
the jar is available in the downloaded distribution.


    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-spring</artifactId>
        <version>whatever version you are using</version>
    </dependency>

RESTeasy comes with its own Spring ContextLoaderListener that registers
a RESTeasy specific BeanPostProcessor that processes JAX-RS annotations
when a bean is created by a BeanFactory. What does this mean? RESTeasy
will automatically scan for @Provider and JAX-RS resource annotations on
your bean class and register them as JAX-RS resources.

Here is what you have to do with your web.xml file

    <web-app>
       <display-name>Archetype Created Web Application</display-name>

       <listener>
          <listener-class>org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap</listener-class>
       </listener>

       <listener>
          <listener-class>org.jboss.resteasy.plugins.spring.SpringContextLoaderListener</listener-class>
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

The SpringContextLoaderListener must be declared after ResteasyBootstrap
as it uses ServletContext attributes initialized by it.

If you do not use a Spring ContextLoaderListener to create your bean
factories, then you can manually register the RESTeasy
BeanFactoryPostProcessor by allocating an instance of
org.jboss.resteasy.plugins.spring.SpringBeanProcessor. You can obtain
instances of a ResteasyProviderFactory and Registry from the
ServletContext attributes org.jboss.resteasy.spi.ResteasyProviderFactory
and org.jboss.resteasy.spi.Registry. (Really the string FQN of these
classes). There is also a
org.jboss.resteasy.plugins.spring.SpringBeanProcessorServletAware, that
will automatically inject references to the Registry and
ResteasyProviderFactory from the Servlet Context. (that is, if you have
used RestasyBootstrap to bootstrap Resteasy).

Our Spring integration supports both singletons and the "prototype"
scope. RESTEasy handles injecting @Context references. Constructor
injection is not supported though. Also, with the "prototype" scope,
RESTEasy will inject any @\*Param annotated fields or setters before the
request is dispatched.

NOTE: You can only use auto-proxied beans with our base Spring
integration. You will have undesirable affects if you are doing
handcoded proxying with Spring, i.e., with ProxyFactoryBean. If you are
using auto-proxied beans, you will be ok.

Spring MVC Integration {#SpringMVC}
----------------------

RESTEasy can also integrate with the Spring DispatcherServlet. The
advantages of using this are that you have a simpler web.xml file, you
can dispatch to either Spring controllers or Resteasy from under the
same base URL, and finally, the most important, you can use Spring
ModelAndView objects as return arguments from @GET resource methods.
Setup requires you using the Spring DispatcherServlet in your web.xml
file, as well as importing the springmvc-resteasy.xml file into your
base Spring beans xml file. Here's an example web.xml file:

    <web-app>
       <display-name>Archetype Created Web Application</display-name>

       <servlet>
          <servlet-name>Spring</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet;</servlet-class>
       </servlet>

       <servlet-mapping>
          <servlet-name>Spring</servlet-name>
          <url-pattern>/*</url-pattern>
       </servlet-mapping>


    </web-app>

Then within your main Spring beans xml, import the
springmvc-resteasy.xml file


    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-2.5.xsd
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    ">

        <!-- Import basic SpringMVC Resteasy integration -->
        <import resource="classpath:springmvc-resteasy.xml"/>
    ....

You can specify resteasy configuration options by overriding the
resteasy.deployment bean which is an instance of
org.jboss.resteasy.spi.ResteasyDeployment. Here's an example of adding
media type suffix mappings as well as enabling the Resteasy asynchronous
job service.


    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation="
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-2.5.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            ">

        <!-- Import basic SpringMVC Resteasy integration -->
        <import resource="classpath:springmvc-resteasy.xml" />

        <!-- override the bean definition for deployment -->
        <bean id="resteasy.deployment" class="org.jboss.resteasy.spi.ResteasyDeployment" init-method="start" destroy-method="stop">
            <property name="asyncJobServiceEnabled" value="true"/>
            <property name="mediaTypeMappings">
                <map>
                    <entry key="json" value="application/json" />
                    <entry key="xml" value="application/xml" />
                </map>
            </property>
        </bean>
    ...

Upgrading in WildFly
--------------------

**Note.** As noted in Chapter 3, the The Resteasy distribution comes
with a zip file called resteasy-jboss-modules-wf8-&lt;version&gt;.zip,
which can be unzipped into the modules/system/layers/base/ directory of
Wildfly to upgrade to a new version of Resteasy. Because of the way
resteasy-spring is used in WildFly, after unzipping the zip file, it is
also necessary to remove the old resteasy-spring jar from
modules/system/layers/base/org/jboss/resteasy/resteasy-spring/main/bundled/resteasy-spring-jar.

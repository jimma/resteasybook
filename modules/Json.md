JSON Support via Jackson {#json}
========================

Besides the Jettision JAXB adapter for JSON, Resteasy also support
integration with the Jackson project. Many users find the output from
Jackson much much nicer than the Badger format or Mapped format provided
by Jettison. Jackson lives at http://jackson.codehaus.org. It allows you
to easily marshal Java objects to and from JSON. It has a Java Bean
based model as well as JAXB like APIs. Resteasy integrates with the
JavaBean model as described at this url:
http://jackson.codehaus.org/Tutorial.

While Jackson does come with its own JAX-RS integration, Resteasy
expanded it a little. To include it within your project, just add this
maven dependency to your build. Resteasy supports both Jackson 1.9.x and
Jackson 2.2.x. Read further on how to use each.

Using Jackson 1.9.x Outside of JBoss AS7
========================================

If you're deploying Resteasy outside of JBoss AS7 add the resteasy
jackson provder to your WAR pom.xml build

    <dependency>
       <groupId>org.jboss.resteasy</groupId>
       <artifactId>resteasy-jackson-provider</artifactId>
       <version>3.1.0-SNAPSHOT</version>
    </dependency>

Using Jackson 1.9.x Inside of JBoss AS7
=======================================

If you're deploying Resteasy with JBoss AS7, there's nothing you need to
do except to make sure you've updated your AS7 distribution with the
latest and greatest Resteasy. See the installation sectio of this
documentation for more details.

Using Jackson 2.2.x Outside of JBoss AS7
========================================

If you're deploying Resteasy outside of JBoss AS7 add the resteasy
jackson provder to your WAR pom.xml build

    <dependency>
       <groupId>org.jboss.resteasy</groupId>
       <artifactId>resteasy-jackson2-provider</artifactId>
       <version>3.1.0-SNAPSHOT</version>
    </dependency>

Using Jackson 2.2.x Inside of JBoss AS7
=======================================

If you want to use Jackson 2.2.x inside of JBoss AS7 you'll have to
create a jboss-deployment-structure.xml file within your WEB-INF
directory. By default AS7 includes the Jackson 1.9.x JAX-RS integration,
so you'll want to exclude it from your dependencies, and add the
jackson2 ones.

    <jboss-deployment-structure>
        <deployment>
            <exclusions>
               <module name="org.jboss.resteasy.resteasy-jackson-provider"/>
            </exclusions>
            <dependencies>
                <module name="org.jboss.resteasy.resteasy-jackson2-provider" services="import"/>
            </dependencies>
        </deployment>
    </jboss-deployment-structure> 

Additional Resteasy Specifics
=============================

The first extra piece that Resteasy added to the integration was to
support "application/\*+json". Jackson would only accept
"application/json" and "text/json" as valid media types. This allows you
to create json-based media types and still let Jackson marshal things
for you. For example:

    @Path("/customers")
    public class MyService {

        @GET
        @Produces("application/vnd.customer+json")
        public Customer[] getCustomers() {}
    }

Another problem that occurs is when you are using the Resteasy JAXB
providers alongside Jackson. You may want to use Jettison and JAXB to
output your JSON instead of Jackson. In this case, you must either not
install the Jackson provider, or use the annotation
@org.jboss.resteasy.annotations.providers.NoJackson on your JAXB
annotated classes. For example:

    @XmlRootElement
    @NoJackson
    public class Customer {...}

    @Path("/customers")
    public class MyService {

        @GET
        @Produces("application/vnd.customer+json")
        public Customer[] getCustomers() {}
    }

If you can't annotate the JAXB class with @NoJackson, then you can use
the annotation on a method parameter. For example:

    @XmlRootElement
    public class Customer {...}

    @Path("/customers")
    public class MyService {

        @GET
        @Produces("application/vnd.customer+json")
        @NoJackson
        public Customer[] getCustomers() {}

        @POST
        @Consumes("application/vnd.customer+json")
        public void createCustomer(@NoJackson Customer[] customers) {...}
    }

Possible Conflict With JAXB Provider {#Possible_Jackson_Problems}
====================================

If your Jackson classes are annotated with JAXB annotations and you have
the resteasy-jaxb-provider in your classpath, you may trigger the
Jettision JAXB marshalling code. To turn off the JAXB json marshaller
use the
@org.jboss.resteasy.annotations.providers.jaxb.IgnoreMediaTypes("application/\*+json")
on your classes.

JSONP Support {#JSONP_Support}
=============

If you're using Jackson, Resteasy has
[JSONP](http://en.wikipedia.org/wiki/JSONP) that you can turn on by
adding the provider
`org.jboss.resteasy.plugins.providers.jackson.JacksonJsonpInterceptor`
(Jackson2JsonpInterceptor if you're using the Jackson2 provider) to your
deployments. If the media type of the response is json and a callback
query parameter is given, the response will be a javascript snippet with
a method call of the method defined by the callback parameter. For
example:

    GET /resources/stuff?callback=processStuffResponse

will produce this response:

    processStuffResponse(<nomal JSON body>)

This supports the default behavior of
[jQuery](http://api.jquery.com/jQuery.ajax/). To enable
JacksonJsonpInterceptor in WildFly, you need to import annotations from
`org.jboss.resteasy.resteasy-jackson-provider` module using
jboss-deployment-structure.xml:

    <jboss-deployment-structure>
      <deployment>
        <dependencies>
          <module name="org.jboss.resteasy.resteasy-jackson-provider" annotations="true"/>
        </dependencies>
      </deployment>
    </jboss-deployment-structure>

You can change the name of the callback parameter by setting the
callbackQueryParameter property.

JacksonJsonpInterceptor can wrap the response into a try-catch block:

    try{processStuffResponse(<normal JSON body>)}catch(e){}

You can enable this feature by setting the resteasy.jsonp.silent
property to true

Jackson JSON Decorator {#Jackson_JSON_Decorator}
======================

If you are using the Jackson 2.2.x provider, Resteasy has provided a
pretty-printing annotation simliar with the one in JAXB provider:

    org.jboss.resteasy.annotations.providers.jackson.Formatted

Here is an example:

    @GET
    @Produces("application/json")
    @Path("/formatted/{id}")
    @Formatted
    public Product getFormattedProduct()
    {
        return new Product(333, "robot");
    }

As the example shown above, the @Formatted annotation will enable the
underlying Jackson option "SerializationFeature.INDENT\_OUTPUT".

Jackson JSONFilter {#Jackson_JSONFilter}
==================

If you are using the Jackson 2.2.x provider, Resteasy has provided a
pretty-printing annotation simliar with the one in JAXB provider:

    org.jboss.resteasy.annotations.providers.jackson.Formatted

Here is an example:

    @GET
    @Produces("application/json")
    @Path("/formatted/{id}")
    @Formatted
    public Product getFormattedProduct()
    {
        return new Product(333, "robot");
    }

As the example shown above, the @Formatted annotation will enable the
underlying Jackson option "SerializationFeature.INDENT\_OUTPUT".

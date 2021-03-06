JAX-RS Content Negotiation {#JAX-RS_Content_Negotiation}
==========================

The HTTP protocol has built in content negotiation headers that allow
the client and server to specify what content they are transferring and
what content they would prefer to get. The server declares content
preferences via the @Produces and @Consumes headers.

@Consumes is an array of media types that a particular resource or
resource method consumes. For example:

    @Consumes("text/*")
    @Path("/library")
    public class Library {

        @POST
        public String stringBook(String book) {...}

        @Consumes("text/xml")
        @POST
        public String jaxbBook(Book book) {...}
    }

When a client makes a request, JAX-RS first finds all methods that match
the path, then, it sorts things based on the content-type header sent by
the client. So, if a client sent:

    POST /library
    Content-Type: text/plain

    This is a nice book

The stringBook() method would be invoked because it matches to the
default "text/\*" media type. Now, if the client instead sends XML:

    POST /library
    Content-Type: text/xml

    <book name="EJB 3.0" author="Bill Burke"/>

The jaxbBook() method would be invoked.

The @Produces is used to map a client request and match it up to the
client's Accept header. The Accept HTTP header is sent by the client and
defines the media types the client prefers to receive from the server.

    @Produces("text/*")
    @Path("/library")
    public class Library {

    @GET
    @Produces("application/json")
    public String getJSON() {...}

    @GET
    public String get() {...}

So, if the client sends:

    GET /library
    Accept: application/json

The getJSON() method would be invoked.

@Consumes and @Produces can list multiple media types that they support.
The client's Accept header can also send multiple types it might like to
receive. More specific media types are chosen first. The client Accept
header or @Produces @Consumes can also specify weighted preferences that
are used to match up requests with resource methods. This is best
explained by RFC 2616 section 14.1 . Resteasy supports this complex way
of doing content negotiation.

A variant in JAX-RS is a combination of media type, content-language,
and content encoding as well as etags, last modified headers, and other
preconditions. This is a more complex form of content negotiation that
is done programmatically by the application developer using the
javax.ws.rs.Variant, VarianListBuilder, and Request objects. Request is
injected via @Context. Read the javadoc for more info on these.

URL-based negotiation {#media_mappings}
=====================

Some clients, like browsers, cannot use the Accept and Accept-Language
headers to negotiation the representation's media type or language.
RESTEasy allows you to map file name suffixes like (.xml, .txt, .en,
.fr) to media types and languages. These file name suffixes take the
place and override any Accept header sent by the client. You configure
this using the resteasy.media.type.mappings and
resteasy.language.mappings context-param variables within your web.xml.

    <web-app>
        <display-name>Archetype Created Web Application</display-name>
        <context-param>
            <param-name>resteasy.media.type.mappings</param-name>
            <param-value>html : text/html, json : application/json, xml : application/xml</param-value>
        </context-param>

       <context-param>
            <param-name>resteasy.language.mappings</param-name>
            <param-value>en : en-US, es : es, fr : fr</param-value>
       </context-param>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher</servlet-class>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>
    </web-app>

Mappings are a comma delimited list of suffix/mediatype or
suffix/language mappings. Each mapping is delimited by a ':'. So, if you
invoked GET /foo/bar.xml.en, this would be equivalent to invoking the
following request:

    GET /foo/bar
    Accept: application/xml
    Accept-Language: en-US

The mapped file suffixes are stripped from the target URL path before
the request is dispatched to a corresponding JAX-RS resource.

Query String Parameter-based negotiation {#param_media_mappings}
========================================

RESTEasy can do content negotiation based in a parameter in query
string. To enable this, the web.xml can be configured like follow:

    <web-app>
        <display-name>Archetype Created Web Application</display-name>
        <context-param>
            <param-name>resteasy.media.type.param.mapping</param-name>
            <param-value>someName</param-value>
        </context-param>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher</servlet-class>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>
    </web-app>

The param-value is the name of the query string parameter that RESTEasy
will use in the place of the Accept header.

Invoking http://service.foo.com/resouce?someName=application/xml, will
give the application/xml media type the highest priority in the content
negotiation.

In cases where the request contains both the parameter and the Accept
header, the parameter will be more relevant.

It is possible to left the param-value empty, what will cause the
processor to look for a parameter named 'accept'.

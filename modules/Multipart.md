Multipart Providers {#Multipart}
===================

Resteasy has rich support for the "multipart/\*" and
"multipart/form-data" mime types. The multipart mime format is used to
pass lists of content bodies. Multiple content bodies are embedded in
one message. "multipart/form-data" is often found in web application
HTML Form documents and is generally used to upload files. The form-data
format is the same as other multipart formats, except that each inlined
piece of content has a name associated with it.

RESTEasy provides a custom API for reading and writing multipart types
as well as marshalling arbitrary List (for any multipart type) and Map
(multipart/form-data only) objects

Input with multipart/mixed {#MultipartInput}
==========================

When writing a JAX-RS service, RESTEasy provides an interface that
allows you to read in any multipart mime type.
org.jboss.resteasy.plugins.providers.multipart.MultipartInput

    package org.jboss.resteasy.plugins.providers.multipart;

    public interface MultipartInput
    {
       List<InputPart> getParts();

       String getPreamble();

       // You must call close to delete any temporary files created
       // Otherwise they will be deleted on garbage collection or on JVM exit
       void close();
    }

    public interface InputPart
    {
       MultivaluedMap<String, String> getHeaders();

       String getBodyAsString();

       <T> T getBody(Class<T> type, Type genericType) throws IOException;

       <T> T getBody(org.jboss.resteasy.util.GenericType<T> type) throws IOException;

       MediaType getMediaType();

       boolean isContentTypeFromMessage();
    }

MultipartInput is a simple interface that allows you to get access to
each part of the multipart message. Each part is represented by an
InputPart interface. Each part has a set of headers associated with it
You can unmarshall the part by calling one of the getBody() methods. The
Type genericType parameter can be null, but the Class type parameter
must be set. Resteasy will find a MessageBodyReader based on the media
type of the part as well as the type information you pass in. The
following piece of code is unmarshalling parts which are XML into a JAXB
annotated class called Customer.

    @Path("/multipart")
    public class MyService
    {
        @PUT
        @Consumes("multipart/mixed")
        public void put(MultipartInput input)
        {
            List<Customer> customers = new ArrayList...;
            for (InputPart part : input.getParts())
            {
                Customer cust = part.getBody(Customer.class, null);
                customers.add(cust);
            }
            input.close();
        }
    }

Sometimes you may want to unmarshall a body part that is sensitive to
generic type metadata. In this case you can use the
org.jboss.resteasy.util.GenericType class. Here's an example of
unmarshalling a type that is sensitive to generic type metadata.

    @Path("/multipart")
    public class MyService
    {
        @PUT
        @Consumes("multipart/mixed")
        public void put(MultipartInput input)
        {
            for (InputPart part : input.getParts())
            {
                List<Customer> cust = part.getBody(new GenericType>List>Customer<<() {});
            }
            input.close();
        }
    }

Use of GenericType is required because it is really the only way to
obtain generic type information at runtime.

java.util.List with multipart data {#multipart_list}
==================================

If your body parts are uniform, you do not have to manually unmarshall
each and every part. You can just provide a java.util.List as your input
parameter. It must have the type it is unmarshalling with the generic
parameter of the List type declaration. Here's an example again of
unmarshalling a list of customers.

    @Path("/multipart")
    public class MyService
    {
        @PUT
        @Consumes("multipart/mixed")
        public void put(List<Customer> customers)
        {
            ...
        }
    }

Input with multipart/form-data {#MultipartFormData}
==============================

When writing a JAX-RS service, RESTEasy provides an interface that
allows you to read in multipart/form-data mime type.
"multipart/form-data" is often found in web application HTML Form
documents and is generally used to upload files. The form-data format is
the same as other multipart formats, except that each inlined piece of
content has a name associated with it. The interface used for form-data
input is
org.jboss.resteasy.plugins.providers.multipart.MultipartFormDataInput

    public interface MultipartFormDataInput extends MultipartInput
    {
        @Deprecated
        Map<String, InputPart> getFormData();

        Map<String, List<InputPart>> getFormDataMap();

        <T> T getFormDataPart(String key, Class<T> rawType, Type genericType) throws IOException;

        <T> T getFormDataPart(String key, GenericType<T> type) throws IOException;
    }

It works in much the same way as MultipartInput described earlier in
this chapter.

java.util.Map with multipart/form-data {#multipart_map}
======================================

With form-data, if your body parts are uniform, you do not have to
manually unmarshall each and every part. You can just provide a
java.util.Map as your input parameter. It must have the type it is
unmarshalling with the generic parameter of the List type declaration.
Here's an example of of unmarshalling a Map of Customer objects which
are JAXB annotated classes.

    @Path("/multipart")
    public class MyService
    {
        @PUT
        @Consumes("multipart/form-data")
        public void put(Map<String, Customer> customers)
        {
            ...
        }
    }

Input with multipart/related {#MultipartRelated}
============================

When writing a JAX-RS service, RESTEasy provides an interface that
allows you to read in multipart/related mime type. A multipart/related
is used to indicate that message parts should not be considered
individually but rather as parts of an aggregate whole. One example
usage for multipart/related is to send a web page complete with images
in a single message. Every multipart/related message has a root/start
part that references the other parts of the message. The parts are
identified by their "Content-ID" headers. multipart/related is defined
by RFC 2387. The interface used for related input is
org.jboss.resteasy.plugins.providers.multipart.MultipartRelatedInput

    public interface MultipartRelatedInput extends MultipartInput
    {
        String getType();

        String getStart();

        String getStartInfo();

        InputPart getRootPart();

        Map<String, InputPart> getRelatedMap();
    }

It works in much the same way as MultipartInput described earlier in
this chapter.

Output with multipart {#multipart_output}
=====================

RESTEasy provides a simple API to output multipart data.

    package org.jboss.resteasy.plugins.providers.multipart;

    public class MultipartOutput
    {
        public OutputPart addPart(Object entity, MediaType mediaType)

        public OutputPart addPart(Object entity, GenericType type, MediaType mediaType)

        public OutputPart addPart(Object entity, Class type, Type genericType, MediaType mediaType)

        public List<OutputPart> getParts()

        public String getBoundary()

        public void setBoundary(String boundary)
    }

    public class OutputPart
    {
        public MultivaluedMap<String, Object> getHeaders()

        public Object getEntity()

        public Class getType()

        public Type getGenericType()

        public MediaType getMediaType()
    }

When you want to output multipart data it is as simple as creating a
MultipartOutput object and calling addPart() methods. Resteasy will
automatically find a MessageBodyWriter to marshall your entity objects.
Like MultipartInput, sometimes you may have marshalling which is
sensitive to generic type metadata. In that case, use GenericType. Most
of the time though passing in an Object and its MediaType is enough. In
the example below, we are sending back a "multipart/mixed" format back
to the calling client. The parts are Customer objects which are JAXB
annotated and will be marshalling into "application/xml".

    @Path("/multipart")
    public class MyService
    {
        @GET
        @Produces("multipart/mixed")
        public MultipartOutput get()
        {
            MultipartOutput output = new MultipartOutput();
            output.addPart(new Customer("bill"), MediaType.APPLICATION_XML_TYPE);
            output.addPart(new Customer("monica"), MediaType.APPLICATION_XML_TYPE);
            return output;
        }
    }

Multipart Output with java.util.List {#multipart_list_output}
====================================

If your body parts are uniform, you do not have to manually marshall
each and every part or even use a MultipartOutput object.. You can just
provide a java.util.List. It must have the generic type it is
marshalling with the generic parameter of the List type declaration. You
must also annotate the method with the @PartType annotation to specify
what media type each part is. Here's an example of sending back a list
of customers back to a client. The customers are JAXB objects

    @Path("/multipart")
    public class MyService
    {
        @GET
        @Produces("multipart/mixed")
        @PartType("application/xml")
        public List<Customer> get()
        {
            ...
        }
    }

Output with multipart/form-data {#multipart_formdata_output}
===============================

RESTEasy provides a simple API to output multipart/form-data.

    package org.jboss.resteasy.plugins.providers.multipart;

    public class MultipartFormDataOutput extends MultipartOutput
    {
        public OutputPart addFormData(String key, Object entity, MediaType mediaType)

        public OutputPart addFormData(String key, Object entity, GenericType type, MediaType mediaType)

        public OutputPart addFormData(String key, Object entity, Class type, Type genericType, MediaType mediaType)

        public Map<String, OutputPart> getFormData()
    }

When you want to output multipart/form-data it is as simple as creating
a MultipartFormDataOutput object and calling addFormData() methods.
Resteasy will automatically find a MessageBodyWriter to marshall your
entity objects. Like MultipartInput, sometimes you may have marshalling
which is sensitive to generic type metadata. In that case, use
GenericType. Most of the time though passing in an Object and its
MediaType is enough. In the example below, we are sending back a
"multipart/form-data" format back to the calling client. The parts are
Customer objects which are JAXB annotated and will be marshalling into
"application/xml".

    @Path("/form")
    public class MyService
    {
        @GET
        @Produces("multipart/form-data")
        public MultipartFormDataOutput get()
        {
            MultipartFormDataOutput output = new MultipartFormDataOutput();
            output.addPart("bill", new Customer("bill"), MediaType.APPLICATION_XML_TYPE);
            output.addPart("monica", new Customer("monica"), MediaType.APPLICATION_XML_TYPE);
            return output;
        }
    }

Multipart FormData Output with java.util.Map {#multipart_map_output}
============================================

If your body parts are uniform, you do not have to manually marshall
each and every part or even use a MultipartFormDataOutput object.. You
can just provide a java.util.Map. It must have the generic type it is
marshalling with the generic parameter of the Map type declaration. You
must also annotate the method with the @PartType annotation to specify
what media type each part is. Here's an example of sending back a list
of customers back to a client. The customers are JAXB objects

    @Path("/multipart")
    public class MyService
    {
        @GET
        @Produces("multipart/form-data")
        @PartType("application/xml")
        public Map<String, Customer> get()
        {
            ...
        }
    }

Output with multipart/related {#multipart_related_output}
=============================

RESTEasy provides a simple API to output multipart/related.

    package org.jboss.resteasy.plugins.providers.multipart;

    public class MultipartRelatedOutput extends MultipartOutput
    {
        public OutputPart getRootPart()

        public OutputPart addPart(Object entity, MediaType mediaType,
            String contentId, String contentTransferEncoding)

        public String getStartInfo()

        public void setStartInfo(String startInfo)
    }

When you want to output multipart/related it is as simple as creating a
MultipartRelatedOutput object and calling addPart() methods. The first
added part will be used as the root part of the multipart/related
message. Resteasy will automatically find a MessageBodyWriter to
marshall your entity objects. Like MultipartInput, sometimes you may
have marshalling which is sensitive to generic type metadata. In that
case, use GenericType. Most of the time though passing in an Object and
its MediaType is enough. In the example below, we are sending back a
"multipart/related" format back to the calling client. We are sending a
html with 2 images.

    @Path("/related")
    public class MyService
    {
        @GET
        @Produces("multipart/related")
        public MultipartRelatedOutput get()
        {
            MultipartRelatedOutput output = new MultipartRelatedOutput();
            output.setStartInfo("text/html");

            Map<String, String> mediaTypeParameters = new LinkedHashMap<String, String>();
            mediaTypeParameters.put("charset", "UTF-8");
            mediaTypeParameters.put("type", "text/html");
            output.addPart(
                "<html><body>\n"
                + "This is me: <img src='cid:http://example.org/me.png' />\n"
                + "<br />This is you: <img src='cid:http://example.org/you.png' />\n"
                + "</body></html>",
                new MediaType("text", "html", mediaTypeParameters),
                "<mymessage.xml@example.org>", "8bit");
            output.addPart("// binary octets for me png",
                new MediaType("image", "png"), "<http://example.org/me.png>",
                "binary");
            output.addPart("// binary octets for you png", new MediaType(
                "image", "png"),
                "<http://example.org/you.png>", "binary");
            client.putRelated(output);
            return output;
        }
    }

@MultipartForm and POJOs {#multipartform_annotation}
========================

If you have a exact knowledge of your multipart/form-data packets, you
can map them to and from a POJO class to and from multipart/form-data
using the
@org.jboss.resteasy.annotations.providers.multipart.MultipartForm
annotation and the JAX-RS @FormParam annotation. You simple define a
POJO with at least a default constructor and annotate its fields and/or
properties with @FormParams. These @FormParams must also be annotated
with @org.jboss.resteasy.annotations.providers.multipart.PartType if you
are doing output. For example:

    public class CustomerProblemForm {
        @FormParam("customer")
        @PartType("application/xml")
        private Customer customer;

        @FormParam("problem")
        @PartType("text/plain")
        private String problem;

        public Customer getCustomer() { return customer; }
        public void setCustomer(Customer cust) { this.customer = cust; }
        public String getProblem() { return problem; }
        public void setProblem(String problem) { this.problem = problem; }
    }

After defining your POJO class you can then use it to represent
multipart/form-data. Here's an example of sending a CustomerProblemForm
using the RESTEasy client framework:

    @Path("portal")
    public interface CustomerPortal {

        @Path("issues/{id}")
        @Consumes("multipart/form-data")
        @PUT
        public void putProblem(@MultipartForm CustomerProblemForm, 
                               @PathParam("id") int id);
    }

    {
        CustomerPortal portal = ProxyFactory.create(CustomerPortal.class, "http://example.com");
        CustomerProblemForm form = new CustomerProblemForm();
        form.setCustomer(...);
        form.setProblem(...);

        portal.putProblem(form, 333);
    }

You see that the @MultipartForm annotation was used to tell RESTEasy
that the object has @FormParam and that it should be marshalled from
that. You can also use the same object to receive multipart data. Here
is an example of the server side counterpart of our customer portal.

    @Path("portal")
    public class CustomerPortalServer {

        @Path("issues/{id})
        @Consumes("multipart/form-data")
        @PUT
        public void putIssue(@MultipartForm CustoemrProblemForm,
                             @PathParam("id") int id) {
           ... write to database...
        }
    }

XML-binary Optimized Packaging (Xop) {#xop_with_multipart_related}
====================================

RESTEasy supports Xop messages packaged as multipart/related. What does
this mean? If you have a JAXB annotated POJO that also holds some binary
content you may choose to send it in such a way where the binary does
not need to be encoded in any way (neither base64 neither hex). This
results in faster transport while still using the convenient POJO. More
about Xop can be read here:
[http://www.w3.org/TR/xop10/](#http://www.w3.org/TR/xop10/). Now lets
see an example:

First we have a JAXB annotated POJO to work with. @XmlMimeType tells
JAXB the mime type of the binary content (its not required to do XOP
packaging but it is recommended to be set if you know the exact type):

    @XmlRootElement
    @XmlAccessorType(XmlAccessType.FIELD)
    public static class Xop {
        
        private Customer bill;
        private Customer monica;

        @XmlMimeType(MediaType.APPLICATION_OCTET_STREAM)
        private byte[] myBinary;

        @XmlMimeType(MediaType.APPLICATION_OCTET_STREAM)
        private DataHandler myDataHandler;

        // methods, other fields ...
    }

In the above POJO myBinary and myDataHandler will be processed as binary
attachments while the whole Xop object will be sent as xml (in the
places of the binaries only their references will be generated).
javax.activation.DataHandler is the most general supported type so if
you need an java.io.InputStream or a javax.activation.DataSource you
need to go with the DataHandler. Some other special types are supported
too: java.awt.Image and javax.xml.transform.Source. Let's assume that
Customer is also JAXB friendly POJO in the above example (of course it
can also have binary parts). Now lets see a an example Java client that
sends this:

    // our client interface:
    @Path("mime")
    public static interface MultipartClient {
        @Path("xop")
        @PUT
        @Consumes(MultipartConstants.MULTIPART_RELATED)
        public void putXop(@XopWithMultipartRelated Xop bean);
    }

    // Somewhere using it:
    {
        MultipartClient client = ProxyFactory.create(MultipartClient.class,
            "http://www.example.org");
        Xop xop = new Xop(new Customer("bill"), new Customer("monica"),
            "Hello Xop World!".getBytes("UTF-8"),
            new DataHandler(new ByteArrayDataSource("Hello Xop World!".getBytes("UTF-8"),
            MediaType.APPLICATION_OCTET_STREAM)));
        client.putXop(xop);
    }

We used @Consumes(MultipartConstants.MULTIPART\_RELATED) to tell
RESTEasy that we want to send multipart/related packages (that's the
container format that will hold our Xop message). We used
@XopWithMultipartRelated to tell RESTEasy that we want to make Xop
messages. So we have a POJO and a client service that is willing to send
it. All we need now a server that can read it:

    @Path("/mime")
    public class XopService {
        @PUT
        @Path("xop")
        @Consumes(MultipartConstants.MULTIPART_RELATED)
        public void putXopWithMultipartRelated(@XopWithMultipartRelated Xop xop) {
            // do very important things here
        }
    }

We used @Consumes(MultipartConstants.MULTIPART\_RELATED) to tell
RESTEasy that we want to read multipart/related packages. We used
@XopWithMultipartRelated to tell RESTEasy that we want to read Xop
messages. Of course we could also produce Xop return values but we would
than also need to annotate that and use a Produce annotation, too.

Note about multipart parsing and working with other frameworks {#multipart_parsing_note}
==============================================================

There are a lot of frameworks doing multipart parsing automatically with
the help of filters and interceptors. Like
org.jboss.seam.web.MultipartFilter in Seam or
org.springframework.web.multipart.MultipartResolver in Spring. However
the incoming multipart request stream can be parsed only once. Resteasy
users working with multipart should make sure that nothing parses the
stream before Resteasy gets it.

Overwriting the default fallback content type for multipart messages {#multipart_overwrite_default_content_type}
====================================================================

By default if no Content-Type header is present in a part, "text/plain;
charset=us-ascii" is used as fallback. This is the value defined by the
MIME RFC. However for example some web clients (like most, if not all,
web browsers) do not send Content-Type headers for all fields in a
multipart/form-data request (only for the file parts). This can cause
character encoding and unmarshalling errors on the server side. To
correct this there is an option to define an other, non-rfc compliant
fallback value. This can be done dynamically per request with the
PreProcessInterceptor infrastructure of RESTEasy. In the following
example we will set "\*/\*; charset=UTF-8" as the new default fallback:

    import org.jboss.resteasy.plugins.providers.multipart.InputPart;

    @Provider
    @ServerInterceptor
    public class ContentTypeSetterPreProcessorInterceptor implements PreProcessInterceptor {

        public ServerResponse preProcess(HttpRequest request, ResourceMethod method)
                throws Failure, WebApplicationException {
            request.setAttribute(InputPart.DEFAULT_CONTENT_TYPE_PROPERTY, "*/*; charset=UTF-8");
            return null;
        }
    }

Overwriting the content type for multipart messages {#multipart_overwrite_content_type}
===================================================

Using an interceptor and the `InputPart.DEFAULT_CONTENT_TYPE_PROPERTY`
attribute allows setting a default Content-Type, but it is also possible
to override the Content-Type, if any, in any input part by calling
org.jboss.resteasy.plugins.providers.multipart.InputPart.setMediaType().
For example:

    @POST
    @Path("query")
    @Consumes(MediaType.MULTIPART_FORM_DATA)
    @Produces(MediaType.TEXT_PLAIN)
    public Response setMediaType(MultipartInput input) throws IOException
    {
        List<InputPart> parts = input.getParts();
        InputPart part = parts.get(0);
        part.setMediaType(MediaType.valueOf("application/foo+xml"));
        String s = part.getBody(String.class, null);
        ...
    }

Overwriting the default fallback charset for multipart messages {#multipart_overwrite_default_charset}
===============================================================

Sometimes, a part may have a Content-Type header with no charset
parameter. If the `InputPart.DEFAULT_CONTENT_TYPE_PROPERTY` property is
set and the value has a charset parameter, that value will be appended
to an existing Content-Type header that has no charset parameter. It is
also possible to specify a default charset using the constant
`InputPart.DEFAULT_CHARSET_PROPERTY` (actual value
"resteasy.provider.multipart.inputpart.defaultCharset"):

    import org.jboss.resteasy.plugins.providers.multipart.InputPart;

    @Provider
    @ServerInterceptor
    public class ContentTypeSetterPreProcessorInterceptor implements PreProcessInterceptor {

        public ServerResponse preProcess(HttpRequest request, ResourceMethod method)
                throws Failure, WebApplicationException {
            request.setAttribute(InputPart.DEFAULT_CHARSET_PROPERTY, "UTF-8");
            return null;
        }
    }

If both `InputPart.DEFAULT_CONTENT_TYPE_PROPERTY` and
`InputPart.DEFAULT_CHARSET_PROPERTY` are set, then the value of
`InputPart.DEFAULT_CHARSET_PROPERTY` will override any charset in the
value of `InputPart.DEFAULT_CONTENT_TYPE_PROPERTY`.

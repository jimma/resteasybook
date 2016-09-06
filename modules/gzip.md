GZIP Compression/Decompression {#gzip}
==============================

Resteasy has automatic GZIP decompression support. If the client
framework or a JAX-RS service receives a message body with a
Content-Encoding of "gzip", it will automatically decompress it. The
client framework automatically sets the Accept-Encoding header to be
"gzip, deflate". So you do not have to set this header yourself.

Resteasy also supports automatic compression. If the client framework is
sending a request or the server is sending a response with the
Content-Encoding header set to "gzip", Resteasy will do the compression.
So that you do not have to set the Content-Encoding header directly, you
can use the @org.jboss.resteasy.annotation.GZIP annotation.

    @Path("/")
    public interface MyProxy {

       @Consumes("application/xml")
       @PUT
       public void put(@GZIP Order order);
    }

In the above example, we tag the outgoing message body, order, to be
gzip compressed. You can use the same annotation to tag server responses

    @Path("/")
    public class MyService {

       @GET
       @Produces("application/xml")
       @GZIP
       public String getData() {...}
    }

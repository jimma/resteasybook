Asynchronous HTTP Request Processing {#Asynchronous_HTTP_Request_Processing}
====================================

Asynchronous HTTP Request Processing is a relatively new technique that
allows you to process a single HTTP request using non-blocking I/O and,
if desired in separate threads. Some refer to it as COMET capabilities.
The primary use case for Asynchronous HTTP is in the case where the
client is polling the server for a delayed response. The usual example
is an AJAX chat client where you want to push/pull from both the client
and the server. These scenarios have the client blocking a long time on
the server’s socket waiting for a new message. What happens in
synchronous HTTP where the server is blocking on incoming and outgoing
I/O is that you end up having a thread consumed per client connection.
This eats up memory and valuable thread resources. Not such a big deal
in 90% of applications (in fact using asynchronous processing may
actually hurt your performance in most common scenarios), but when you
start getting a lot of concurrent clients that are blocking like this,
there’s a lot of wasted resources and your server does not scale that
well.

The JAX-RS 2.0 specification has added asynchronous HTTP support via two
classes. The @Suspended annotation, and AsyncResponse interface.

Injecting an AsynchronousResponse as a parameter to your jax-rs methods
tells Resteasy that the HTTP request/response should be detached from
the currently executing thread and that the current thread should not
try to automatically process the response.

The AsyncResponse is the callback object. The act of calling one of the
resume() methods will cause a response to be sent back to the client and
will also terminate the HTTP request. Here is an example of asynchronous
processing:

    import javax.ws.rs.Suspend;
    import javax.ws.rs.core.AsynchronousResponse;

    @Path("/")
    public class SimpleResource
    {

       @GET
       @Path("basic")
       @Produces("text/plain")
       public void getBasic(@Suspended final AsyncResponse response) throws Exception
       {
          Thread t = new Thread()
          {
             @Override
             public void run()
             {
                try
                {
                   Response jaxrs = Response.ok("basic").type(MediaType.TEXT_PLAIN).build();
                   response.resume(jaxrs);
                }
                catch (Exception e)
                {
                   e.printStackTrace();
                }
             }
          };
          t.start();
       }
    }
       

AsyncResponse also has other methods to cancel the execution. See
javadoc for more details.

**NOTE:** The old Resteasy proprietary API for async http has been
deprecated and may be removed as soon as Resteasy 3.1. In particular,
the Resteasy @Suspend annotation is replaced by
`javax.ws.rs.container.Suspended`, and
`org.jboss.resteasy.spi.AsynchronousResponse` is replaced by
`javax.ws.rs.container.AsyncResponse`. Note that @Suspended does not
have a value field, which represented a timeout limit. Instead,
AsyncResponse.setTimeout() may be called.
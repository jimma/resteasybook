@HeaderParam {#_HeaderParam}
============

The @HeaderParam annotation allows you to map a request HTTP header to
your method invocation.

GET /books?num=5

    @GET
    public String getBooks(@HeaderParam("From") String from) {
    ...
    }

Like PathParam, your parameter type can be an String, primitive, or
class that has a String constructor or static valueOf() method. For
example, MediaType has a valueOf() method and you could do:

    @PUT
    public void put(@HeaderParam("Content-Type") MediaType contentType, ...)

Using @Path and @GET, @POST, etc. {#Using_Path}
=================================

    @Path("/library")
    public class Library {

       @GET
       @Path("/books")
       public String getBooks() {...}

       @GET
       @Path("/book/{isbn}")
       public String getBook(@PathParam("isbn") String id) {
          // search my database and get a string representation and return it
       }

       @PUT
       @Path("/book/{isbn}")
       public void addBook(@PathParam("isbn") String id, @QueryParam("name") String name) {...}

       @DELETE
       @Path("/book/{id}")
       public void removeBook(@PathParam("id") String id {...}
       
    }

Let's say you have the Resteasy servlet configured and reachable at a
root path of http://myhost.com/services. The requests would be handled
by the Library class:

-   GET http://myhost.com/services/library/books
-   GET http://myhost.com/services/library/book/333
-   PUT http://myhost.com/services/library/book/333
-   DELETE http://myhost.com/services/library/book/333

The @javax.ws.rs.Path annotation must exist on either the class and/or a
resource method. If it exists on both the class and method, the relative
path to the resource method is a concatenation of the class and method.

In the @javax.ws.rs package there are annotations for each HTTP method.
@GET, @POST, @PUT, @DELETE, and @HEAD. You place these on public methods
that you want to map to that certain kind of HTTP method. As long as
there is a @Path annotation on the class, you do not have to have a
@Path annotation on the method you are mapping. You can have more than
one HTTP method as long as they can be distinguished from other methods.

When you have a @Path annotation on a method without an HTTP method,
these are called JAXRSResourceLocators.

@Path and regular expression mappings {#_Path_and_regular_expression_mappings}
=====================================

The @Path annotation is not limited to simple path expressions. You also
have the ability to insert regular expressions into @Path's value. For
example:

    @Path("/resources)
    public class MyResource {

       @GET
       @Path("{var:.*}/stuff")
       public String get() {...}
    }

The following GETs will route to the getResource() method:

    GET /resources/stuff
    GET /resources/foo/stuff
    GET /resources/on/and/on/stuff

The format of the expression is:

    "{" variable-name [ ":" regular-expression ] "}"

The regular-expression part is optional. When the expression is not
provided, it defaults to a wildcard matching of one particular segment.
In regular-expression terms, the expression defaults to

    "([]*)"

For example:

@Path("/resources/{var}/stuff")

will match these:

    GET /resources/foo/stuff
    GET /resources/bar/stuff

but will not match:

    GET /resources/a/bunch/of/stuff

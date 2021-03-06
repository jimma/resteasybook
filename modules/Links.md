Linking resources {#Linkresource}
=================

There are two mechanisms available in RESTEasy to link a resource to
another, and to link resources to operations: the Link HTTP header, and
Atom links inside the resource representations.

Link Headers {#LinkHeader}
============

RESTEasy has both client and server side support for the [Link header
specification](http://tools.ietf.org/html/draft-nottingham-http-link-header-06).
See the javadocs for org.jboss.resteasy.spi.LinkHeader,
org.jboss.resteasy.spi.Link, and
org.jboss.resteasy.client.ClientResponse.

The main advantage of Link headers over Atom links in the resource is
that those links are available without parsing the entity body.

Atom links in the resource representations {#AtomLinks}
==========================================

RESTEasy allows you to inject [Atom
links](http://tools.ietf.org/html/rfc4287#section-4.2.7) directly inside
the entity objects you are sending to the client, via auto-discovery.

> **Warning**
>
> This is only available when using the Jettison or JAXB providers (for
> JSON and XML).

The main advantage over Link headers is that you can have any number of
Atom links directly over the concerned resources, for any number of
resources in the response. For example, you can have Atom links for the
root response entity, and also for each of its children entities.

Configuration {#configuration}
-------------

There is no configuration required to be able to inject Atom links in
your resource representation, you just have to have this maven artifact
in your path:

  Group                Artifact         Version
  -------------------- ---------------- ----------------
  org.jboss.resteasy   resteasy-links   3.1.0-SNAPSHOT

  : Maven artifact for Atom link injection

Your first links injected {#firstlinks}
-------------------------

You need three things in order to tell RESTEasy to inject Atom links in
your entities:

-   Annotate the JAX-RS method with `@AddLinks` to indicate that you
    want Atom links injected in your response entity.

-   Add `RESTServiceDiscovery` fields to the resource classes where you
    want Atom links injected.

-   Annotate the JAX-RS methods you want Atom links for with
    `@LinkResource`, so that RESTEasy knows which links to create for
    which resources.

The following example illustrates how you would declare everything in
order to get the Atom links injected in your book store:

``` {.Java}
@Path("/")
@Consumes({"application/xml", "application/json"})
@Produces({"application/xml", "application/json"})
public interface BookStore {

    @AddLinks
    @LinkResource(value = Book.class)
    @GET
    @Path("books")
    public Collection<Book> getBooks();

    @LinkResource
    @POST
    @Path("books")
    public void addBook(Book book);

    @AddLinks
    @LinkResource
    @GET
    @Path("book/{id}")
    public Book getBook(@PathParam("id") String id);

    @LinkResource
    @PUT
    @Path("book/{id}")
    public void updateBook(@PathParam("id") String id, Book book);

    @LinkResource(value = Book.class)
    @DELETE
    @Path("book/{id}")
    public void deleteBook(@PathParam("id") String id);
}
```

And this is the definition of the Book resource:

``` {.Java}
@Mapped(namespaceMap = @XmlNsMap(jsonName = "atom", namespace = "http://www.w3.org/2005/Atom"))
@XmlRootElement
@XmlAccessorType(XmlAccessType.NONE)
public class Book {
    @XmlAttribute
    private String author;

    @XmlID
    @XmlAttribute
    private String title;

    @XmlElementRef
    private RESTServiceDiscovery rest;
}
```

If you do a GET /order/foo you will then get this XML representation:

``` {.XML}
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book xmlns:atom="http://www.w3.org/2005/Atom" title="foo" author="bar">
 <atom:link href="http://localhost:8081/books" rel="list"/>
 <atom:link href="http://localhost:8081/books" rel="add"/>
 <atom:link href="http://localhost:8081/book/foo" rel="self"/>
 <atom:link href="http://localhost:8081/book/foo" rel="update"/>
 <atom:link href="http://localhost:8081/book/foo" rel="remove"/>
</book>
```

And in JSON format:

``` {.JavaScript}
{
 "book":
 {
  "@title":"foo",
  "@author":"bar",
  "atom.link":
   [
    {"@href":"http://localhost:8081/books","@rel":"list"},
    {"@href":"http://localhost:8081/books","@rel":"add"},
    {"@href":"http://localhost:8081/book/foo","@rel":"self"},
    {"@href":"http://localhost:8081/book/foo","@rel":"update"},
    {"@href":"http://localhost:8081/book/foo","@rel":"remove"}
   ]
 }
}
```

Customising how the Atom links are serialised {#serialised}
---------------------------------------------

Because the `RESTServiceDiscovery` is in fact a JAXB type which inherits
from `List` you are free to annotate it as you want to customise the
JAXB serialisation, or just rely on the default with `@XmlElementRef`.

Specifying which JAX-RS methods are tied to which resources {#specify-resources}
-----------------------------------------------------------

This is all done by annotating the methods with the `@LinkResource`
annotation. It supports the following optional parameters:

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Parameter   Type       Function                                                 Default
  ----------- ---------- -------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  value       `Class`    Declares an Atom link for the given type of resources.   Defaults to the entity body type (non-annotated parameter), or the method's return type. This default does not work with `Response` or `Collection` types, they need to be explicitly specified.

  rel         `String`   The Atom link relation                                   list
                                                                                  
                                                                                  :   For `GET` methods returning a `Collection`
                                                                                  
                                                                                  self
                                                                                  
                                                                                  :   For `GET` methods returning a non-`Collection`
                                                                                  
                                                                                  remove
                                                                                  
                                                                                  :   For `DELETE` methods
                                                                                  
                                                                                  update
                                                                                  
                                                                                  :   For `PUT` methods
                                                                                  
                                                                                  add
                                                                                  
                                                                                  :   For `POST` methods
                                                                                  
                                                                                  
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  : `@LinkResource` parameters

You can add several `@LinkResource` annotations on a single method by
enclosing them in a `@LinkResources` annotation. This way you can add
links to the same method on several resource types. For example the
`/order/foo/comments` operation can belongs on the `Order` resource with
the `comments` relation, and on the `Comment` resource with the `list`
relation.

Specifying path parameter values for URI templates {#uri}
--------------------------------------------------

When RESTEasy adds links to your resources it needs to insert the right
values in the URI template. This is done either automatically by
guessing the list of values from the entity, or by specifying the values
in the `@LinkResource` `pathParameters` parameter.

### Loading URI template values from the entity

URI template values are extracted from the entity from fields or Java
Bean properties annotated with `@ResourceID`, JAXB's `@XmlID` or JPA's
`@Id`. If there are more than one URI template value to find in a given
entity, you can annotate your entity with `@ResourceIDs` to list the
names of fields or properties that make up this entity's Id. If there
are other URI template values required from a parent entity, we try to
find that parent in a field or Java Bean property annotated with
`@ParentResource`. The list of URI template values extracted up every
`@ParentResource` is then reversed and used as the list of values for
the URI template.

For example, let's consider the previous Book example, and a list of
comments:

``` {.Java}
@XmlRootElement
@XmlAccessorType(XmlAccessType.NONE)
public class Comment {
    @ParentResource
    private Book book;

    @XmlElement
    private String author;

    @XmlID
    @XmlAttribute
    private String id;

    @XmlElementRef
    private RESTServiceDiscovery rest;
}
```

Given the previous book store service augmented with comments:

``` {.Java}
@Path("/")
@Consumes({"application/xml", "application/json"})
@Produces({"application/xml", "application/json"})
public interface BookStore {

    @AddLinks
    @LinkResources({
        @LinkResource(value = Book.class, rel = "comments"),
        @LinkResource(value = Comment.class)
    })
    @GET
    @Path("book/{id}/comments")
    public Collection<Comment> getComments(@PathParam("id") String bookId);

    @AddLinks
    @LinkResource
    @GET
    @Path("book/{id}/comment/{cid}")
    public Comment getComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

    @LinkResource
    @POST
    @Path("book/{id}/comments")
    public void addComment(@PathParam("id") String bookId, Comment comment);

    @LinkResource
    @PUT
    @Path("book/{id}/comment/{cid}")
    public void updateComment(@PathParam("id") String bookId, @PathParam("cid") String commentId, Comment comment);

    @LinkResource(Comment.class)
    @DELETE
    @Path("book/{id}/comment/{cid}")
    public void deleteComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

}
```

Whenever we need to make links for a `Book` entity, we look up the ID in
the `Book`'s `@XmlID` property. Whenever we make links for `Comment`
entities, we have a list of values taken from the `Comment`'s `@XmlID`
and its `@ParentResource`: the `Book` and its `@XmlID`.

For a `Comment` with `id` `"1"` on a `Book` with `title` `"foo"` we will
therefore get a list of URI template values of `{"foo", "1"}`, to be
replaced in the URI template, thus obtaining either
`"/book/foo/comments"` or `"/book/foo/comment/1"`.

### Specifying path parameters manually

If you do not want to annotate your entities with resource ID
annotations (`@ResourceID`, `@ResourceIDs`, `@XmlID` or `@Id`) and
`@ParentResource`, you can also specify the URI template values inside
the `@LinkResource` annotation, using Unified Expression Language
expressions:

  Parameter        Type         Function                                                                Default
  ---------------- ------------ ----------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------
  pathParameters   `String[]`   Declares a list of UEL expressions to obtain the URI template values.   Defaults to using `@ResourceID`, `@ResourceIDs`, `@XmlID` or `@Id` and `@ParentResource` annotations to extract the values from the model.

  : `@LinkResource` URI template parameter

The UEL expressions are evaluated in the context of the entity, which
means that any unqualified variable will be taken as a property for the
entity itself, with the special variable `this` bound to the entity
we're generating links for.

The previous example of `Comment` service could be declared as such:

``` {.Java}
@Path("/")
@Consumes({"application/xml", "application/json"})
@Produces({"application/xml", "application/json"})
public interface BookStore {

    @AddLinks
    @LinkResources({
        @LinkResource(value = Book.class, rel = "comments", pathParameters = "${title}"),
        @LinkResource(value = Comment.class, pathParameters = {"${book.title}", "${id}"})
    })
    @GET
    @Path("book/{id}/comments")
    public Collection<Comment> getComments(@PathParam("id") String bookId);

    @AddLinks
    @LinkResource(pathParameters = {"${book.title}", "${id}"})
    @GET
    @Path("book/{id}/comment/{cid}")
    public Comment getComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

    @LinkResource(pathParameters = {"${book.title}", "${id}"})
    @POST
    @Path("book/{id}/comments")
    public void addComment(@PathParam("id") String bookId, Comment comment);

    @LinkResource(pathParameters = {"${book.title}", "${id}"})
    @PUT
    @Path("book/{id}/comment/{cid}")
    public void updateComment(@PathParam("id") String bookId, @PathParam("cid") String commentId, Comment comment);

    @LinkResource(Comment.class, pathParameters = {"${book.title}", "${id}"})
    @DELETE
    @Path("book/{id}/comment/{cid}")
    public void deleteComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

}
```

Securing entities {#securing-entities}
-----------------

You can restrict which links are injected in the resource based on
security restrictions for the client, so that if the current client
doesn't have permission to delete a resource he will not be presented
with the `"delete"` link relation.

Security restrictions can either be specified on the `@LinkResource`
annotation, or using RESTEasy and EJB's security annotation
`@RolesAllowed` on the JAX-RS method.

  Parameter    Type       Function                                                                                            Default
  ------------ ---------- --------------------------------------------------------------------------------------------------- -----------------------------------------------------------
  constraint   `String`   A UEL expression which must evaluate to true to inject this method's link in the response entity.   Defaults to using `@RolesAllowed` from the JAX-RS method.

  : `@LinkResource` security restrictions

Extending the UEL context {#extend-uel}
-------------------------

We've seen that both the URI template values and the security
constraints of `@LinkResource` use UEL to evaluate expressions, and we
provide a basic UEL context with access only to the entity we're
injecting links in, and nothing more.

If you want to add more variables or functions in this context, you can
by adding a `@LinkELProvider` annotation on the JAX-RS method, its
class, or its package. This annotation's value should point to a class
that implements the `ELProvider` interface, which wraps the default
`ELContext` in order to add any missing functions.

For example, if you want to support the Seam annotation
`s:hasPermission(target, permission)` in your security constraints, you
can add a `package-info.java` file like this:

``` {.Java}
@LinkELProvider(SeamELProvider.class)
package org.jboss.resteasy.links.test;

import org.jboss.resteasy.links.*;
```

With the following provider implementation:

``` {.Java}
package org.jboss.resteasy.links.test;

import javax.el.ELContext;
import javax.el.ELResolver;
import javax.el.FunctionMapper;
import javax.el.VariableMapper;

import org.jboss.seam.el.SeamFunctionMapper;

import org.jboss.resteasy.links.ELProvider;

public class SeamELProvider implements ELProvider {

    public ELContext getContext(final ELContext ctx) {
        return new ELContext() {

            private SeamFunctionMapper functionMapper;

            @Override
            public ELResolver getELResolver() {
                return ctx.getELResolver();
            }

            @Override
            public FunctionMapper getFunctionMapper() {
                if (functionMapper == null)
                    functionMapper = new SeamFunctionMapper(ctx
                            .getFunctionMapper());
                return functionMapper;
            }

            @Override
            public VariableMapper getVariableMapper() {
                return ctx.getVariableMapper();
            }
        };
    }

}
```

And then use it as such:

``` {.Java}
@Path("/")
@Consumes({"application/xml", "application/json"})
@Produces({"application/xml", "application/json"})
public interface BookStore {

    @AddLinks
    @LinkResources({
        @LinkResource(value = Book.class, rel = "comments", constraint = "${s:hasPermission(this, 'add-comment')}"),
        @LinkResource(value = Comment.class, constraint = "${s:hasPermission(this, 'insert')}")
    })
    @GET
    @Path("book/{id}/comments")
    public Collection<Comment> getComments(@PathParam("id") String bookId);

    @AddLinks
    @LinkResource(constraint = "${s:hasPermission(this, 'read')}")
    @GET
    @Path("book/{id}/comment/{cid}")
    public Comment getComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

    @LinkResource(constraint = "${s:hasPermission(this, 'insert')}")
    @POST
    @Path("book/{id}/comments")
    public void addComment(@PathParam("id") String bookId, Comment comment);

    @LinkResource(constraint = "${s:hasPermission(this, 'update')}")
    @PUT
    @Path("book/{id}/comment/{cid}")
    public void updateComment(@PathParam("id") String bookId, @PathParam("cid") String commentId, Comment comment);

    @LinkResource(Comment.class, constraint = "${s:hasPermission(this, 'delete')}")
    @DELETE
    @Path("book/{id}/comment/{cid}")
    public void deleteComment(@PathParam("id") String bookId, @PathParam("cid") String commentId);

}
```

Resource facades {#facades}
----------------

Sometimes it is useful to add resources which are just containers or
layers on other resources. For example if you want to represent a
collection of `Comment` with a start index and a certain number of
entries, in order to implement paging. Such a collection is not really
an entity in your model, but it should obtain the `"add"` and `"list"`
link relations for the `Comment` entity.

This is possible using resource facades. A resource facade is a resource
which implements the `ResourceFacade<T>` interface for the type `T`, and
as such, should receive all links for that type.

Since in most cases the instance of the `T` type is not directly
available in the resource facade, we need another way to extract its URI
template values, and this is done by calling the resource facade's
pathParameters() method to obtain a map of URI template values by name.
This map will be used to fill in the URI template values for any link
generated for `T`, if there are enough values in the map.

Here is an example of such a resource facade for a collection of
`Comment`s:

``` {.Java}
@XmlRootElement
@XmlAccessorType(XmlAccessType.NONE)
public class ScrollableCollection implements ResourceFacade<Comment> {

    private String bookId;
    @XmlAttribute
    private int start;
    @XmlAttribute
    private int totalRecords;
    @XmlElement
    private List<Comment> comments = new ArrayList<Comment>();
    @XmlElementRef
    private RESTServiceDiscovery rest;

    public Class<Comment> facadeFor() {
        return Comment.class;
    }

    public Map<String, ? extends Object> pathParameters() {
        HashMap<String, String> map = new HashMap<String, String>();
        map.put("id", bookId);
        return map;
    }
}
```

This will produce such an XML collection:

``` {.XML}
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<collection xmlns:atom="http://www.w3.org/2005/Atom" totalRecords="2" start="0">
 <atom.link href="http://localhost:8081/book/foo/comments" rel="add"/>
 <atom.link href="http://localhost:8081/book/foo/comments" rel="list"/>
 <comment xmlid="0">
  <text>great book</text>
  <atom.link href="http://localhost:8081/book/foo/comment/0" rel="self"/>
  <atom.link href="http://localhost:8081/book/foo/comment/0" rel="update"/>
  <atom.link href="http://localhost:8081/book/foo/comment/0" rel="remove"/>
  <atom.link href="http://localhost:8081/book/foo/comments" rel="add"/>
  <atom.link href="http://localhost:8081/book/foo/comments" rel="list"/>
 </comment>
 <comment xmlid="1">
  <text>terrible book</text>
  <atom.link href="http://localhost:8081/book/foo/comment/1" rel="self"/>
  <atom.link href="http://localhost:8081/book/foo/comment/1" rel="update"/>
  <atom.link href="http://localhost:8081/book/foo/comment/1" rel="remove"/>
  <atom.link href="http://localhost:8081/book/foo/comments" rel="add"/>
  <atom.link href="http://localhost:8081/book/foo/comments" rel="list"/>
 </comment>
</collection>
```

@CookieParam {#_CookieParam}
============

The @CookieParam annotation allows you to inject the value of a cookie
or an object representation of an HTTP request cookie into your method
invocation

GET /books?num=5

    @GET
    public String getBooks(@CookieParam("sessionid") int id) {
    ...
    }

    @GET
    public String getBooks(@CookieParam("sessionid") javax.ws.rs.core.Cookie id) {...}

Like PathParam, your parameter type can be an String, primitive, or
class that has a String constructor or static valueOf() method. You can
also get an object representation of the cookie via the
javax.ws.rs.core.Cookie class.

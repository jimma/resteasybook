@FormParam {#_FormParam}
==========

When the input request body is of the type
"application/x-www-form-urlencoded", a.k.a. an HTML Form, you can inject
individual form parameters from the request body into method parameter
values.

    <form method="POST" action="/resources/service">
    First name: 
    <input type="text" name="firstname">
    <br>
    Last name: 
    <input type="text" name="lastname">
    </form>

If you post through that form, this is what the service might look like:

    @Path("/")
    public class NameRegistry {

       @Path("/resources/service")
       @POST
       public void addName(@FormParam("firstname") String first, @FormParam("lastname") String last) {...}

You cannot combine @FormParam with the default
"application/x-www-form-urlencoded" that unmarshalls to a
MultivaluedMap&lt;String, String&gt;. i.e. This is illegal:

    @Path("/")
    public class NameRegistry {

       @Path("/resources/service")
       @POST
       @Consumes("application/x-www-form-urlencoded")
       public void addName(@FormParam("firstname") String first, MultivaluedMap<String, String> form) {...}


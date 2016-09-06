JSON Support via Java EE 7 JSON-P API {#json-p}
=====================================

No, this is not the JSONP you are thinking of! JSON-P is a new Java EE 7
JSON parsing API. Horrible name for a new JSON parsing API! What were
they thinking? Anyways, Resteasy has a provider for it. If you are using
Wildfly, it is required by Java EE 7 so you will have it automatically
bundled. Otherwise, use this maven dependency.

    <dependency>
       <groupId>org.jboss.resteasy</groupId>
       <artifactId>resteasy-json-p-provider</artifactId>
       <version>3.1.0-SNAPSHOT</version>
    </dependency>

It has built in support for JsonObject, JsonArray, and JsonStructure as
request or response entities. It should not conflict with Jackson or
Jettison if you have that in your path too.
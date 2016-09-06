YAML Provider {#Built_in_YAML_Provider}
=============

Since 3.1.0-SNAPSHOT release, resteasy comes with built in support for
YAML using the SnakeYAML library. To enable YAML support, you need to
drop in the SnakeYaml 1.8 jar and the resteasy-yaml-provider.jar
(whatever the current version is) in RestEASY's classpath.

SnakeYaml jar file can either be downloaded from Google code at
http://code.google.com/p/snakeyaml/downloads/list

Or if you use maven, the SnakeYaml jar is available through SonaType
public repositories and included using this dependency:

     <dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.8</version>
     </dependency>
          

When starting resteasy look out in the logs for a line stating that the
YamlProvider has been added - this indicates that resteasy has found the
Jyaml jar:

2877 Main INFO org.jboss.resteasy.plugins.providers.RegisterBuiltin -
Adding YamlProvider

The Yaml provider recognises three mime types:

-   text/x-yaml
-   text/yaml
-   application/x-yaml

This is an example of how to use Yaml in a resource method.

     import javax.ws.rs.Consumes;
     import javax.ws.rs.GET;
     import javax.ws.rs.Path;
     import javax.ws.rs.Produces;

     @Path("/yaml")
     public class YamlResource
     {

    @GET
    @Produces("text/x-yaml")
    public MyObject getMyObject() {
       return createMyObject();
    }
    ...
     }
     

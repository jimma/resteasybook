Content-Range Support {#ContentRange}
=====================

Resteasy supports `Range` requests for `java.io.File` response entities.

       @Path("/")
       public class Resource {
          @GET
          @Path("file")
          @Produces("text/plain")
          public File getFile()
          {
             return file;
          }
       }

          Response response = client.target(generateURL("/file")).request()
                  .header("Range", "1-4").get();
          Assert.assertEquals(response.getStatus(), 206);
          Assert.assertEquals(4, response.getLength());
          System.out.println("Content-Range: " + response.getHeaderString("Content-Range"));


          

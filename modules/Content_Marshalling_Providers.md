Content Marshalling/Providers {#Content_Marshalling_Providers}
=============================

Default Providers and default JAX-RS Content Marshalling {#Default_Providers_and_default_JAX-RS_Content_Marshalling}
========================================================

Resteasy can automatically marshal and unmarshal a few different message
bodies.

  Media Types                                                                                             Java Type
  ------------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------
  application/\*+xml, text/\*+xml, application/\*+json, application/\*+fastinfoset, application/atom+\*   JaxB annotated classes
  application/\*+xml, text/\*+xml                                                                         org.w3c.dom.Document
  \*/\*                                                                                                   java.lang.String
  \*/\*                                                                                                   java.io.InputStream
  text/plain                                                                                              primitives, java.lang.String, or any type that has a String constructor, or static valueOf(String) method for input, toString() for output
  \*/\*                                                                                                   javax.activation.DataSource
  \*/\*                                                                                                   java.io.File
  \*/\*                                                                                                   byte\[\]
  application/x-www-form-urlencoded                                                                       javax.ws.rs.core.MultivaluedMap

Content Marshalling with @Provider classes {#Content_Marshalling_with__Provider_classes}
==========================================

The JAX-RS specification allows you to plug in your own request/response
body reader and writers. To do this, you annotate a class with @Provider
and specify the @Produces types for a writer and @Consumes types for a
reader. You must also implement a MessageBodyReader/Writer interface
respectively. Here is an example:

    @Provider
    @Produces("application/x-java-serialized-object")
    @Consumes("application/x-java-serialized-object")
    public class SerializableProvider implements MessageBodyReader<Serializable>, MessageBodyWriter<Serializable>
    {
       public static final MediaType APPLICATION_SERIALIZABLE_TYPE = new MediaType("application", "x-java-serialized-object");
       public static final String APPLICATION_SERIALIZABLE = APPLICATION_SERIALIZABLE_TYPE.toString();
       
       public boolean isWriteable(Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType)
       {
          return Serializable.class.isAssignableFrom(type)
                && APPLICATION_SERIALIZABLE_TYPE.getType().equals(mediaType.getType())
                && APPLICATION_SERIALIZABLE_TYPE.getSubtype().equals(mediaType.getSubtype());
       }

       public long getSize(Serializable t, Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType)
       {
          return -1;
       }

       public void writeTo(Serializable t, Class<?> type, Type genericType,
             Annotation[] annotations, MediaType mediaType,
             MultivaluedMap<String, Object> httpHeaders, OutputStream entityStream)
             throws IOException, WebApplicationException
       {
          BufferedOutputStream bos = new BufferedOutputStream(entityStream);
          ObjectOutputStream oos = new ObjectOutputStream(bos);
          oos.writeObject(t);
          oos.close();
       }

       public boolean isReadable(Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType)
       {
          return Serializable.class.isAssignableFrom(type)
                && APPLICATION_SERIALIZABLE_TYPE.getType().equals(mediaType.getType())
                && APPLICATION_SERIALIZABLE_TYPE.getSubtype().equals(mediaType.getSubtype());
       }

       public Serializable readFrom(Class<Serializable> type, Type genericType,
             Annotation[] annotations, MediaType mediaType,
             MultivaluedMap<String, String> httpHeaders, InputStream entityStream)
             throws IOException, WebApplicationException
       {
          BufferedInputStream bis = new BufferedInputStream(entityStream);
          ObjectInputStream ois = new ObjectInputStream(bis);
          try
          {
             return Serializable.class.cast(ois.readObject());
          }
          catch (ClassNotFoundException e)
          {
             throw new WebApplicationException(e);
          }
       }
    }
          

The Resteasy ServletContextLoader will automatically scan your
WEB-INF/lib and classes directories for classes annotated with @Provider
or you can manually configure them in web.xml. See
Installation/Configuration.

Providers Utility Class {#MessageBodyWorkers}
=======================

javax.ws.rs.ext.Providers is a simple injectable interface that allows
you to look up MessageBodyReaders, Writers, ContextResolvers, and
ExceptionMappers. It is very useful, for instance, for implementing
multipart providers. Content types that embed other random content
types.

    public interface Providers
    {

       /**
        * Get a message body reader that matches a set of criteria. The set of
        * readers is first filtered by comparing the supplied value of
        * {@code mediaType} with the value of each reader's
        * {@link javax.ws.rs.Consumes}, ensuring the supplied value of
        * {@code type} is assignable to the generic type of the reader, and
        * eliminating those that do not match.
        * The list of matching readers is then ordered with those with the best
        * matching values of {@link javax.ws.rs.Consumes} (x/y > x&#47;* > *&#47;*)
        * sorted first. Finally, the
        * {@link MessageBodyReader#isReadable}
        * method is called on each reader in order using the supplied criteria and
        * the first reader that returns {@code true} is selected and returned.
        *
        * @param type        the class of object that is to be written.
        * @param mediaType   the media type of the data that will be read.
        * @param genericType the type of object to be produced. E.g. if the
        *                    message body is to be converted into a method parameter, this will be
        *                    the formal type of the method parameter as returned by
        *                    <code>Class.getGenericParameterTypes</code>.
        * @param annotations an array of the annotations on the declaration of the
        *                    artifact that will be initialized with the produced instance. E.g. if the
        *                    message body is to be converted into a method parameter, this will be
        *                    the annotations on that parameter returned by
        *                    <code>Class.getParameterAnnotations</code>.
        * @return a MessageBodyReader that matches the supplied criteria or null
        *         if none is found.
        */
       <T> MessageBodyReader<T> getMessageBodyReader(Class<T> type,
                                                     Type genericType, Annotation annotations[], MediaType mediaType);

       /**
        * Get a message body writer that matches a set of criteria. The set of
        * writers is first filtered by comparing the supplied value of
        * {@code mediaType} with the value of each writer's
        * {@link javax.ws.rs.Produces}, ensuring the supplied value of
        * {@code type} is assignable to the generic type of the reader, and
        * eliminating those that do not match.
        * The list of matching writers is then ordered with those with the best
        * matching values of {@link javax.ws.rs.Produces} (x/y > x&#47;* > *&#47;*)
        * sorted first. Finally, the
        * {@link MessageBodyWriter#isWriteable}
        * method is called on each writer in order using the supplied criteria and
        * the first writer that returns {@code true} is selected and returned.
        *
        * @param mediaType   the media type of the data that will be written.
        * @param type        the class of object that is to be written.
        * @param genericType the type of object to be written. E.g. if the
        *                    message body is to be produced from a field, this will be
        *                    the declared type of the field as returned by
        *                    <code>Field.getGenericType</code>.
        * @param annotations an array of the annotations on the declaration of the
        *                    artifact that will be written. E.g. if the
        *                    message body is to be produced from a field, this will be
        *                    the annotations on that field returned by
        *                    <code>Field.getDeclaredAnnotations</code>.
        * @return a MessageBodyReader that matches the supplied criteria or null
        *         if none is found.
        */
       <T> MessageBodyWriter<T> getMessageBodyWriter(Class<T> type,
                                                     Type genericType, Annotation annotations[], MediaType mediaType);

       /**
        * Get an exception mapping provider for a particular class of exception.
        * Returns the provider whose generic type is the nearest superclass of
        * {@code type}.
        *
        * @param type the class of exception
        * @return an {@link ExceptionMapper} for the supplied type or null if none
        *         is found.
        */
       <T extends Throwable> ExceptionMapper<T> getExceptionMapper(Class<T> type);

       /**
        * Get a context resolver for a particular type of context and media type.
        * The set of resolvers is first filtered by comparing the supplied value of
        * {@code mediaType} with the value of each resolver's
        * {@link javax.ws.rs.Produces}, ensuring the generic type of the context
        * resolver is assignable to the supplied value of {@code contextType}, and
        * eliminating those that do not match. If only one resolver matches the
        * criteria then it is returned. If more than one resolver matches then the
        * list of matching resolvers is ordered with those with the best
        * matching values of {@link javax.ws.rs.Produces} (x/y > x&#47;* > *&#47;*)
        * sorted first. A proxy is returned that delegates calls to
        * {@link ContextResolver#getContext(java.lang.Class)} to each matching context
        * resolver in order and returns the first non-null value it obtains or null
        * if all matching context resolvers return null.
        *
        * @param contextType the class of context desired
        * @param mediaType   the media type of data for which a context is required.
        * @return a matching context resolver instance or null if no matching
        *         context providers are found.
        */
       <T> ContextResolver<T> getContextResolver(Class<T> contextType,
                                                 MediaType mediaType);
    }

A Providers instance is injectable into MessageBodyReader or Writers:

    @Provider
    @Consumes("multipart/fixed")
    public class MultipartProvider implements MessageBodyReader {

        private @Context Providers providers;

        ...

    }

Configuring Document Marshalling {#Configuring_Document_Marshalling}
================================

XML document parsers are subject to a form of attack known as the XXE
(Xml eXternal Entity) Attack
(<http://www.securiteam.com/securitynews/6D0100A5PU.html>), in which
expanding an external entity causes an unsafe file to be loaded. For
example, the document

    <?xml version="1.0"?>
    <!DOCTYPE foo
    [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
    <search>
        <user>bill</user>
        <file>&xxe;<file>
    </search>

could cause the passwd file to be loaded.

By default, Resteasy's built-in unmarshaller for org.w3c.dom.Document
documents will not expand external entities, replacing them by the empty
string instead. It can be configured to replace external entities by
values defined in the DTD by setting the context parameter

> resteasy.document.expand.entity.references

to "true" in the web.xml file:

    <context-param>
        <param-name>resteasy.document.expand.entity.references</param-name>
        <param-value>true</param-value>
    </context-param>

Another way of dealing with the problem is by prohibiting DTDs, which
Resteasy does by default. This behavior can be changed by setting the
context parameter

> resteasy.document.secure.disableDTDs

to "false".

Documents are also subject to Denial of Service Attacks when buffers are
overrun by large entities or too many attributes. For example, if a DTD
defined the following entities

    <!ENTITY foo 'foo'>
    <!ENTITY foo1 '&foo;&foo;&foo;&foo;&foo;&foo;&foo;&foo;&foo;&foo;'>
    <!ENTITY foo2 '&foo1;&foo1;&foo1;&foo1;&foo1;&foo1;&foo1;&foo1;&foo1;&foo1;'>
    <!ENTITY foo3 '&foo2;&foo2;&foo2;&foo2;&foo2;&foo2;&foo2;&foo2;&foo2;&foo2;'>
    <!ENTITY foo4 '&foo3;&foo3;&foo3;&foo3;&foo3;&foo3;&foo3;&foo3;&foo3;&foo3;'>
    <!ENTITY foo5 '&foo4;&foo4;&foo4;&foo4;&foo4;&foo4;&foo4;&foo4;&foo4;&foo4;'>
    <!ENTITY foo6 '&foo5;&foo5;&foo5;&foo5;&foo5;&foo5;&foo5;&foo5;&foo5;&foo5;'>

then the expansion of &foo6; would result in 1,000,000 foos. By default,
Resteasy will limit the number of expansions and the number of
attributes per entity. The exact behavior depends on the underlying
parser. The limits can be turned off by setting the context parameter

> resteasy.document.secure.processing.feature

to "false".

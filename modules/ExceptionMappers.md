Exception Handling {#ExceptionHandling}
==================

Exception Mappers {#ExceptionMappers}
=================

ExceptionMappers are custom, application provided, components that can
catch thrown application exceptions and write specific HTTP responses.
The are classes annotated with @Provider and that implement this
interface

             package javax.ws.rs.ext;

             import javax.ws.rs.core.Response;

             /**
             * Contract for a provider that maps Java exceptions to
             * {@link javax.ws.rs.core.Response}. An implementation of this interface must
             * be annotated with {@link Provider}.
             *
             * @see Provider
             * @see javax.ws.rs.core.Response
             */
             public interface ExceptionMapper<E>
             {
                /**
                * Map an exception to a {@link javax.ws.rs.core.Response}.
                *
                * @param exception the exception to map to a response
                * @return a response mapped from the supplied exception
                */
                Response toResponse(E exception);
             }
          

When an application exception is thrown it will be caught by the JAX-RS
runtime. JAX-RS will then scan registered ExceptionMappers to see which
one support marshalling the exception type thrown. Here is an example of
ExceptionMapper


             @Provider
             public class EJBExceptionMapper implements ExceptionMapper<javax.ejb.EJBException>
             {
                public Response toResponse(EJBException exception) {
                   return Response.status(500).build();
                }
             }
          

You register ExceptionMappers the same way you do
MessageBodyReader/Writers. By scanning, through the resteasy provider
context-param (if you're deploying via a WAR file), or programmatically
through the ResteasyProviderFactory class.

Resteasy Built-in Internally-Thrown Exceptions {#builtinException}
==============================================

Resteasy has a set of built-in exceptions that are thrown by it when it
encounters errors during dispatching or marshalling. They all revolve
around specific HTTP error codes. You can find them in RESTEasy's
javadoc under the package org.jboss.resteasy.spi. Here's a list of them:

  Exception                                             HTTP Code   Description
  ----------------------------------------------------- ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ReaderException                                       400         All exceptions thrown from MessageBodyReaders are wrapped within this exception. If there is no ExceptionMapper for the wrapped exception or if the exception isn't a WebApplicationException, then resteasy will return a 400 code by default.
  WriterException                                       500         All exceptions thrown from MessageBodyWriters are wrapped within this exception. If there is no ExceptionMapper for the wrapped exception or if the exception isn't a WebApplicationException, then resteasy will return a 400 code by default.
  o.j.r.plugins.providers.jaxb.JAXBUnmarshalException   400         The JAXB providers (XML and Jettison) throw this exception on reads. They may be wrapping JAXBExceptions. This class extends ReaderException
  o.j.r.plugins.providers.jaxb.JAXBMarshalException     500         The JAXB providers (XML and Jettison) throw this exception on writes. They may be wrapping JAXBExceptions. This class extends WriterException
  ApplicationException                                  N/A         This exception wraps all exceptions thrown from application code. It functions much in the same way as InvocationTargetException. If there is an ExceptionMapper for wrapped exception, then that is used to handle the request.
  Failure                                               N/A         Internal Resteasy. Not logged
  LoggableFailure                                       N/A         Internal Resteasy error. Logged
  DefaultOptionsMethodException                         N/A         If the user invokes HTTP OPTIONS and no JAX-RS method for it, Resteasy provides a default behavior by throwing this exception

Overriding Resteasy Builtin Exceptions {#overring_resteasy_exceptions}
======================================

You may override Resteasy built-in exceptions by writing an
ExceptionMapper for the exception. For that matter, you can write an
ExceptionMapper for any thrown exception including
WebApplicationException

Asynchronous Job Service {#async_job_service}
========================

The Resteasy Asynchronous Job Service is an implementation of the
Asynchronous Job pattern defined in O'Reilly's "Restful Web Services"
book. The idea of it is to bring asynchronicity to a synchronous
protocol.

Using Async Jobs {#async_job}
================

While HTTP is a synchronous protocol it does have a faint idea of
asynchronous invocations. The HTTP 1.1 response code 202, "Accepted"
means that the server has received and accepted the response for
processing, but the processing has not yet been completed. The Resteasy
Asynchronous Job Service builds around this idea.

    POST http://example.com/myservice?asynch=true

For example, if you make the above post with the asynch query parameter
set to true, Resteasy will return a 202, "Accepted" response code and
run the invocation in the background. It also sends back a Location
header with a URL pointing to where the response of the background
method is located.

    HTTP/1.1 202 Accepted
    Location: http://example.com/asynch/jobs/3332334

The URI will have the form of:

    /asynch/jobs/{job-id}?wait={millisconds}|nowait=true

You can perform the GET, POST, and DELETE operations on this job URL.
GET returns whatever the JAX-RS resource method you invoked returned as
a response if the job was completed. If the job has not completed, this
GET will return a response code of 202, Accepted. Invoking GET does not
remove the job, so you can call it multiple times. When Resteasy's job
queue gets full, it will evict the least recently used job from memory.
You can manually clean up after yourself by calling DELETE on the URI.
POST does a read of the JOB response and will remove the JOB it has been
completed.

Both GET and POST allow you to specify a maximum wait time in
milliseconds, a "wait" query parameter. Here's an example:

    POST http://example.com/asynch/jobs/122?wait=3000

If you do not specify a "wait" parameter, the GET or POST will not wait
at all if the job is not complete.

NOTE!! While you can invoke GET, DELETE, and PUT methods asynchronously,
this breaks the HTTP 1.1 contract of these methods. While these
invocations may not change the state of the resource if invoked more
than once, they do change the state of the server as new Job entries
with each invocation. If you want to be a purist, stick with only
invoking POST methods asynchronously.

Security NOTE! Resteasy role-based security (annotations) does not work
with the Asynchronous Job Service. You must use XML declarative security
within your web.xml file. Why? It is impossible to implement role-based
security portably. In the future, we may have specific JBoss
integration, but will not support other environments.

Oneway: Fire and Forget {#oneway}
=======================

Resteasy also supports the notion of fire and forget. This will also
return a 202, Accepted response, but no Job will be created. This is as
simple as using the oneway query parameter instead of asynch. For
example:

    POST http://example.com/myservice?oneway=true

Security NOTE! Resteasy role-based security (annotations) does not work
with the Asynchronous Job Service. You must use XML declaritive security
within your web.xml file. Why? It is impossible to implement role-based
security portably. In the future, we may have specific JBoss
integration, but will not support other environments.

Setup and Configuration {#async_job_setup}
=======================

You must enable the Asynchronous Job Service in your web.xml file as it
is not turned on by default.


    <web-app>
        <!-- enable the Asynchronous Job Service -->
        <context-param>
            <param-name>resteasy.async.job.service.enabled</param-name>
            <param-value>true</param-value>
        </context-param>

        <!-- The next context parameters are all optional.  
             Their default values are shown as example param-values -->

        <!-- How many jobs results can be held in memory at once? -->
        <context-param>
            <param-name>resteasy.async.job.service.max.job.results</param-name>
            <param-value>100</param-value>
        </context-param>

        <!-- Maximum wait time on a job when a client is querying for it -->
        <context-param>
            <param-name>resteasy.async.job.service.max.wait</param-name>
            <param-value>300000</param-value>
        </context-param>

        <!-- Thread pool size of background threads that run the job -->
        <context-param>
            <param-name>resteasy.async.job.service.thread.pool.size</param-name>
            <param-value>100</param-value>
        </context-param>

        <!-- Set the base path for the Job uris -->
        <context-param>
            <param-name>resteasy.async.job.service.base.path</param-name>
            <param-value>/asynch/jobs</param-value>
        </context-param>

        <listener>
            <listener-class>
                org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap
            </listener-class>
        </listener>

        <servlet>
            <servlet-name>Resteasy</servlet-name>
            <servlet-class>
                org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
            </servlet-class>
        </servlet>

        <servlet-mapping>
            <servlet-name>Resteasy</servlet-name>
            <url-pattern>/*</url-pattern>
        </servlet-mapping>

    </web-app>

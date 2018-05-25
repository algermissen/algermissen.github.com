---
layout: post
title: Declarative Cache Control with JAX-RS 2.0
---
{% include JB/setup %}

**UPDATE**: The Maven/IDE setup below turned out not to work properly in all cases. Try [this](http://jalg.net/2012/11/getting-started-playing-around-with-jax-rs-2-0-in-an-ee-container/) for better results.

The final release of JAX-RS 2.0 is nearing. Time for a closer look at the new features.

As JAX-RS 2.0 isn't yet part of any standard release, some up front work is inevitable. Fortunately, [Marek](http://marek.potociar.net) has some [excellent write ups](http://marek.potociar.net/2012/08/10/jersey-2-0-m06-has-been-released/) to get us started using the [JAX-RS](http://jax-rs-spec.java.net) 2.0 reference implementation [Jersey 2](http://jersey.java.net/jersey20.html).

I am currently working with Eclipse a lot and here is how you quick-start a JAX-RS 2.0 project with Jersey 2:

In Eclipse, choose New -> Other -> Maven Project and create an archetype-based project. For this you first need to add the archetype Marek is mentioning in his post (I am using the Grizzly version here):

[caption id="attachment_132" align="alignnone" width="614"][![](http://jalg.net/wp-content/uploads/2012/09/jersey2archetype.png)](http://jalg.net/wp-content/uploads/2012/09/jersey2archetype.png)[/caption]

Then, create a project with that archetype:

[caption id="attachment_135" align="alignnone" width="598"][![](http://jalg.net/wp-content/uploads/2012/09/selectArchetype.png)](http://jalg.net/wp-content/uploads/2012/09/selectArchetype.png)[/caption]

Now you are all set and ready to explore JAX-RS 2.0.

Making JAX-RS responses cacheable isn't exactly elegant in 1.1. If you want caching, you need to return a Response object and manually add Cache-Control headers to it. Far from ideal. (And it gets even uglier if you try to unit test your resource classes just to find out that you cannot do so if you return Response objects because the idiomatic invocation of build() demands a JAX-RS runtime).

What, if we instead could annotate a resource method to decorate the response with a Cache-Control header? Turns out, that this is straight forward in JAX-RS 2.0 thanks to the new Filter API.

All we need is a filter and an annotation to selectively bind that filter to any resource method of our liking. Here is the annotation which takes a Cache-Control header value as an argument:
    
    <code>
    package net.jalg.ccdecor;
    import java.lang.annotation.*;
    import javax.ws.rs.NameBinding;
    
    @NameBinding
    @Target({ ElementType.TYPE, ElementType.METHOD })
    @Retention(value = RetentionPolicy.RUNTIME)
    public @interface Cacheable {
      String cc() default "public, must-revalidate"; 
    }
    </code>

The new @NameBinding annotation tells a JAX-RS 2.0 runtime that our annotation should be used to match filters to resource methods.

Here is the filter:
    
    <code>
    package net.jalg.ccdecor;
    
    import java.io.IOException;
    import java.lang.annotation.Annotation;
    
    import javax.ws.rs.container.ContainerRequestContext;
    import javax.ws.rs.container.ContainerResponseContext;
    import javax.ws.rs.container.ContainerResponseFilter;
    import javax.ws.rs.ext.Provider;
    
    @Provider
    @Cacheable
    public class CacheControlDecorator implements ContainerResponseFilter {
    
      @Override
      public void filter(ContainerRequestContext requestContext,
            ContainerResponseContext responseContext) throws IOException {
        for (Annotation a : responseContext.getEntityAnnotations()) {
          if (a.annotationType() == Cacheable.class) {
            String cc = ((Cacheable) a).cc();
            responseContext.getHeaders().add("Cache-Control", cc);
          }
        }
      }
    }
    </code>

Don't worry too much about that code for a minute but look at how the filter can be bound to a resource method:
    
    <code>
      @GET
      @Cacheable(cc="public, maxAge=3600")
      @Produces(MediaType.TEXT_PLAIN)
      public String getIt() {
        return "Got it!";
      }
    </code>

The binding is achieved using our @Cacheable annotation.

Using annotation arguments it is possible to pass usage information to the filter. The annotations of the resource method are available in the filter by way of the getEntityAnnotations() method of the passed ContainerResponseContext argument (see above).

We just need to extract the cache control information from the annotation and add the Cache-Control header to the response.

You can download the example project from [github](https://github.com/algermissen/cache-control-decorator).



---
layout: post
title: Using iron to Encapsulate Cookies
---
{% include JB/setup %}

[Eran Hammer](http://hueniverse.com) has recently created [iron](https://github.com/hueniverse/iron), a cryptographic procedure and tool to seal arbitrary data in a way so that it cannot be read and also cannot be changed without being noticed.

Besides its intended use in combination with [Oz](https://github.com/hueniverse/oz) it can also be used in other scenarios. One of them being encapsulated HTTP cookies. While it is in no way a new thing to pass state to Web clients in encrypted form so they cannot read it or tamper with it, doing it right is relatively hard. Hence, it is great to now have a quasi-standard and sound [definition](https://github.com/hueniverse/iron#introduction).

**Update**: Looking something up in [Theo Schlossnagle](https://twitter.com/postwait)'s [_Building Scalable Internet Architectures_](http://omniti.com/writes/scalable-internet-architectures) I found a passage _Addressing Security and Integrity_ in chapter 7, p.130 that resembles the topic of this post.


To bring iron into the Java world, I have created [jiron](https://github.com/algermissen/jiron) and an [example project](https://github.com/algermissen/iron-cookie) to illustrate how to use iron for HTTP cookie authentication with encapsulated tokens.

First, let's see, how to use jiron to seal and unseal data:
    
    import net.jalg.jiron.Jiron;
    
    String data = "Some secret stuff we want protected";
    
    String sealed = Jiron.seal(data, "sealing-password",
                                            Jiron.DEFAULT_ENCRYPTION_OPTIONS,
                                            Jiron.DEFAULT_INTEGRITY_OPTIONS);
       
    String unsealed = Jiron.unseal(sealed, "sealing-password",
    					Jiron.DEFAULT_ENCRYPTION_OPTIONS,
    					Jiron.DEFAULT_INTEGRITY_OPTIONS);

The String `sealed` is what you can pass around and be sure that nobody can read or modify it, unless in possession of the sealing password.

Suppose you have a Web site that employs cookie-based authentication and suppose you have some data you want to store in that cookie directly so you do not have to go to a database shared by all server instances for looking up that data. For example the user's login, her real name and information when the cookie expires and re-authentication is enforced.

Initially, a user not in possession of the cookie will be redirected to a login form and the cookie will be issued upon submission of valid credentials. Here is an excerpt of the [form processing JAX-RS resource](https://github.com/algermissen/iron-cookie/blob/master/src/main/java/net/jalg/ironcookie/LoginResource.java):
    
    String data = login + "|" + expires + "|" + realname;
    String cookieValue = Jiron.seal(data, ApplicationConfig.ENCRYPTION_KEY,
    					Jiron.DEFAULT_ENCRYPTION_OPTIONS,
    					Jiron.DEFAULT_INTEGRITY_OPTIONS);
    
    NewCookie c = new NewCookie(AuthFilter.COOKIE_NAME, cookieValue);
    return Response.seeOther(redirectUri).cookie(c).build();

In the HTTP response the cookie will look something like (line breaks added for clarity):
    
    Set-Cookie:authtoken=Fe26.1**7A6A689DEC9563773B1264CF4CC585CFC9CF84403EE9143650D2EC2EE<br></br>24E36A3*k9yW1ZlC5Tr1a5o2Os_QMQ*sxZkQJfHsfyZE_0DI3ugHKdQ3IXwy1jySoz7GrKiTWU*9C58B956A11<br></br>81D1F1E3A1A7A1B67D2CF7738B0BFB16C5EB4B381151CAEEC4C6C*3PxSsg5aThLGvU2e8ItXfep-hpMw5x96<br></br>L_PbelhnU84;Version=1
    
To protect [a resource class](https://github.com/algermissen/iron-cookie/blob/master/src/main/java/net/jalg/ironcookie/DashboardResource.java) by enforcing cookie auth, we bind a JAX-RS 2.0 filter to that class using a new [annotation](https://github.com/algermissen/iron-cookie/blob/master/src/main/java/net/jalg/ironcookie/TokenAuthProtected.java):
    
    //...
    @TokenAuthProtected
    public String getDashboard(@Context SecurityContext sc) { ... }

The [auth enforcing filter class](https://github.com/algermissen/iron-cookie/blob/master/src/main/java/net/jalg/ironcookie/AuthFilter.java) extracts the cookie, checks expiration time and puts the data contained in the cookie in a place where it can be accessed by the resource class.

    Cookie cookie = context.getCookies().get(COOKIE_NAME);
    // Redirect to form if no cookie
    String data = Jiron.unseal(cookie.getValue(), ApplicationConfig.ENCRYPTION_KEY,
    					Jiron.DEFAULT_ENCRYPTION_OPTIONS,
    					Jiron.DEFAULT_INTEGRITY_OPTIONS);
    // ...
    String[] fields = data.split("\\|");
    final String username = fields[0];
    final long expires = Long.parseLong(fields[1]);
    final String realname = fields[2];
    // ...
    if (now > expires) {
      context.abortWith(Response.temporaryRedirect(loginFormUri).build());
      return;
    }
    
    context.setSecurityContext(new TokenSecurityContext(username, realname));

Using the security context and a custom Principal, the resource class is provided access to the user's login and realname.

(There is currently discussion in the JAX-RS 2 expert group to enable passing information from filters to resource classes using properties on the filter chain. That would be the better solution than the SecurityContext/Principal 'hack').

Please have a look at the source code for the code details.


---
layout: post
title: ESI Facelift
---
{% include JB/setup %}

Recently [Mark](http://www.mnot.net) [described](http://www.mnot.net/blog/2011/10/21/why_esi_is_still_important_and_how_to_make_it_better) why [Edge-Side-Includes](http://en.wikipedia.org/wiki/Edge_Side_Includes) are still a relevant technology and presented some ideas regarding an overhaul of [ESI 1.0](http://www.w3.org/TR/esi-lang).

Coincidentally, I ran into an interesting use case for ESI at about the same time and our requirements seem to overlap quite a bit.

Before I go into the details of what I would like to see in ESI 2.0 (yes, the major version number change indicates incompatibility) here are the aspects that need to be considered during the design process:

### Language Syntax

ESI 1.0 is an XML language. While this was the obvious choice ten years ago when XML was the proverbial hammer to every nail times have changed. To be a good citizen, ESI 2.0 would need to be friendly to non-XML formats such as JSON and [certain](http://www.ietf.org/rfc/rfc2445.txt) [plain text](ttp://www.ietf.org/rfc/rfc2426.txt) [species](http://www.w3.org/TeamSubmission/turtle/), too.

This isn't too much of a change, given that none of the ESI implementations I have looked at actually parse ESI tags as XML. Which makes sense, because the surrounding (X)HTML is almost guaranteed to choke an XML parser anyhow.

### Functionality

This aspect covers two issues, really: what new functionality should ESI 2.0 support and which of the older features can be dropped? Mark mentions (among others) templating capabilities for the latter and <esi:inline> and the _alt_-attribute for the former.

### Cacheability

Given that the major advantage of integrating services at the edge is leveraging the existing caching architecture the design process of ESI 2.0 requires a close eye on how well the language features can benefit from caching. 

### Implementability

Whatever the result of an ESI 2.0 design effort, its implementation must be possible using the given architectures of the common open source and non-open source caching products. For example, ESI 2.0 features must not get in the way of an asynchronous request processing model.

### An ESI 2.0 Use Case

Yesterday I [described](http://jalg.net/2011/11/cool-uris-and-integration/) an integration scenario where articles in a content management system (CMS) reference (via URIs) company information maintained in another system (CIMS). The articles are published as Web pages and somewhere in the Web page delivery process the company information references are resolved and the returned information is included in the delivered page. The purpose of this setup is twofold: On the one hand the intention is to deliver up to date company information (maybe the stock price is included). On the other hand a clear separation of concerns is achieved, facilitating independent operation and evolution of the two systems.

There are traditionally two approaches for assembling the article Web page from the data provided by the two services: [Server Side Templating](http://en.wikipedia.org/wiki/Web_template_system#Server-side_systems) (e.g. JSPs or Server-Side-Includes) and [Client Side Templating](http://en.wikipedia.org/wiki/Web_template_system#Client-Side_Systems) (e.g. AJAX or iframes). However, and here is where the topic of this posting comes in again, there is also a third option: assembling the content on the edge, in short: _Edge Side Templating_.

And this is where Mark's ideas overlap with mine. (Which I am thankful for because I wasn't entirely sure until our paths crossed, whether embedding a templating mechanism in a Web cache was actually as useful as I thought)

Regarding the discussion about the pros and cons of doing the templating on the edge please refer to Mark's [posting](http://www.mnot.net/blog/2011/10/21/why_esi_is_still_important_and_how_to_make_it_better) - in the comments there are a bunch of arguments made already for either side.

Having said all that, let me now talk about what I would like to see in ESI 2.0.

### Language Syntax

While in my scenario the including entity (the article referencing the company information) would likely be HTML I have said above that we need something more non-XML friendly. One way would be to reuse the existing ESI XML-tags but explicitly allowing them in non-XML content:
    
    GET /article/123
    Host: cms.example.org
    
    200 Ok
    Content-Type: text/plain
    
    ACME in the News
    
    <esi:include src="http://cims.example.org/company/acme"></esi:include>
    
(I am deliberately excluding any [ESI headers](http://www.w3.org/TR/edge-arch) from all the examples)

As far as I can tell at the moment, this would work for non-XML and still allow XML-friendly embedding in XML response entities. If <esi:...> is too likely to overlap with actual content more 'mangled tags' would need to be used, e.g. (<% ... %>).

### Obsolete ESI 1.0 Functionality

There are a number of more or less sophisticated features in ESI 1.0 that are seldom used and in addition are not implemented by many products. Among these are 
  
  * The alt-attribute on <esi:inline>
  * <esi:choose>,<esi:when>,<esi:otherwise>: it is likely that this branching construct can be replaced with functionality that will be in the templating language anyway (as Mark suggests in his posting, showing a dynamicly assembled include target URI: <esi:include src="/{user_prefs.top_left_module}"/>)
  * <esi:vars>: would likely be changed; I agree with Mark's suggestions regarding variable handling by the templating language as well as with his ideas on providing better access to the request and response via predefined objects
  * <esi:inline> when you first look at it, it appears rather weired and [it took me a while to figure out how inline works](http://stackoverflow.com/questions/7555172/edge-side-includes-how-do-esiinline-tags-work). But I think <esi:inline> is pretty useful for emulating batch retrievals. However, what I do not like about it is that it couples the referencing and the referenced systems: they need to agree on where to place the inline elements and that is pure out-of-band information coupling hell. I'd rather like to see the combination of [pipelining](http://www.mnot.net/blog/2011/08/05/pipeline_now) and [caching](http://www.mnot.net/cache_docs/) to handle batch retrieval requirements.

Besides the above, the list of features to drop should of course be driven by what people actually implemented and use today.

But it is time now to turn to the most interesting aspect of all this: new functionality.

### Edge-Side-Templating

This is also #1 on Mark's list and he refers to it as the ability "to source variables from a URI". The basic idea is this: an ESI element (say <esi:load>) tells the ESI processor to initiate a request to a target resource and make the response entity available as a variable. That variable could then be used in expressions to assemble the overall Web page.
    
    GET /article/123
    Host: cms.example.org
    
    200 Ok
    Content-Type: text/html
    
    <html>
    
    <h1>ACME in the News</h1>
    <p>
    <esi:load var="acme" href="http://cims.example.org/company/acme"></esi:load>
    Yesterday, ${acme.name} (${acme.stock.symbol}) had a closing stock price of ${acme.stock.price}.
    </p>
    </html>

However, there are two problems with this.

The first problem is that there is code in the ESI processor that needs to parse the response entity of the load-request and make it accessible under the variable name. What syntax would this parser expect? Should JSON be mandatory? Or better: XML and JSON because they can both be turned in a generic fashion into the same variable structure? What if both are not feasible because we do not want to change the server of the loaded resource? I can hear a plugin mechanism or code-on-demand knocking at the door. Welcome complexity and feature bloat...

The second problem is that the code that uses the variable makes assumptions about its structure - apparently based on out-of-band knowledge. This makes my REST alarm bells go off immediately!

A solution to this issue would be to add an _accept_ attribute to <esi:load> that would directly translate to the Accept header of the load-request.
    
    GET /article/123
    Host: cms.example.org
    
    200 Ok
    Content-Type: text/html
    
    <html>
    
    <h1>ACME in the News</h1>
    <p>
    <esi:load var="acme" href="http://cims.example.org/company/acme" accept="application/company+json"></esi:load>
    Yesterday, ${acme.name} (${acme.stock.symbol}) had a closing stock price of ${acme.stock.price}.
    </p>
    </html>

Ah, yes. That immediately looks nice and RESTful.

Caveat: Do you see the next feature bloat? We could have all sorts of stuff fiddling with the request via more attributes: accept-language,... and then branching based on the response: onContentType=".." do this or onError="406" do that. User Agent scripting, sort of.

It will be a real challenge to pick those features that add real value and maintain simplicity at the same time.

Regarding problem #1 above this might be one of the rare true use cases for the +json and [+xml](http://tools.ietf.org/html/rfc3023#appendix-A) media type suffixes because they provide an elegant way for the ESI processor to pick parsers based on the Content-Type header without entity introspection.

### Request- and Response Objects

Mark proposes the addition of prepopulated request- and response objects to obtain request information and set response parameters. I agree that this would be extremely useful (and be far more consistent compared to the current ESI variable set).

However, the same warning applies here: This can end up pretty quickly in a means to script the ESI processor in all sorts of ways and clever balance is required between feature set, and processing performance and implementability.

After all, we want to improve ESI, not put a Java EE container into an HTTP cache...

### Timeout and Error Handling

This is also from Mark's list and pretty important. I wonder whether the cache itself would be allowed to override an timeout attribute on <esi:load> or <esi:include>.

I'll skip error handling for now - the post has grown far too long already.

### Templating Language Functionality

This one though needs particular attention. A templating mechanism immediately raises the question of how much functionality one wants to offer. Variable souring and access is an inherent requirement. Mark also mentions string manipulation functions. For example:
    
    <esi:load var="acme" href="http://cims.example.org/company/acme"></esi:load>
    Yesterday, ${substr(acme.name,0.10)} (${toUpper(acme.stock.symbol)}) had a closing stock price of ${acme.stock.price}.

I think branching and looping constructs also make sense but they might conflict with the intermdiary processing model and request handling performance.

The further new functionality is taken, the closer we get to a full blown scripting engine (e.g. a JavaScript engine) and this sounds definitely not like what we want for ESI 2.0 (..or do we??)

Either way, it is again a tradeoff to be made between the size of the feature set, and implementability and performance.

One additional thing to consider with regard to the feature set is the effect on service independence. If the feature set is too small there might be requirements on the service that manages the loaded resource. For example, if there is no string function to uppercase a string that service owner might receive a request to change to service implementation to also include certain data in uppercase form (because that cannot be done in the ESI processor). 

I think that this is the line to draw because it does not make sense to do RESTbased integration to achieve maximal service independence and then let an <esi:include>ing service determine aspects of the implementation of other services. (See the criticism on <esi:inline> above).

That is my take on ESI 2.0 so far.

I would be happy to hear who else is interested in pursuing this further.

### UPDATE

Here is a list of issues that come to my mind as I play around with reworking ESI:

  * When ESI elements are not defined to be XML anymore, should the ESI processor apply any encoding found in the XML preamble or should it in any case only apply the one found in the HTTP headers?


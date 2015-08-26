+++
title = "Thoughts about current APIs implementations"
description = "After I've been consuming several REST APIs for a while I've found that some of them fails in small things which aren't just consumer preferences"
date = "2015-03-30T11:00:00+00:00"
tags = ["API", "Software Engineering", "REST", "Implementation", "HTTP", "Communication Protocol", "Service"]
categories = [
  "Software Engineering"
]
+++

Nowadays there is a __big amount of Internet services which operate over a <a href="http://en.wikipedia.org/wiki/Representational_state_transfer" target="_blank">REST</a> <a href="http://en.wikipedia.org/wiki/Application_programming_interface" target="_blank">API</a>__, and a big part of these services are businesses whose product/service is 100% offered/delivered/exposed over it, and moreover most of these business can be classified under the "start-up" fuzzy term (<a href="http://en.wikipedia.org/wiki/Startup_company" target="_blank" rel="nofollow">1</a>, <a href="http://www.urbandictionary.com/define.php?term=startup" target="_blank" rel="nofollow">2</a>, etc).

This trending has been very good adopted last years for developer communities, __<a href="http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol" target="_blank">HTTP</a> has become the standard communication (application layer) protocol__ (you can blame Internet about it), until reach the point that if a service doesn't have a REST API then it may be discarded over another one whose quality is worse but offer a REST API.

REST is not the only way to expose a service over HTTP, for example __SOAP is another one and it's standard unlike REST which is a software architecture style__, however I guess that the adoption for REST instead of others has been the simplicity (<a href="http://json.org/" target="_blank">JSON</a> data format helped a lot); the tradeoff is, that now, developers, as consumers, face with a variety of ways to define the same, versioning, error handling, etc., and as producers (REST API's owner), what ways to choose to expose those.

I've been consuming quite a bit HTTP APIs, the most of them in hackathons, others for professional purpose and never produced one for professional purpose nonetheless we're producing one in the company where I'm currently working and facing the challenge to make decisions as the <a href="https://news.ycombinator.com/item?id=1523664" target="_blank">versioning one (URL vs headers)</a>.

Here, I'm not going to fire the next hot discussion about API best practices, I'm not currently the right person for that, as I've said, I've never built one for the real world, so I believe that there are better people in the world who can write post or speak about it, for example, people who are talking in <a href="http://www.meetup.com/London-API-Group/" target="_blank" rel="nofollow">London API</a>, which is co-organised by a friend of mine (<a href="https://twitter.com/orliesaurus" target="_blank" rel="nofollow">@orliesaurus</a>).


Therefore I'm going to write about three aspects which annoys me but at the same time I consider that they are quite objective.


## Why not use the standard when the standard exist

Last Saturday, as quite usual, I was attending to a Hackathon; the hackathon was focused in using one REST API service.

The required API to use was related with __getting information__ about transportation; I used one of the resources which returns the route to go from a specified origin (coordinates point) to a specified destination, however it requires a bunch of parameters more as start time, type of transport, and so on. Clearly, the end point is to get information, then the request should be a GET request, however the API required a POST, why?

For my understanding the right HTTP verb is GET __<sup>1</sup>__, because the <a href="https://tools.ietf.org/html/rfc7231#section-4.3.1" target="_blank">HTTP standard says</a>

> The GET method requests transfer of a current selected representation
  for the target resource.  GET is the primary mechanism of information
  retrieval and the focus of almost all performance optimizations.
  Hence, when people speak of retrieving some identifiable information
  via HTTP, they are generally referring to making a GET request.

Then, I guess that the decision of using POST is because the parameters are sent in the request body due that they aren't nice values to use as URL path moreover they are too many to send them in URLs' query and I agree to send them in the body to keep a clean URL, nevertheless I disagree to send them by POST, mainly because <a href="https://tools.ietf.org/html/rfc7231#section-4.3.3" target="_blank">the HTTP standard says</a>

> The POST method requests that the target resource process the
  representation enclosed in the request according to the resource's
  own specific semantics.


hence, I've made the assumption that the decision to use POST was, because they believed that GET cannot contains body __<sup>2</sup>__, however the standard doesn't say that it cannot contain body, it says

> A payload within a GET request message has no defined semantics;
  sending a payload body on a GET request might cause some existing
  implementations to reject the request.


This is not the only place that, we can find examples which drive us to make the assumption that GET cannot hold body, other example is the well known and useful REST client tool called <a href="https://www.getpostman.com/" target="_blank" rel="nofollow">Postman</a> where if you cannot make a GET request with body as you can see in the next picture how GET doesn't offer you the input box to put the body content


<img alt="Postman screenshot to make a GET request" src="https://s-media-cache-ak0.pinimg.com/originals/7a/1b/73/7a1b73e3356abd0931fbdfad47c41afd.jpg" class="graphic-medium graphic-medium-centre">

instead of POST

<img alt="Postman screenshot to make a POST request" src="https://s-media-cache-ak0.pinimg.com/originals/0f/7d/f5/0f7df545b03812b36a8c6ec80f501084.jpg" class="graphic-medium graphic-medium-centre">

Perhaps the owner of the mentioned API was thinking in this kind of conflict and preferred to expose the resource by POST.

I have to recognise that the examples of the real world confused me on it as well, so I've learnt the lesson, try always to read the standard or at least do it when I'm slightly in doubt.


By they way, there is an <a href="https://github.com/a85/POSTMan-Chrome-Extension/issues/919" target="_blank" rel="nofollow">issue in Postman Github repository about this</a>, however it seems that the problem is the platform where Postman runs.


## Use different URLs for different resources

How the API make available the resources is something that matters therefore __creating URLs which make sense with the exposed resource, help a lot to understand the resource, the returned data, moreover that it'll make the life easier to write the documentation__ to explain what the resource return and what parameters require and support.

Take a look to this API resource

<img alt="screenshot of documentation page of /items resource of visit Britain REST API" src="https://s-media-cache-ak0.pinimg.com/originals/5a/c9/68/5ac968db19e80a86f62f9c4996d2b6f9.jpg" class="graphic-medium graphic-medium-centre">

Basically you get a lit of categories which are entities to catalogue locations with the same resource that you get locations using the query parameter "type", therefore the data structure of the response body, obviously is different for each type and in the resource documentation isn't clear at all to know that parameters aren't ignored on each type.

Maybe I miss something, but why haven't they created a different resource for each type, or at leas for type "category" and "locations"?


## Use a standardised data format

Another important thing to appreciate is the returned data format. Ideally, the __API should return two, JSON and XML__ (yes XML it's not dead) and if it only returns one, I'd suggest JSON, at least today, maybe not some years ago.

If an API returns raw text, then the developer will have a lot of work parsing the raw data besides just struggling of making the transformations of the returned data structure to the structure which fulfill their requirements.

This may seems that it doesn't happen, and today any api return JSON or XML or both, well, then you can take a look to <a href="http://www.clickatell.com/apis-scripts/apis/http-s/" target="_blank" rel="nofollow">Clickatell HTTP API</a> <a href="http://www.clickatell.com/downloads/http/Clickatell_HTTP.pdf" target="_blank" rel="nofollow">documentation</a> <sup>pdf doc</sup>

or just forget to open the PDF document and waste your time searching where about you can see it and just take a quick look to the example that I copied from it

```
ID: xxxxx
or
ERR: Error number, error description
```

Nice, no?


## Wrapping up

Nowadays the works is full of REST APIs, they are not standardised, so several things are implemented/exposed in different ways, versioning, parameters to send, etc.

Event though they are implemented differently, if they keep the resources under more or less meaning URLs (paths with words which match with the returned content) and send the data in at least one of the spread formats (JSON or XML), then the pains to use them are reduced quite a bit.

I haven't mentioned anything about API documentations, I think that it's quite evident that the API needs a decent documentation otherwise developer will have to waste their time doing reverse engineering the exposed API; appreciated things to appear in the documentation are how to authenticate (in case that it's needed), the exposed resources with, how to be requested (verb, params, the headers), what is the response data structure when succeed and when fails, response headers to consider and maybe some other that doesn't come up now to my brain. A clear document format oriented to browse the resources and a play ground are also very helpful.

In my case I appreciate that the API use the HTTP standard as much as possible, when it cannot fulfill the requirements, then it's OK, but using the standard inappropriately or just using an alternative non-standard way to do something when the standard offer one is bad.


What do you think about the APIs that you use? have you always found clear APIs?


----
__I've update this post__ on 14/04/2015 adding the next two footnotes after I've realised that I wrote something that it's not totally right; I've also written <a href="{{< ref "post/clarifying-my-mistake-get-requests-with-body.md" >}}" target="_blank">a second blog post about my mistake</a> which gives a better clarification than this two footnotes.

<p class="definition"><sup>1</sup> The right verb is still being GET, if POST is used as the mentioned API does, however if GET is used it must be without BODY and all the parameters must be in the URI through query parameters or in some part of the path it make sense create different resources.</p>
<p class="definition"><sup>2</sup> The current HTTP/1.1 standard doesn't disallow to use BODY in GET, however it mustn't considered, so it's pointless.</p>

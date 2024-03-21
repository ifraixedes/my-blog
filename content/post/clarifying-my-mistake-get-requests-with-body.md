+++
title = "Clarifying My Mistake about GET Requests with Body"
description = "Having a body in a GET request doesn't fulfill the current HTTP/1.1 standard, take a look here to know why"
date = "2015-04-13T00:00:00+00:00"
tags = ["api", "http", "standard"]
categories = [
  "software engineering"
]

aliases = [
  "/post/clarifying-my-mistake-get-requests-with-body",
  "/2015/04/13/clarifying-my-mistake-get-requests-with-body/"
]
+++

Just the same week after I wrote the post <a href="{{< ref "post/thoughts-about-current-apis-implementations.md" >}}" target="_blank">Thoughts about current APIs implementations</a>, I had a discussion, with one of my colleagues, about if sending a body in a __GET Request__ fulfills with the current [HTTP/1.1 Standard](https://tools.ietf.org/html/rfc7231), which it's something that I commented on that post and I mentioned that it was the way to make requests with quite few parameters rather than use POST and not having a very large and ugly URL.

After that conversation, he made me see my mistake showing me that doing that it's no appropriated; here you can read why not and the possible solutions which fulfill the standard.


### Clarifying the Current HTTP/1.1 Standard

My colleague brought on the table a response sent by <a href="http://en.wikipedia.org/wiki/Roy_Fielding" target="_blank">Roy T. Fielding</a>, one of the principal authors of HTTP specification <a href="http://www.w3.org/Protocols/rfc2616/rfc2616.html" target="_blank">RFC2616 appeared in June 1999</a> which has been superseded by multiples RFCs <a href="https://tools.ietf.org/html/rfc7230" target="_blank">7230</a>, <a href="https://tools.ietf.org/html/rfc7231" target="_blank">7231</a>, <a href="https://tools.ietf.org/html/rfc7232" target="_blank">7232</a>, <a href="https://tools.ietf.org/html/rfc7233" target="_blank">7233</a>, <a href="https://tools.ietf.org/html/rfc7234" target="_blank">7234</a>, <a href="https://tools.ietf.org/html/rfc7235" target="_blank">7235</a>, <a href="https://tools.ietf.org/html/rfc7236" target="_blank">7236</a> and <a href="https://tools.ietf.org/html/rfc7237" target="_blank">7237</a>.

The <a href="https://groups.yahoo.com/neo/groups/rest-discuss/conversations/messages/9962" target="_blank" rel="nofollow">response</a> says

> Yes. In other words, any HTTP request message is allowed to contain
a message body, and thus must parse messages with that in mind.
Server semantics for GET, however, are restricted such that a body,
if any, has no semantic meaning to the request. The requirements
on parsing are separate from the requirements on method semantics.

> So, yes, you can send a body with GET, and no, it is never useful
to do so.

> This is part of the layered design of HTTP/1.1 that will become
clear again once the spec is partitioned (work in progress).

> ....Roy


Then my interpretations about __"GET request message has no defined semantics"__ which I extracted from the paragraph

> A payload within a GET request message has no defined semantics;
  sending a payload body on a GET request might cause some existing
  implementations to reject the request.

which you can find in <a href="https://tools.ietf.org/html/rfc7231#section-4.3.1" target="_blank">7231#4.3.1</a> __was wrong__.

As I mentioned, it's not wrong to send a payload (a.k.a body) in a GET request, however because it doesn't have semantics, means that the doing __two or more requests with the same URI with different payload should return the same response__, due that it'll be ignored, hence body will never change what is requested.


### What are the possibilities

If we stick to the standard there are basically two solutions

1. All the parameters are in the URL, considering part of the path and query parameters.
2. Use a POST request with the parameters, then the server must create a response which store in somehow a response back with a URI to access to it, allowing the client to make a GET request with that URI to retrieve the generated response, refer to <a href="http://stackoverflow.com/questions/4203686/how-can-i-deal-with-http-get-query-string-length-limitations-and-still-want-to-b" target="_blank">this Stack Overflow response for more details</a>.


As I also mentioned in my <a href="{{< ref "post/thoughts-about-current-apis-implementations.md" >}}" target="_blank">previous post</a>, if we add all the parameters in the URI then we get ugly URLs and perhaps, they aren't ideal, but in the case of REST APIs is something that doesn't matter due machines deal with them besides, obviously, we don't need to consider <a href="http://en.wikipedia.org/wiki/Web_crawler" target="_blank">web crawlers</a>, so they may only annoy us in our logs.

The second solutions, from my point of view, it's awkward and I'd only use as exception, when the amount of parameters generate a large URI which may be rejected, even though the current <a href="https://tools.ietf.org/html/rfc3986" target="_blank">URI standard, RFC 3986</a> and the <a href="https://tools.ietf.org/html/rfc7230#section-3.1.1" target="_blank">URI standard, RFC 3986</a> don't have any length limitation, they've found, in the real world, some limitations and the recommendations for senders and recipients is to support at least 8000 bytes which are quite a few to generate a large URI.

To see a very good clarification of it, considering real world URI's length limitations, read <a href="http://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers" target="_blank">this another Stack Overflow response</a>.


### Conclusion

The standard isn't quite clear and doesn't ban to send payload even though it means nothing.

The URIs are not that short that we can think and the web browsers have, so using GET with a long URLs should almost solve all the use cases, for those that fall outside the solution is the awkward two step request POST and GET, but if you want to stick to the standard don't use POST as single GET.


I hope that you've got to be clearer about this topic when you decide/have/must design an REST API, as I've got.


Have a nice week!

+++
title = "Behind the scene of a Gumtree Spoofed email"
description = "A real story about what there is behind the scene of a Gumtree spoofed email"
date = "2015-02-23T00:00:00+00:00"
tags = ["general public", "security", "email", "cybercrime"]
categories = [
  "Security"
]

aliases = [
  "/post/beghind-scene-gumtree-spoofed-email.md",
  "/2015/02/23/behind-scene-gumtree-spoofed-email/"
]
+++

In my previous post, I wrote about <a href="{{< ref "post/basic-hints-indentify-gumtree-spoofed-email.md" >}}" target="_blank">Basic Hints to Identify a Gumtree Spoofed email</a>, which is an introduction and the context of this post.

In this post I'm writing about what I found in the URL linked by that spoofed email.

This post, as the previous one related with this topic, doesn't bring any hacking technique, again, I'm not a hacker, I'm another common geek guy, hence don't expect to read a new kind of attack or hacking trend, just a little bit more technical stuff of what I just did for fun when I received that email.


## The raw content of the email

The first thing that I did, was taking a look to the raw format of the email, however as I already expected, nothing important was there, <a href="http://en.wikipedia.org/wiki/Sender_Policy_Framework" target="_blank">SPF</a> passed how it was expected because the email sender address is a gmail one, hence Google isn't going to ban itself if the email has been sent through their servers and nobody is going to use a gmail email address to send through other servers than the official ones because nobody gets any value of it; if somebody wants to do that, is because they want to send you an email with the domain address that they try to fake the original sender, in this case would be <a href="http://gumtree.com" target="_blank" rel="nofollow">Gumtree</a> not gmail.

Moreover in this case the content of the email was encoded in base64, so it was easy to see the domain name where the link is pointing, just hovering with the mouse and taking a look to the bottom of the browser window than decode the base64. Anyway, the content was the next

![phishing email raw format](https://s-media-cache-ak0.pinimg.com/originals/bc/ef/1c/bcef1c808165c0b85b7a0682c434b9d7.jpg)


## Following the links via `curl`

I got the link, nonetheless, I don't trust too much to visit the link straightaway without taking a look a little bit what behind it without a usual browser, just in case that they could take advantage of some security hole; although I'd probably browser it at the end, I prefer to have a roughly idea what scripts and URLs includes.

Then `curl` the URL

<img alt="curl the URL" src="http://zippy.gfycat.com/FeistyFrequentAndeancockoftherock.gif" class="graphic-medium graphic-medium-centre">


After that I realised that they used a URL in the middle to redirect to the proper site, <a href="http://curl.haxx.se/docs/manpage.html" target="_blank">`curl`</a> doesn't manage redirections by default, then I followed the redirection with `curl -L`.

<img alt="curl the URL with redirection" src="http://zippy.gfycat.com/ShamelessQuaintAntipodesgreenparakeet.gif" class="graphic-medium graphic-medium-centre">

As we can see, it's not only one redirection, they added a second one, this time a javascript one.

Why did they do all of these? I don't know the right answer, I haven't built it ;P, however I guess that they did for these reasons:

1. The link's URL is to avoid to be marked as spam in gmail, having a URL that the path (last part of the URL) is `images` helps to sidetrack the spam filters.
2. That URL response with a <a href="http://en.wikipedia.org/wiki/HTTP_301" target="_blank">301 HTTP status code (Moved Permanently)</a> adds another barrier for <a href="http://en.wikipedia.org/wiki/Web_crawler" target="_blank">crawler</a> which may follow the link, some of them don't follow those redirections then, more difficult to get there and find out that it's faking Gumtree.
3. Following the 301 redirection, the content of the page is just a simple chunk of javascript which redirect the browser when it gets execute; they added this new one to be harder to follow by crawlers, the most of them don't execute javascript, therefore the probability to detect the scam automatically drops down a lot.

Then, I followed the javascript redirection, expecting that would be the last one, because the URL revealed the Gumtree domain name on it, hence what is the point to another one when there isn't anything to hide behind it?

Due that, I decided to `curl` the URL and filter the content (with <a href="http://en.wikipedia.org/wiki/Grep" target="_blank">`grep`</a>) that could harm, however it's not the only one, but finding out `css`, `js` links, `<script>` tags and `src` attributes will show what in the page may be dangerous with no much effort.

<img alt="curl and filter the content" src="http://zippy.gfycat.com/FinishedWebbedAdouri.gif" class="graphic-medium graphic-medium-centre">


Great, just only some css and one image, moreover they are linked to Gumtree domain (sa.gumtree.com), then if we trust in Gumtree, we know that they shouldn't contain any malicious stuff.


#### Visit the web site with a browser

After all of those steps, I was ready to browse the URL with my browser (last official <a href="http://www.google.com/chrome/" target="_blank" rel="nofollow">Chrome</a> version), however, I always prefer to do in <a href="https://support.google.com/chrome/answer/95464?hl=en-GB"  target="_blank">incognito mode</a> and disable javascript execution (just in case) and cache to avoid to leave sh** in my computer.

<img alt="disabling javascript and cache and browsing the URL" src="http://zippy.gfycat.com/FirmYearlyKitten.gif" class="graphic-medium graphic-medium-centre">

As we can see the web site try to simulate Gumtree login form; have you got the point of the URL? they added Gumtree domain name into it to mislead the visitor and try to get him/her to believe that he/she is in a Gumtree web page, but clearly it's at the end in the <a href="http://en.wikipedia.org/wiki/Query_string" target="_blank">query string part of the URL</a> so it doesn't apply to the domain name.

Obviously, I wasn't going to introduce my login details there as you must not too; I took a look to the HTML code to see what is the URL that collects the form data and take a simple look to see if there was something weird there, however, remember that we haven't seen any javascript in page so probably the form shouldn't have any javascript validation.

<img alt="taking a look to the HTML code of the form" src="http://media.giphy.com/media/ytwDCK1o9L7KmGqefm/giphy.gif" class="graphic-medium graphic-medium-centre">

I could press the button to go to that URL, however I preferred to load the URL straightaway and see how good they built it.


<img alt="loading the URL from straightaway than submitting it" src="http://media.giphy.com/media/5yLgocc3WHwbwoTn7Us/giphy.gif" class="graphic-medium graphic-medium-centre">


Clearly, no quality at all, they didn't implement any check, the URL could be loaded by <a href="http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods" target="_blank">GET than POST</a> and the Gumtree domain in the URL is just a visual misleading.

We see that the page after that it's the interesting one for them, they want to collect our credit/debit card details, basically from where they make money after all their effort; I applied the same technique than previously and I loaded the URL straightaway, I'm feeling a little bit safe with it, but I don't have any strong technical argument to do it.

And that's it, they should get that they want, then they show you the expected message and redirect you to Gumtree home page to avoid that the victim realises of their domain, just in case that he/she hasn't before, and lose the income.

<img alt="loading credit/debit card details URL form straightaway" src="http://media.giphy.com/media/5yLgociQbt93scFPyPC/giphy.gif" class="graphic-medium graphic-medium-centre">

<img alt="the redirection to Gumtree home page" src="http://media.giphy.com/media/5yLgocjcqYJrKdSpGqk/giphy.gif" class="graphic-medium graphic-medium-centre">


### Conclusion

Phishing emails are something that still existing; nowadays, I guess that less people get haunted with those however they probably get some, even though the rate should be low, to build a fake Gumtree one is easy and cheap in terms of look & feel and the place to store the data collected for those forms.

As an advise, check always the __entire URL__ of the browser and don't get distracted by a part of the URL and ask yourself to check thoroughly to see if you can see any strange thing before sending any credit/debit card information.

Be safe.

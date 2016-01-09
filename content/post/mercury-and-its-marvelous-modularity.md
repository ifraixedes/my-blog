+++
title = "Mercury & its marvelous modularity"
description = "After I took a look to Mercury README long time ago, I've got my hands dirty with it during a 24 hours Hackathon, so here you can read my thoughts"
date = "2015-06-30T11:00:00+00:00"
slug = "mercury-and-its-marvelous-modularity"
tags = ["frontend", "javascript", "virtual-dom", "modularity", "Hackathon"]
categories = [
  "Frameworks"
]

aliases = [
  "/post/mercury-and-its-marvelous-modularity",
  "/2015/06/30/mercury-and-its-marvelous-modularity/"
]
+++

Long time ago, I've discovered <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a>; at that moment, Virtual-DOM started to be hot due the increasing popularity of React, highly influenced by the world wide known Facebook, and I got curious to search if there was anything else which also take advantage of the Virtual-DOM mechanism.

As it isn't surprisingly, there were at that moment <a target="_blank" href="http://facebook.github.io/react/">React</a> and <a target="__blank_" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a> and also a bunch of others<sup>1</sup>.

__Disclaimer__: I'm not experienced in React, I've just read several post of it and things related with it and watched some videos, on the other hand, this post isn't about XXX is better than YYY, there're always some subjective aspects of choosing frameworks, tools, etc, as other important non-technical stuff, as the aim to achieve, which strongly influence on what to choose/use to do that what actually really matters, to achieve the goal.


## The Context

My first step into Mercury has only been to implement a __very tiny__ <a target="_blank" href="https://en.wikipedia.org/wiki/Single-page_application"> SPA (single web application)</a> for the  topic of the hackathon which I participated; __tiny__ because 24 hours it isn't much time and moreover when we were a two people team, with all the development load for one of them, in this case my self.

Anyway, the hackathon isn't important for the purpose of this post, it's only to put you in context about the first time which I stepped into <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a>.


## Modularity Matters

I'm into modularity, it's my choice and it doesn't to be the best choice, as every approach, philosophy or whatever else has its tradeoffs, so depending of the situation, personality, relations, influence and so on, you prefer one over another because you feel that you get less painful situations, solve better your problems, fulfill your requirements or basically you're more comfortable, but again, I'm as well into "the right tool for each case".

In <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a> modularity is something that __nobody can say that it's bullshit marketing__, just take a look to <a target="_blank" href="https://github.com/Raynos/mercury/blob/649410edca7236c5b99ec45f5b139c01528022d8/index.js#L13L66" rel="nofollow">the main module file</a>.

Basically, Mercury, at the time of this blog, is <a target="_blank" href="https://github.com/Raynos/mercury/blob/649410edca7236c5b99ec45f5b139c01528022d8/index.js" rel="nofollow">just a javascript file of 126 LOC</a> which basically export a bunch of other tiny modules (each one exactly does one thing).

Its modularity has been one of the main points to actually want to spend my __spare time__ hacking with it in front of any other one which isn't pure modular.


## How to Start

<a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a> doesn't have a shiny website, a thorough documentation or bunch of people messing around with it, so you have to start in the ___hard way___.

There're <a target="_blank" href="https://github.com/Raynos/mercury/tree/master/examples/" rel="nofollow">several examples in the same repository</a>, however no tons of documentation, so you basically must read the code.

I think that the best way to start, it's reading the simple easiest implemented examples, which are only a few lines of code, so they're not that hard to having a overall understanding how it works.

After that or at the same time, you can check some of the <a target="_blank" href="https://github.com/Raynos/mercury/tree/master/docs" rel="nofollow">docs</a>, not all of them, because there're some which are advance topics that you shouldn't use for your first step into; however if you have time, having a quick read doesn't hurt; in my case so far I skipped the docs in "modules" subdirectory.

After that, you should start to implement a tiny application, probably bigger that a "Hello World", integrating some functionality as pull some data from a server and displaying in a table or a structured HTML than inside of a DOM element as `p`, to get the chance to taste the Virtual-DOM and probably <a target="_blank" href="https://github.com/Matt-Esch/virtual-dom#example---creating-a-vtree-using-virtual-hyperscript" rel="nofollow">Virtual-Hyperscript</a>.

During your implementation, you'll probably check the code of the examples and the docs several times; you may also try some changes that you may feel that it'd be nice if they work because they allow to organise and structure better your code, however if something stop to work, you'll have to spend time finding what is wrong and why actually it doesn't work, nonetheless I believe that it's the best way to learn because you'll discover things that they don't work and if you're curious, as I'm, you'll try to understand why they don't work than solving it with no clue why it's started to work again.


## Advantages of its Modularity

As I've mentioned above, Mercury isn't a popular framework as many other are, you probably think to skipped due the lack of tons of documentation, shiny website and all this kind of stuff.

I think that even though all those stuff are important, I feel that I haven't needed them, I could jump into it and used only with the examples and docs and checking its code and submodules, which are exactly where the meat is, to have an overall understanding in how to use it and its internals.

Being at that level of modularity allows to browser the code and find very easily each module which does actually one thing, so you can dig more on each modules when you feel that you want to know more about the internals, without getting lost as I've felt several times with big code bases, which are not composed by several modules and in case that they're, the most of the times, they don't reach this level due that the modules have been build for the purpose of the code base hence they lose the concept of module when they aren't thought to be used out of the main code base.


## My Footprint

As I've mentioned before, the implementation that I've done with <a target="_blank" href="https://github.com/Raynos/mercury">Mercury</a> was for a topic of a hackathon, because I always want to have some reward of my time, in this case at the end, only, has been the learning one.

The application isn't fancy at all, in terms of any thing, I had to do all the implementation, server, a little bit the styling and the frontend in Mercury, so I had to learn it as well.

I got the first issues that everybody may have which are how you organise your code and make it more modular to avoid a massive large scripts, I did some splitting and no a lot of more for two reasons:

1. We had to rush up to to be able to present.
2. At that moment, I didn't know how to split more, however after the hackathon has finished, I had a second read to the docs and I had a better understanding of some parts which in my first read I couldn't get them, hence now I've got them and I've known how to do it ;D

If you're interested you can see the code in this <a target="_blank" href="https://github.com/ifraixedes/hack-destination-london-2015" rel="nofollow">repository</a>, however, bear in mind that the examples in <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a> repository may be more useful.

## Conclusion

Modularity is something very important for me at the moment to choose a framework, because it allow me to understand the internals in less time. I believe that knowing the internals is king to solve future problems, do crazy stuff or increase the performance when you need it.

Now, I totally agree with the sentence which appear in the README, ___"A truly modular frontend framework"___, <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a> is about that, it isn't bullshit marketing.

Getting into it, allowed me to achieve my aim, which was having a better understanding about Virtual-DOM strategy and how it works than using a framework to build something and just only learn the public interface of the framework, because I don't see that much value on it if I'm not going to use for several projects, which isn't too bad if they have another aim, as making money, fulfill a business requirement and so on.

Hope that you find this post more interesting than another massive post about something related with one of the currently famous framework, being good or bad.


Have you tinkered with <a target="_blank" href="https://github.com/Raynos/mercury" rel="nofollow">Mercury</a>? please share you comments if you have.


----
<p class="definition"><sup>1</sup> I may have realised very late of React, it may have been there for a while, but at least people surrounding myself in the virtual/physical world weren't talking about it as they're talking at this time.<p>

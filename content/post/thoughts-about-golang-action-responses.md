+++
title = "Thoughts about Golang Action Responses"
description = "After reading about a different approach of Golang HTTP Handlers to have clean code, I've implemented a benchmark to about how it performs"
date = "2015-05-20"
tags = ["HTTP", "Golang", "Software Engineering", "Implementation"]
categories = [
  "Software Engineering"
]

aliases = [
  "/post/thoughts-about-golang-action-responses.md",
  "/2015/05/20/thoughts-about-golang-action-responses/"
]
+++

A few weeks ago one of my colleagues found a blog post about __<a href="http://openmymind.net/Go-action-responses/" target="_blank">a different approach to implement "actions" HTTP Handlers in Golang</a>__, which differs in how the most of the current Golang Web frameworks do which are heavily inspired in the interface offered by <a href="http://golang.org/pkg/net/http/#HandlerFunc" target="_blank">Golang HTTP official package</a>.

<a href="https://twitter.com/karlseguin" target="_blank" rel="nofollow">Karl Seguin</a> (the author of the post) commented that the approach gives a natural way to implement HTTP handlers returning response than passing them as an argument, something that works when the consumer not to need to have a full access control over the response, which are in most of the cases.

I've read this post and I've spend some time analysing it, because my colleagues asked to the rest of team to have a view and share our thoughts; at that moment I just replied him "[my first thoughts]({{< relref "#my-first-thoughts">}})" and "[what I don't like]({{< relref "#what-i-don-t-like" >}})" but I didn't have any substantial argument around my thoughts, so I've take part of my spare time to consolidate them and add some tangible metrics and write this post about it.


## My First Thoughts

My first impression was good, it __provides a way to have a less verbose, hence easy to read, implementations__ for the Handlers which deals with the logic of the request (called "actions"), it means that commong/global Handlers, usually called "middlewares", aren't contemplated.

The approach focus in "actions", making emphasis on increasing the readability of their implementations and common operations across them; it makes sense as a big web application has a lot of more "actions" than "middlewares" then developers spend the most of their time dealing with them, hence readability matters.

Here it's nothing new, basically I agree with Karl.


## What I don't like

To clarify before to comment what I don't like from the approach, I want to say that "I don't like" doesn't mean "I hate" or "the approach is discarded for those reasons", as usual there are tradeoffs on each approach and depending of the situation some tradeoffs ares less important than others and those define what approach to use for it.

The approach defines a struct "NormalResponse" which holds the content of a HTTP response, status code, headers and body; it helps to avoid any write on HTTP ResponseWriter until the "action" ends therefore it's possible to deal with errors without moving the responsibility to the developers who implement the "actions" which may drive to have inconsistent response, e.g. Send 200 status code when error happened.

"NormalResponse" struct is great for that, however __it introduces some overhead__; each action __create a new instance of a struct which is used to hold the values to send throw the wire__, hence those values must copied from "NormalResponse" instance to ResponseWriter; it means that for each request we introduce a new instance which must be garbage collected, therefore we introduce a performance penalty on our web application.


## How Performance Matters

As you may know, garbage collector isn't the best part of Golang and giving to it too much work could drive your application to screw up or at least not to get that response time that you desire; actually I've seen this possible tradeoff straightaway, due quite a few posts and library implementations which I've read where the authors are proud about zero or at least low number of allocations, so it nailed in my brain to be always aware.

Golang team is improving the garbage collector and they're going to release a <a href="http://llvm.cc/t/go-1-4-garbage-collection-plan-and-roadmap-golang-org/33" target="_blank">new improved version with the next release 1.5</a>, however it's outside of the topic of this post, the present is 1.4 release and a garbage-collected language doesn't mean you can ignore memory allocation issues.


As I've commented in other posts, like <a href="{{< ref "post/golang-performance-slices-structs-and-struct-pointers-analysing-metrics.md#conclusions" >}}" target="_blank">this</a> and <a href="{{< ref "post/performance-js-objects-created-different-ways.md#conclusions" >}}" target="_blank">this</a>, performance is important but __there are many other things which can be more important than performance__ in a lot of situations besides to be more important to find where are the bottle necks before removing the dust; hence this argument, as I've already commented above, doesn't discard the approach at all, but it's worth bearing in mind.


## Let's See some Numbers

Because Golang offers a bunch of good tools to benchmark and profile your implementations (I used them in these two post series <a href="{{< ref "post/golang-performance-slices-structs-and-struct-pointers-gathering-metrics.md" >}}" target="_blank">1</a> and <a href="{{< ref "post/golang-performance-slices-structs-and-struct-pointers-analysing-metrics.md" >}}" target="_blank">2</a>), I've made a simple "action" implementation, one using the "NormalResponse" approach and another with a standard handler to see if my thoughts were right or wrong.

The results that I've got are

```
spike ➜  go test -v -bench . -benchtime 10s -benchmem
testing: warning: no tests to run
PASS
BenchmarkResponse       10000000              1908 ns/op             728 B/op          6 allocs/op
BenchmarkStd            10000000              1357 ns/op             504 B/op          5 allocs/op
ok      spike   36.009s
```

As we can see, "NormalResponse" needs, roughly, 40% more nanoseconds per operation than a standard handler.

To compare the pprof graphs, cpu usage and memory consumption, I've executed the benchmark individually to have separated graphs; so I've executed those as previously but changing `-bench .` by `-bench Response` and `-bench Std`, for "NormalResponse" and "Standard Handler", respectively; both benchmarks run the same amount of times, as before "10000000".

for "NormalResponse", I've got (CPU first and memory second)

![NormalResponse CPU profile graph](https://s-media-cache-ak0.pinimg.com/originals/71/40/d1/7140d1c8694e9a2ec1780f232327a96c.png)

![NormalResponse Memory Allocation Space](https://s-media-cache-ak0.pinimg.com/originals/68/11/d7/6811d7cbe2d7fe949cad4621ba72183b.png)


And for "Standard Handler"

![Standard Handler CPU profile graph](https://s-media-cache-ak0.pinimg.com/originals/01/7f/4d/017f4dc3ab76f4c189e8bda8cbfc6705.png)

![Standard Handler Memory Allocation Space](https://s-media-cache-ak0.pinimg.com/originals/92/4d/13/924d13cb99fb7fdb6c6d54f7f3fe660a.png)


Taking a look to the CPU graphs, we can see that the "External Code" triggered by GC and System are the heavy load part of both executions so "ResponseNormal" doesn't introduce overhead in the garbage collection job, then my thoughts were wrong; however the __two phases response, "action" and `WriteTo` call makes the difference with a standard Handler and the bottle neck is the Map operations__.

In case of the memory consumption, we can see in the graphs how "NormalResponse" needs more memory, mostly, that difference is due `WriteTo` function and specifically the memory consumption are the operations on the Header type which is a Golang Map, again.

My first thoughts about the performance penalty that "NomralResponse" approach may haven't been right; it has a performance penalty, however it isn't due GC operations, it's about Map operations. Knowing where the bottle neck is we may introduce improvement, for example changing the type used by "NormalResponse" to store the headers to send and revaluate to see the gain.


## Conclusion

Performance matters however it isn't the only important thing to take care for all the implementation. Depending what you're implementing, the situation and requirements, performance may have less relevance than other stuff as readability, maintainability, etc.

Benchmarks are king to know how your implementations perform in front of others, nonetheless you should always __profile your code to see if you implementations can be improve without removing a big part of their advantages or at least you'll consolidate or refute your thoughts__ (it's my case here) why your implementation performs different.


By the way you can find the implementation of this benchmark in this <a href="https://gist.github.com/ifraixedes/056175e0cf312db88f0e" target="_blank">gist</a>


I hope that you've got something from here.

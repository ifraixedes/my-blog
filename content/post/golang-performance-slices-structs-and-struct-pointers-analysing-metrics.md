+++
title = "Profiling Golang Slices of Structs vs Pointers: Analysing Metrics"
description = "Analysing profiling metrics (cpu and memory) about Golang slices of structs versus pointers to structs with surprisingly results what performs better"
date = "2015-03-23T00:00:00+00:00"
tags = ["golang", "performance", "profiling", "analysis"]
categories = [
  "Performance"
]
+++

This is the second part of a two post series about the performance of Golang slices between those which contain structs and those which contain pointers to the same structs and how they are populated through different struct "constructor" and "converter" implementations, hence you need to read the <a href="{{< ref "post/golang-performance-slices-structs-and-struct-pointers-gathering-metrics.md" >}}" target=_blank>previous post</a> to understand this.

After gathering the profiling metrics from <a href="https://gist.github.com/ifraixedes/f11fba231ac8cb4cb1d4#file-structs_test-go" target="_blank">benchmarks</a> and <a href="https://gist.github.com/ifraixedes/f11fba231ac8cb4cb1d4#file-main-go" target=_blank>`main`</a> executions, I'm going to go through them to see how the different implemented approaches performs and hopefully to get some conclusions about them to have a better decision making for future Golang developments.


## Benchmarks

Executing the benchmark, `go test -v -bench . -benchtime 10s -benchmem -memprofile mem.out  -cpuprofile cpu.out`, we get an terminal output with some numbers besides of two files which contain CPU and memory profiling metrics.

The terminal output looks like this

![benchmarks results terminal output](https://s-media-cache-ak0.pinimg.com/originals/01/1c/6d/011c6dc4f064a6265d03e591aab1fe51.jpg)


With that, we already have what of them performs better and what is the worst, however, in my case I'm getting the opposite that initially I expected to find. I was expecting that avoiding memory copy between struct constructors which return pointers and slices which store pointers, the gain should be notorious better than copying values across the memory.

Let's, first, generate some graphs with the profiling data with pprof and see if we can be clearer about the results that we've got.

With `go tool pprof -png  -output  ~/Workspace/tmp/blog/cpu.png perf.test cpu.out` we get a graph with the CPU profiling metrics, which looks like

![benchmarks CPU graph](https://s-media-cache-ak0.pinimg.com/originals/12/56/6f/12566f2f1dbbd3d1ade1c80f5cc74c69.jpg)

And the allocated space with `go tool pprof -png -alloc_space -output  ~/Workspace/tmp/blog/mem-alloc-space.png perf.test mem.out`, which looks like

![benchmarks space allocation graph](https://s-media-cache-ak0.pinimg.com/originals/02/4c/64/024c6488103261951d5a82d2b12d51fb.jpg)

At first glance, I'm confused, so the __benchmarks' output shows different results than the profiling graphs__, which the best and worse approaches are switched around, however that's not really the case; __graphs show the accumulated of the benchmarks execution__, which run each approach a different number of times until they can measure an average time of each iteration; therefore the ___benchamkars output the average results per loop___, and graphs the whole accumulation of the whole execution. To confirm that, I've created spreadsheet where I've pulled the data from them, I've made the calculations from the accumulated data from the graphs to iterations besides to work out some comparisons between the approaches, to be clear the differences between them

<a href="http://goo.gl/jIkeZp" target=_blank>![spreadsheet benchmark comparison table](https://s-media-cache-ak0.pinimg.com/originals/5c/b7/78/5cb77848806bdd9ada1a0f5214d5994e.jpg)</a>

Click in the image to jump to the spreadsheet if you wish to see it.

Clearly, we can see the winner, with no pointers at all, we get the best CPU performance and the same memory performance that the slices of structs but using a constructor which returns pointer to a struct, which are 50% less memory usage than the other two approaches.


## Main

I've also implemented a `main` function, right?, maybe because I though that the benchmarks could show a different reality due the warm-up, therefore, let's analyse the `main` profiling data with each different approach.

Following the order that I've used in the spreadsheet, we have

The "struct" method CPU `go tool pprof -png  -output  ~/Workspace/tmp/blog/cpu-struct.png main cpu-struct.out`

![CPU profiling graph of "struct" method](https://s-media-cache-ak0.pinimg.com/originals/1d/22/bd/1d22bdb662f2b849bdf45bb277b6dd30.jpg)

and the "used memory space" `go tool pprof -png -inuse_space -output  ~/Workspace/tmp/blog/mem-inuse-space-struct.png main mem-struct.out`

![memory usage graph of "struct" method](https://s-media-cache-ak0.pinimg.com/originals/6a/ef/13/6aef13daa6b952e05fb9830e87e2aa97.jpg)

The "hybrid" method CPU `go tool pprof -png  -output  ~/Workspace/tmp/blog/cpu-hybrid.png main cpu-hybrid.out`

![CPU profiling graph of "hybrid" method](https://s-media-cache-ak0.pinimg.com/originals/ee/6f/47/ee6f47baac71aea04ceee502d66402b1.jpg)

and its memory usage `go tool pprof -png -inuse_space -output  ~/Workspace/tmp/blog/mem-inuse-space-hybrid.png main mem-hybrid.out`

![memory usage graph of "hybrid" method](https://s-media-cache-ak0.pinimg.com/originals/b4/dd/a5/b4dda54b0bcda8d88ed32df29476e96e.jpg)

The "pointer" one `go tool pprof -png  -output  ~/Workspace/tmp/blog/cpu-pointer.png main cpu-pointer.out`

![CPU profiling graph of "pointer" method](https://s-media-cache-ak0.pinimg.com/originals/79/cb/57/79cb5731eebdeb63068233256b601c6f.jpg)

and its memory usage `go tool pprof -png -inuse_space -output  ~/Workspace/tmp/blog/mem-inuse-space-pointer.png main mem-pointer.out`

![memory usage graph of "pointer" method](https://s-media-cache-ak0.pinimg.com/originals/b4/dd/a5/b4dda54b0bcda8d88ed32df29476e96e.jpg)

and the last one, the no-pointers, CPU `go tool pprof -png  -output  ~/Workspace/tmp/blog/cpu-no-pointers.png main cpu-no-pointers.out`

![CPU profiling graph of "no-pointers" method](https://s-media-cache-ak0.pinimg.com/originals/38/b3/77/38b3775ce311060259c07d054c583c45.jpg)

and memory usage `go tool pprof -png -inuse_space -output  ~/Workspace/tmp/blog/mem-inuse-space-no-pointers.png main mem-no-pointers.out`

![memory usage graph of "no-pointers" method](https://s-media-cache-ak0.pinimg.com/originals/6a/ef/13/6aef13daa6b952e05fb9830e87e2aa97.jpg)


## Analysis

We can discard the memory usage because the heap frames in `main` executions don't provide a consistent information when the slice stores structs or pointers; in the other hand, we've got the same results for the CPU performance between `main` and benchmark executions for all the approaches. I've also used pprof command line to list top 10 and list the functions, however I haven't found anything useful or something that we cannot see in the graphs, so I haven't included here to avoid fuzz.

__Surprisingly the fastest implementation is the one that doesn't involve pointers at all__, because the used constructor creates structs that are returned by copy, because it doesn't return a pointer, and the used slice stores structs rather than pointers, so another copy should be done in the assignation between the constructor return value and the position referenced by the slice's index.

I was expecting to find just the opposite, using pointers, hence just changing pointers addresses over copying values between memory spaces should be the fasters solution, however it's the slowest one.

Taking a look to the CPU graphs of the `main` execution with the two methods that use pointers, "pointer" and "hybrid", we can see how the `main` runtime and the function that `main` executes, don't even appear, because the CPU time is spent in two linked boxes named "System" and "External Code" and in the case of "pointer" method, we can also see two more related with the "gc" (garbage collector).

After this I can extract some conclusions and some doubts as well, so let's see then.


## Conclusions

__Making assumptions based in theories (references versus values) is a kind of making blind decisions__ when we don't exactly know how compilers and runtimes perform operations; hence __implementing benchmark and collecting profiling metrics of our developments are good practices__ that we should consider when the performance matters or whenever we're worry about it.

Golang has native packages and tools that allows to achieve them in not than painful way as other languages may be, besides it promotes them as a good development practice.

On the other hand, __this proof of concept doesn't mean that the right method to implement in every scenario__ is not to use pointers (addresses) and return and assign copies every where, due to this two fair facts:

1. The structs used here are very small structs, one of them only contains one `uint64` and the other one, two `uint32`, so the memory copy can be very optimised for those types or for small size structs, however it's something that I don't really know and they're only my thoughts.

2. The implementation is a process that runs and ends, which I consider that it can be different for a process which runs for long time (in theory infintely) and it has to deal with different loads, as a service or server could be; those services may need to optimise memory space sacrificing the CPU usage or vice versa, but again, those are my thoughts and each scenario must be assessed individually.


If you want to see a more complex implementation example and its performance analysis take a look to this <a href="http://blog.golang.org/profiling-go-programs" target=_blank>post in Golang blog</a>



As I said, this is a proof of concept, it's not any experience in running something in production or the steps followed to find bottle necks or performance improvements, furthermore I don't have years of experience with Golang and I'm not a performance analysis expert; I may be wrong in some of the metrics gathering, analysis or conclusions, and I will appreciate that if you realise about those or you have a better explanation or you thing that you can contribute in something, please write a comment and share them with others.


I've hope that you've got something interesting from here.

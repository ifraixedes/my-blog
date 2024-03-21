+++
date = "2016-03-15T19:30:46Z"
description = "A brief script, the materials and related links of my talk about Profiling Golang Programs on March of 2016 at Golang BCN group"
title = "Talk at Golang BCN about Profiling Go Programs"
social_image = "https://s-media-cache-ak0.pinimg.com/originals/f9/b7/a8/f9b7a8695ca571e8ec7b97dd2cfa72a9.png"
tags = ["golang", "performance"]
categories = [
  "software development"
]
+++

This post has two main purposes:

* Have a kind of script for my talk, at {{<ext-link "Golang BCN" "http://golangbcn.org/">}}, for the topic mentioned in the title of this post
* The place where the attendees and followers of Golang BCN can access to the material used in my talk

My talk doesn't have any slide, I think that's better to spend my time mentioning the code to run and what I do to use the profiler with the basic implementation which I've created for this purpose.

## Get Ready

To follow me you have first to make sure that you have installed in your machine:

* Golang 1.6; probably an early close version would work too, however I used that one because it was the last stable release at the time that I prepared the talk.
* {{<ext-link "Graphviz" "http://www.graphviz.org/">}} which is required to be able to generate `svg` with `pprof` (The tool offered by Golang out of the box to profile your programs)

NOTE that I used a linux machine to run it; I haven't done anything which doesn't look cross platform, but bear in mind that you may have issues run it on other operative system

Get the code, cloning this {{<ext-link "gist" "https://gist.github.com/ifraixedes/fd8b585c3da6dacb6385">}} in your `GOPATH`

## Basic implementation examples

To give this introductory talk to Golan profiler, I've implemented 4 flavors of a {{<ext-link "Power of 2 function" "https://en.wikipedia.org/wiki/Power_of_two">}} and 4 flavors of function which returns {{<ext-link "Fibonacci numbers" "https://en.wikipedia.org/wiki/Fibonacci_number">}}

The implementation have a test for each function, to make sure that they work as expected, some benchmarks which will use to profile the 8 mentioned functions and a basic server with 2 endpoints to show how `net/http/pprof` package expose profile information on an http server just only importing it.

I've also added a `Makefile` as helper to run the most of the commands which I need to profile the benchmarks and the basic http server.

## Profiling benchmarks

To profile benchmarks, we need to run the benchmarks with some flags to get profile metrics data files; we also have to get the compiled binary, because, thereafter, we can match the profile metrics with the function names and the lines of the source code where those metrics refer.

So we do something like `go test -bench . -benchmem -memprofile mem.out -cpuprofile cpu.out`

With the previous command, run the benchmarks and output a memory and cpu profile data; the binary is also generated with an auto-generated name, pass `-o mybin` to give it the name that you would like.

With the output generated in our terminal, we already know which flavor function (on each functions set) is the fastest. However, having the profiling data we can know from each function, which are the cost of their internal operations; for this simple functions isn't worth as they only have a few instructions, however in large programs where there are quite a few nested function calls, we can see which are the cost of each call and deepen in those ones which are the most costly and know where the bottlenecks are to see if we can change the implementation for more optimized ones.

The profiling data can be analyzed using `pprof` tool, executing something like `go tool pprof mybin cpu.out`.

When we run pprof in that way, we get into a "repl" where we can execute commands to read the the different information which profile data file contains and see some of them next to the line of the implementation which they refer

Another way, which probably is the most practical, is to generate a flow graph of the function calls with the cost of each (or the most costly if there are too many) function call; we can get that executing something like `go tool pprof -svg -output cpu.svg mybin cpu.out`

And we are ready to open `cpu.svg` and see where are our bottlenecks.

## Profiling a service

A service is an application that, ideally, will run forever, exposing an interface which other applications (clients) can request to perform the exposed operations. Due they behavior, they have to offer an interface to get the profile data or they have to export it on a defined intervals of time.

Golang has the package `{{<ext-link "net/http/pprof" "https://golang.org/pkg/net/http/pprof/">}}` which allows to expose, automatically, a few HTTP endpoints to collect the profile data, if the service has an http server, otherwise we can create one HTTP only for that purpose.

The exposed endpoints are listed under `/debug/pprof/` path and to collect the profile data we execute `pprof` pointing to the specific path, for example

* cpu: `go tool pprof http://localhost:8000/debug/pprof/profile`
* heap: `go tool pprof http://localhost:8000/debug/pprof/heap`
* goroutine block: `go tool pprof http://localhost:8000/debug/pprof/block`

Each of those calls generate an output profile data file as we has got from the benchmark and with those you can use `pprof` in the same way to analyze the data.


This is all for now!

Thanks for reading it

---
Related links

* The main reference about profiling Go programs posted in the Golang official Blog: {{<ext-link "Profiling Go Programs" "http://blog.golang.org/profiling-go-programs">}}
* I wrote a couple of post in my early days in Golang; I used the profiler to drop my curiosity about the performance of values and pointers passed as a parameter, so you can read other place how to use the profiler, {{<ext-link "gathering the metrics" "http://blog.fraixed.es/post/golang-performance-slices-structs-and-struct-pointers-gathering-metrics/">}} and {{<ext-link "analyzing them" "http://blog.fraixed.es/post/golang-performance-slices-structs-and-struct-pointers-analysing-metrics/">}}

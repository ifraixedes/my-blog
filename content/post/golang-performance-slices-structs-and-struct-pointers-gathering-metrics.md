+++
title = "Profiling Golang Slices of Structs vs Pointers: Gathering Metrics"
description = "Gathering profiling metrics (cpu and memory) about Golang slices of structs versus pointers to structs with surprisingly results what performs better"
date = "2015-03-16T11:00:00+00:00"
tags = ["golang", "performance", "profiling"]
categories = [
  "Performance"
]
section = "programming"
+++

After the post that I wrote last week about <a href="{{< ref "post/golang-slices-structs-or-pointers-to-structs-dilemma.md" >}} target="_blank">Golang slices</a>, and what type of slices to use when we use "constructor" functions, I've got a huge curiosity to know the difference, in performance wise, what are the differences of using slices which contain literally structs versus the ones which contain pointers to the same types of structs, and how it can be affected depending of which of them is used.

The possible combinations are, having a "constructor" function which given a struct of type A, returns a struct of type B, which can be a struct (copied retruned) or pointer (reference returned) and storing the B structs into slices of B structs (copied on each assignation) or pointers of B structs (referenced in each assignation); I know that this explanation is a mess, however go ahead to be clear what I mean.

To know about it, I've written a silly specific Golang example which will allows us to gather some __benchmarking and profiling metrics__; the motivation is due by how easy can be implemented and analised through the tool set that Golang natively brings; if  you aren't aware about them, take a look to <a href="http://golang.org/pkg/testing/#hdr-Benchmarks" target="_blank">Benchmark function in testing package</a> and <a href="http://golang.org/pkg/runtime/pprof/" target="_blank">CPU and memory profiling can be done with runtime/pprof package</a>.

In this post, I'll show the implementation and the process to gather performance metrics, and in the following post, we'll analyse them.


## The Implementation

I've taken as base, the same example that I wrote last week about slices and I've added all the different cases to compare.

Let's take a look to the main parts of the implementation to be in context when we analyse the results.

### The Types

The types are the same, of my previous post, plus one more, however I've renamed one to hopefully, avoid confusion between them in the results.

{{< highlight go >}}
type fromStruct struct {
	id uint64
}

type toStruct struct {
	upper uint32
	lower uint32
}

type fromStructSlice []fromStruct
type toStructSlice []toStruct
type toStructSliceP []*toStruct
{{< /highlight >}}

### The Struct Types Constructors

I've created two constructors, one that I'd already implemented in my pervious post, which returns a reference to the struct created inside, and one that I've added here, which returns a copy of the struct (which should perform worse, but we'll see exactly what, in the analysis part).

{{< highlight go >}}
func newToStruct(num uint64) *toStruct {
	return &toStruct{
		upper: uint32(num >> 32),
		lower: uint32(num),
	}
}

func toAStruct(num uint64) toStruct {
	return toStruct{
		upper: uint32(num >> 32),
		lower: uint32(num),
	}
}
{{< /highlight >}}

### The Slices Converters

For the slices converters, I've had to work a little bit more, because we have more combinations; again, one of them has been copied from my previous post and renamed, too.

{{< highlight go >}}
func fromStructSliceToStructSlice(fss fromStructSlice) (tss toStructSlice) {
	tss = make(toStructSlice, len(fss))

	for i, fs := range fss {
		tss[i] = *newToStruct(fs.id)
	}

	return tss
}

func fromStructSliceToStructSliceTmpP(fss fromStructSlice) (tss toStructSlice) {
	tss = make(toStructSlice, len(fss))

	var ts *toStruct
	for i, fs := range fss {
		ts = newToStruct(fs.id)
		tss[i] = *ts
	}

	return tss
}

func fromStructSliceToStructSliceP(fss fromStructSlice) (tss toStructSliceP) {
	tss = make(toStructSliceP, len(fss))

	for i, fs := range fss {
		tss[i] = newToStruct(fs.id)
	}

	return tss
}

func fromStructSliceToStructSliceNoPointers(fss fromStructSlice) (tss toStructSlice) {
	tss = make(toStructSlice, len(fss))

	for i, fs := range fss {
		tss[i] = toAStruct(fs.id)
	}

	return tss
}
{{< /highlight >}}

Be aware, that I've added one, `fromStructSliceToStructSliceTmpP` which is almost the same than `fromStructSliceToStructSlice`, however it doesn't do the assignation to the slice straightaway and uses a local variable; the reason is, because I've thought that having an indirection, perhaps, the compiler doesn't optimise the assignation that it may do in the other one, detecting the returned pointer which the pointed content is going to be copied in another part of memory, then why cannot it be a little bit smart and put the values into the memory part hold by the slice? OK, I've know that it is a paranoia, but adding that case was not too much work and we'll have one case more to compare even though it is a awkward implementation.


### The Benchmarks

I've written a benchmark for each slices converter and also tests for them and the constructors, however I'm not going to add the tests heres because I've written them to test the implementations and they don't provide anything interesting for the purpose of this post series, which is the performance.

{{< highlight go >}}

func BenchmarkFromStructSliceToStructSlice(b *testing.B) {
	fss := generateFromStructSlice(numElems)
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		fromStructSliceToStructSlice(fss)
	}
}

func BenchmarkFromStructSliceToStructSliceTmpP(b *testing.B) {
	fss := generateFromStructSlice(numElems)
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		fromStructSliceToStructSliceTmpP(fss)
	}
}

func BenchmarkFromStructSliceToStructSliceP(b *testing.B) {
	fss := generateFromStructSlice(numElems)
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		fromStructSliceToStructSliceP(fss)
	}
}

func BenchmarkFromStructSliceToStructSliceNotPointers(b *testing.B) {
	fss := generateFromStructSlice(numElems)
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		fromStructSliceToStructSliceNoPointers(fss)
	}
}

{{< /highlight >}}

You can see nothing fancy in them as I've kept them simple.

### The Main

`main` wasn't required to implement if the package isn't named `main`; actually I named it different beforehand, nonetheless later, I thought that it was not too bad, to implement a main function which allows to execute each case individually and output the CPU and Heap to files to be able to analyse them with `pprof`, and to have the results of a normal execution than only the benchmarks ones. Therefore I've renamed the package to "main" and I've implemented it.

{{< highlight go >}}
package main

import (
	"flag"
	"fmt"
	"log"
	"os"
	"runtime/pprof"
)

const numElems uint32 = 1 << 24

var transformMethod string

func init() {
	flag.StringVar(&transformMethod, "method", "", "the method to execute to transform values from one slice type to another: struct, pointer, hybrid or no-pointers")
}

func profileMem(suffix string) {
	f, err := os.Create(fmt.Sprintf("mem-%s.out", suffix))
	if err != nil {
		log.Fatal(err)
	}
	pprof.WriteHeapProfile(f)
	f.Close()
}

func main() {
	flag.Parse()
	fss := generateFromStructSlice(numElems)

	switch transformMethod {
	case "struct":
		f, err := os.Create("cpu-struct.out")
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()

		res := fromStructSliceToStructSlice(fss)
		profileMem("struct")
		log.Printf("%d", len(res))
	case "pointer":
		f, err := os.Create("cpu-pointer.out")
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()

		res := fromStructSliceToStructSliceP(fss)
		profileMem("pointer")
		log.Printf("%d", len(res))
	case "hybrid":
		f, err := os.Create("cpu-hybrid.out")
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()

		res := fromStructSliceToStructSliceTmpP(fss)
		profileMem("hybrid")
		log.Printf("%d", len(res))
	case "no-pointers":
		f, err := os.Create("cpu-no-pointers.out")
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()

		res := fromStructSliceToStructSliceNoPointers(fss)
		profileMem("no-pointers")
		log.Printf("%d", len(res))
	default:
		log.Fatal("invalid method")
	}
}

{{< /highlight >}}


## The Execution

The execution of the benchmarks is as easy as executing the tests through `go test` command but with one flag.

```
go test -v -bench . -benchtime 10s -benchmem -memprofile mem.out  -cpuprofile cpu.out .
```

The only required parameter to execute them, is `-bench`, which return the time spent in each iteration and the number of iterations that have been run as samples to figure the result.

In my case, I've also added the `-benchtime 10s` to run them 10 seconds than the default time, which is 1 second, `-benchmen` to get the memory allocation statistics and `-memprofile` and `-cpuprofile` to output the results to files be able to analyse them afterwards, with _pprof_.

You can read about the available `go test` flags, <a href="http://golang.org/cmd/go/#hdr-Description_of_testing_flags" target="_blank">http://golang.org/pkg/runtime/pprof/</a>


On the other hand, I've built "main" rather than only running it with `go run`, because to analyse the output data with pprof, the binary is required, hence I've built it with `go build main.go structs.go`.

To execute main, it's just `./main -method {which-one}` where `{which-one}` must be `struct`, `pointer`, `hybrid` or `no-pointers` which are the values that `main` checks for the only available flag, `-method`, although you may probably know about it from the above `main` implementation. Each method, output two filenames, "mem-xxx.out" and "cpu-xxx.out", where "xxx" are the values of the `-method` flag, to avoid to override a previous execution with different method.

All these outputs files are the files that we'll use to analyse the profiler data, through pprof, to see how the performance is affected by the different approaches.


## Conclusion

Golang has natively support to build benchmarks and packages to profile the CPU and Heap, so those tasks are not tedious to use as may be with another language.

Here I've used them to get profiling metrics of different ways to achieve the same, on the post [Profiling Golang Slices of Structs vs Pointers: Analyzing Metrics]{{<ref "post/golang-performance-slices-structs-and-struct-pointers-analysing-metrics.md">}} you can read how I analyze the outputs of the benchmarks and how I use pprof to know the % of CPU used by the internal function calls, so don't miss to have a look onto it.

By the way, __all the code is available in <a href="http://goo.gl/tC60Yl" target="_blank">this gist</a>__


I hope that it has been interesting.

+++
title = "The Slices of Structs or Slices of Pointers to Structs Dilemma"
description = "In Golang slices aren't  arrays of variable size, they're  structs, due that fact it can cause confusion about to store in it struct or pointers to structs"
date = "2015-03-09T11:00:00+00:00"
tags = ["golang", "conventions", "idioms"]
categories = [
  "Software Development"
]

aliases = [
  "/post/golang-slices-structs-or-pointers-to-structs-dilemma.md",
  "/2015/03/09/golang-slices-structs-or-pointers-to-structs-dilemma/"
]
+++

<a href="http://golang.org/" target="_blank">Golang</a> is a <a href="http://golang.org/doc/faq#ancestors" target="_blank">C family language</a> which has brought me, again, the ability to use memory references through pointers, even though with some significant differences: <a href="http://golang.org/doc/faq#no_pointer_arithmetic" target="_blank">no pointers arithmetic</a> and <a href="http://golang.org/doc/faq#garbage_collection" target="_blank">garbage collector</a>.

Its similarities with C, drove me, in the beginning, to some confusions about using pointers or values in one of its native internal powerful types, <a href="http://blog.golang.org/slices" target="_blank">slices</a>, here I'm going to talk about those.

### The context

Some months ago, I've started to read a little bit [Go documentation](http://golang.org/doc/effective_go.html), visit its <a href="http://tour.golang.org/" target="_blank">tour</a> and tinker it with some small implementations, just to get use with the syntax, types and quirks.

The last five months I've been increasing the number of hours to spend with it; following some resources which provide me good resources as videos, blog posts and marvelous repositories and at the same time to have the chance to use that theoretical knowledge in my professional environment.

Quite a few times a more active activity than consuming good resources, has been reading code, making small refactorings or adding small features and reviewing code; however that code has been generated for us (my own team members) and everyone are embracing Go roughly for the same period of time, even though some of them worked in the first implementation of one of the main company services, therefore the experience on hands on it is slightly different.


With this is small introductions to read and write lines of code inside of a small team with not that much experience in Go, allowed us to have the chance to discuss easily things as "why A and not B" when we've read something that we've seen that it's odd, messy, awkward or basically because some of us (myself included) don't understand why the author wrote it. Even though the author has reasons for for it, we have raised some doubts about __code organisation__, how to write more __comprehensible and clear code__, how __to use Go idioms__, what can __perform better__ and so on.


### The concerns about slices of structs

In the first instance, due the __nature of the <a href="http://blog.golang.org/go-slices-usage-and-internals" target="_blank">slices</a>__ is easy to think that it doesn't matter if you store into them, structs or pointers to structs, because slices are structures "which point to their elements", and it they are passed as a parameter to a function or returned from it, there wouldn't be any difference in terms of performance,  because their elements are not copied due they contain a reference to the memory space which the elements are alocated.

Nonetheless it isn't very strange to have, or find, a __function which creates a new type struct__ to wrap all the operations needed to initialised it than have to do them manually each time that one has to be created. Basically it's very usual to have one function that create a structure when it's not a straightforward struct where some of there values are initialised, worked out, composed or converted from other types or they have methods defined; several native Go packages use this approach for example <a href="http://golang.org/pkg/net/http/#NewRequest" target="_blank" rel="nofollow">Request</a>, moreover it's part of the <a href="http://golang.org/doc/effective_go.html#package-names" target="_blank">"constructors convention"</a>.

We probably agree that those __"constructors" clearly perform better returning  a pointer to the struct than the itself struct__; returning those by value doesn't make sense because they'd be copied and those copies don't provide any benefit because they have been created inside of the function hence any other reference to them exist at that moment, at least if nothing crazy has been implemented, as passing a pointer which will be pointed to it moreover to return it and we'd like to keep their independent then to return a copy would offer that independence, however let's forget the crazy scenarios that the most of the time they don't probably make sense.

Considering those __"constructors" functions__, if we need to __create slices__ of those types structs then, the most obvious thing is to have slices of pointers, because when we have to create room for the slice, we'll only allocate memory for those pointers, otherwise we'd allocate memory for those structs and thereafter, when the constructor return a new one, the values should be copied from the pointer returned by the constructor to the struct allocated in the slice.

Here there is a simple snipped of code with this concept

{{< highlight go >}}
type fromStruct struct {
	id uint64
}

type toStruct struct {
	upper uint32
	lower uint32
}

// Return pointer to a struct to avoid to
//return a copy of it which has been created inside
func newToStruct(num uint64) *toStruct {
	return &toStruct{
		upper: uint32(num >> 32),
		lower: uint32(num),
	}
}

// Return a slice of pointer to a struct because they structs are created
// by a constructor function then added to the slice, and having pointers
// we avoid to copy their values, each time that one is added
func fromStructSliceToStructSlice(fss []fromStruct) (tss []*toStruct) {
	tss = make([]*toStruct, len(fss))

	for i, fs := range fss {
		tss[i] = newToStruct(fs.id)
	}

	return tss
}
{{< /highlight >}}

With __this prospect everything is quite clear__ and the __answer is to use slice of pointers to structs__, nonetheless I didn't find that conclusion straightaway when I didn't have that perspective and when I read several lines of code where slices of pointers to struct are passed to functions, because my first thought was that who wrote them, used pointers to struct to avoid to pass copies of those to functions, due as easy is as to get confused between [slices and arrays](http://blog.golang.org/go-slices-usage-and-internals) which is <a href="http://golang.org/doc/effective_go.html#allocation_make" target="blank">mentioned in the documentation</a> but when it's read in one go is easy not to assimilate the concept in the first read; by the way, when I read and that though raised, I asked about it, however I didn't get the previously exposed prospect and the answer was that probably were for that confusing reason what slices really are and at that moment to write that implementation, the concept was unknown.

__Reading some open source code__, I've discover a __simple and elegant way to avoid to see stars between the square brackets and the type name__; defining a type for slices of pointer to structs free your function declaration of several brackets and start. Applying this in the code snipped that I've written perviously, it looks as

{{< highlight go >}}
type fromStruct struct {
	id uint64
}

type toStruct struct {
	upper uint32
	lower uint32
}

type fromStructSlice []fromStruct
type toStructSlice []*toStruct

func newToStruct(num uint64) *toStruct {
	return &toStruct{
		upper: uint32(num >> 32),
		lower: uint32(num),
	}
}

func fromStructSliceToStructSlice(fss fromStructSlice) (tss toStructSlice) {
	tss = make(toStructSlice, len(fss))

	for i, fs := range fss {
		tss[i] = newToStruct(fs.id)
	}

	return tss
}
{{< /highlight >}}

If you want to see an example of one open source project, you can see it, in one of the <a href="https://github.com/spf13/hugo/blob/91d16fbba09d3552af7b5327797dd2b5d9861b35/hugolib/pagination.go#L31" target="_blank" rel="nofollow">hugo packages</a>.


### Conclusion

In Go, whenever you have a type struct, which is created through a "constructor" function, which will be used into slices, use slices of pointers to structs and define types for those if you don't like to see lots of stars and brackets in your function declarations.


Hope that you read before you have had to figure yourself.

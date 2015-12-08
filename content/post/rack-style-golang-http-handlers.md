+++
title = "Rack Style Golang HTTP Handlers"
description = "After I looked a different approach to write Golang HTTP Handlers for common actions I've come up with a new approach to write those in a Ruby Rack style"
date = "2015-06-03T11:00:00+00:00"
tags = ["HTTP", "Golang", "Software Engineering", "Implementation", "Ruby Rack"]
categories = [
  "Software Engineering"
]

aliases = [
  "/post/rack-style-golang-http-handlers",
  "/2015/06/03/rack-style-golang-http-handlers/"
]
+++

A couple of weeks ago <a href="{{< ref "post/thoughts-about-golang-action-responses.md" >}}" target="_blank">I published a blog post with some thoughts</a> about  __<a href="http://openmymind.net/Go-action-responses/" target="_blank">"actions" HTTP Handlers in Golang</a>__, which specifies an opinionated way to have a handler function signature to define common responses for HTTP applications.

In my blog post, I also showed the results of a tiny benchmark test about how the approach performs over a <a href="http://golang.org/pkg/net/http/#Handler" target="_blank">Standard HTTP Handlers</a>

After that and getting the firsts steps of a web service implementation with the approach, with a few changes that we had to do to afford several needs, I've been thinking how to get another approach which keeps the befits of the original one, besides to have a central point for error management, to be able to perform global tasks as Logging, common responses for those errors avoiding to leak system information, and so on.


## What I'm Talking About

After the title and the introduction probably the most of you are wondering "what exactly I'm talking about", "a new revolutionary way?"", actually no, this isn't the heaven ;D

Let's first start with the firs sentence that we can read in __<a href="http://rack.github.io/" target="_blank">Ruby Rack's web site</a>__: ___To use Rack, provide an 'app': an object that responds to the call method, taking the environment hash as a parameter, and returning an Array with three elements"___

The interesting words, for the purpose of the approach which I'm talking about are: __returning and Array with three elements__

Great, that sucks in Golang, do you know?; Golang is type checking, so you cannot mix & match different types in an <a href="https://blog.golang.org/slices" target="_blank">Array or Slice</a>, sure nonetheless it has a very nice feature which I really love, "__<a href="https://golang.org/doc/effective_go.html#multiple-returns" blank="_blank">Functions can return Multiple Values</a>__".


Bearing in mind that we can have the same concept than Rack, even better we get the type checking guarantee, so great to get some programming errors at compiling time.

That was great to me to go through, however I've had a thought later on, which I feel as it's more in the Go's way, so I've come up with the idea to add a fourth value to return, which is an __<a href="http://golang.org/pkg/builtin/#error" target="_blank">Error type</a>__.


## Some Code doesn't Hurt

Probably, I could skip to write any code in this post, so it may not needed; every one who's read the previous section, has understood the concept, at least if they have read <a href="{{< ref "post/thoughts-about-golang-action-responses.md" >}}" target="_blank">my previous blog post</a> and <a href="http://openmymind.net/Go-action-responses/" target="_blank">"Go Actions" blog post</a>.

However I think that adding a few line of code doesn't hurt and I guess that it helps to digest the concept moreover if you've landed into it with no clue about the mentioned posts; therefore there it goes<sup>1<sup>


{{< highlight go >}}
type RackHandler func(*http.Request) (status int, headers [][2]string, body []byte, err error)

func Wrap(action RackHandler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		s, hs, b, err := action(r)
		if err != nil {
			// Do something with the error. e.g. logging
			// Using a switch statement to perform different actions depending status code
			// If error happens, body should contain only user friendly information which
			// doesn't leak any system information
			switch s {
			case http.StatusInternalServerError:
				fallthrough
			default:
				hs = nil
				s = http.StatusInternalServerError
				b = nil
			}
		}

		if hs != nil {
			header := w.Header()
			for _, h := range hs {
				header.Add(h[0], h[1])
			}
		}

		w.WriteHeader(s)

		if b != nil {
			w.Write(b)
		}
	})
}
{{< /highlight >}}


Pretty much the concept is that simple; of course you'll probably have to add more stuff to afford all the functionalities that a serious HTTP application needs, as the previous snippet is only a bare implementation.

Let's see, as well, how an "actions" looks

{{< highlight go >}}
var responseCounter = 0

func HelloBuddy(req *http.Request) (int, [][2]string, []byte, error) {
	responseCounter++
	op := responseCounter % 2

	switch op {
	case 0:
		return 200,
			[][2]string{{"Content-Type", "text/html"}},
			[]byte(`<!DOCTYPE html>
			<html>
				<head><title>Hello</title></head>
				<body><h1>Hello Buddy!</h1></body>
			</html>
		`), nil

	default:
		return http.StatusInternalServerError, nil, nil, errors.New("Odd request")
	}
}
{{< /highlight >}}

Sorry, for the silly functionality of the "action", I hope that you understand that it is not about what it does, is how it looks besides that it matches with the <a href="https://gist.github.com/ifraixedes/056175e0cf312db88f0e#file-response-go-L71" target="_blank">same one implemented for the benchmark test of my previous post</a>.


## A Few Comments about The Approach

The implementation, from my point of view, it looks a little bit simpler, because it doesn't need a struct to wrap the response, nonetheless it's my personal opinion.

The place where I see a benefit is the `Wrap` function, because it can check if the "action" returns an error and do general tasks, as I've already mentioned, "logging", returning defined errors which don't have any important information of the infrastructure behind the HTTP application, and so on.

Having a returned error value we can use __<a href="http://blog.golang.org/errors-are-values" target="_blank">defined values</a>__ and/or __<a href="http://blog.golang.org/error-handling-and-go" target="_blank">specific types</a>__ to execute different actions and having homogeneous error responses for defined issues.

On the other hand, an inconvenient, for somebody, can be, in the return statement of the "actions", they may look rare and/or verbose, but you know, there is always some tradeoffs.


## Conclusion

<a href="http://openmymind.net/Go-action-responses/" target="_blank">"Go Actions"</a> looks as a nice approach, however I've felt a little bit uncomfourtable<sup>2</sup> having a struct which wraps the response to send back to the client.

On the other hand, I've prefer error values and types to be able to have a central point to manage the errors and send back the response to the client than having to create different functions to return different error responses, nonetheless that can be achieved in "Go Actions" just adding an second error type return value to the "action" function signature.


"Go Action" approach is used, with some tweaks, in the implementation of a real HTTP Application which we're building, however I'm feeling that we're getting some pains that the "Rack" approach could solve or a least making those less painful, however it hasn't been used under the same scenario, so far, hence I'm not really sure if it can be better or worse.


Do you have any thought about these approaches or a new ones that aren't popular? I'll appreciate if you have some, you share them though the comments.


----
<p class="definition"><sup>1</sup> code of this post is released under <a href="http://www.wtfpl.net/txt/copying/" target="_blank">WTFPL</a></p>
<p class="definition"><sup>2</sup> It's my own personal opinion</p>

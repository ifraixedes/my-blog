+++
title = "3 Little Things in NodeJS which Help When You Have to Refactor"
description = "Software development involves refactoring in it own nature, applying some small things can help you to have an easy and painless refactoring process"
date = "2015-02-02T00:00:00+00:00"
tags = ["refactoring", "NodeJS", "javascript", "expressjs", "prototyping", "soft dev"]
categories = [
  "Software Development"
]
aliases = [
  "/posts/3-little-things-nodejs-help-to-refactor.md",
  "/2015/02/02/3_little_things_nodejs_help_to_refactor/"
]
+++

We can say that in software development the concept of <a href="http://martinfowler.com/books/refactoring.html" target="_blank">refactoring</a> is present all the time; it seems that is in its nature.

We refactor code for several purposes, having a list with all of them, can be tricky, however here there are some:

* We are building a prototype and we iterate over it to add, remove or change features.
* We must clean code which was created in some point in a rush or it was created in the prototype cycle.
* We must fix a bug.
* We must upgrade dependencies which have changed the exposed API.
* We must fix a performance issue.
* We've come up with a new way/idea to get a cleaner or optimised implementation.

In my case, we are doing code refactoring almost everyday, even though, in my case, when I'm working in a new feature over a existent code base, sometimes I find some existent part which may have a little refactoring, and then I do, or if I'm working in a totally new implementation, I usually start from a roughly idea, which I <a href="http://www.extremeprogramming.org/rules/spike.html" target="_blank">create a spike</a> and then I refactor iteratively.

In this post, I'm going to mention 3 little things that I've learnt of refactoring NodeJS code bases, personal and professional ones, therefore I've learnt some of them from my own mistakes.


## Extracting `options` object parameter at the beginning of the function

One of the common implementation pattern that I use a lot in NodeJS is creating scripts that expose a function which receives only one parameter, an options object, e.g.

{{< highlight js >}}
  // @param {Object} opts
  module.exports = function (opts) {
    // Do whatever

    return {
      tell: tell
    };
  };

  function tell(somebody, message) {
    return 'Hey ' + somebody + ', Ivan wants to tell you: ' + message;
  }
{{< /highlight >}}

This pattern help me to prototype implementations because I can add more functionalities which require some options without changing the expose signature of the function, nonetheless don't take me how it has to be used everywhere or keep it as the pattern to use to expose any API.

Ok, the pattern is great and if the function only has a few lines it is easy to take it to use in another prototype which may reuse it, however if the implementation is larger in number of lines or complex, for example it performs asynchronous operations (if you use callbacks then I would add, to the function of the example, a second parameter to pass the callback) then, knowing what options the function requires is painful, if they are accessed any where in the function.

One solution for that is to add a block comment on top of the function declaration with all the options parameters required and optional, nonetheless it isn't in my likes, because if you're prototyping, it'll require to keep update when one parameter is added, removed or swapped from required to optional or vice versa, therefore it is prone to be outdated, hence useless.

A second solution, which is in my preferences, is extracting the options parameters just at the beginning of the function, just after you make all the validations of the required options or having a function with that purpose, which returns an opts object with the required values if opts pass the needed validations; you may already know what I mean, nonetheless let's see an example:

{{< highlight js >}}
// @param {Object} opts
module.exports = function (opts) {
  // Having a helper function for the validations is
  // worthwhile when there are several options to check
  // values and/or types
  if (!opts) {
    throw new Error('opts is required');
  }

  var greeting = opts.greeting || 'Hey';
  var teller = opts.teller || 'Ivan';

  return {
    tell: tell.bind(null, greeting, teller)
  };
};

function tell(greeting, teller, somebody, message) {
  return greeting + ' ' + somebody + ', ' + teller + ' wants to tell you: ' + message;
}
{{< /highlight >}}

If the function is used internally and the options values are set by our implementation, validations are very basic, then extracting is clearer; hence as a rule of thumb keep the extraction in only one place because, with a quick glance, you can figure out in the future what option parameters the function needs.

On the other hand, the good news is that ECMAScript 6 (ES6) will bring a destructuring feature, which allow to do something like this but compacted in the function signature, besides other cool stuff. You can read about it in the awesome Alex Rauschmayer blog post called <a href="http://www.2ality.com/2015/01/es6-destructuring.html" target="_blank">Destructuring and parameter handling in ECMAScript 6</a> to know deeply about ES6 destructuring and <a href="http://www.2ality.com/2015/01/es6-destructuring.html#simulating_named_parameters_in_javascript" href="_blank">Simulating Named Parameters in JavaScript</a> section to see the exposed case.


## Environment variables can be evil

Environment variables can be very useful to parametrise your applications and change the application behaviour however using too many or not to add them into documentation easy to find, can be a nightmare if they are accessed across all the code base.

However if they are used with awareness, only using a few and very identified then, they avoid to have a burden configuration file which has to be versioned for different environments.

In NodeJS, there is the famous NODE_ENV environment variable and it's fine to define change the behaviour of your application or library (if it is justified), however I've found that not always is good to rely on it due that some npm packages use it to operate differently in production than development or testing environments.

The cases that I've found they are better to use other variable than NODE_ENV to specify the environment is when you have something that only has to run exclusively in production, for example, you  would like to set NODE_ENV to production to do load testing of your application, but you don't want to send errors message to a central service that you use to collect them, because obviously you're testing it in a controlled environment and don't want to pollute your productions logs.

Anyway if your implementations need environment variables, I would pull all of them out in one place and would spread them as a property of an options object parameter, similar what I exposed in the previous section; unifying to one place where the application take them, eases to see in quick view what environment variables your application needs and what default values are assigned if they could be not specified, then know easily what values to set in other environments when the default values aren't appropriated.


## Use absolute URL computed into variables

This is a recommendation bound to <a href="http://expressjs.com/4x/api.html" target="_blank">express 4.x</a>, specifically to <a href="http://expressjs.com/4x/api.html#router" target="_blank">Router</a> feature which is available since version 4, nonetheless it may apply for others Web frameworks in a similar way.

Router is one of the awesome features that express 4 has brought, it allows to create mini-applications which specify their routes, middlewares included; they offer the possibility to reuse your implemented routes in different applications, obviously if they are quite generic and you developed them bearing in mind that autonomy.

However, I created some of them bearing in mind that, but I missed one thing. My routes render views rather than to send JSON, however it could apply to JSON if you have to send URLs to follow, and they need to link one to another and vice versa.

Because route can be mounted by other <a href="http://expressjs.com/4x/api.html#app.use" target="_blank">Routers and a App</a> you cannot hardcode an absolute URL, then I thought use relative URLs.

For example, having this two routes in a router:

{{< highlight js >}}
  router.get('/user/:id', function (req, res, next) { ... });
  router.get('/group/:id', function (req, res, next) { ... })
{{< /highlight >}}

Then each route view has a link to each other, e.g.

* user view

{{< highlight html >}}
  <a href="group/10">NodeJS developers</a>
{{< /highlight >}}

* group view

{{< highlight html >}}
  <a href="user/0">Group Administrator</a>
{{< /highlight >}}

Then I got in troubles because the links don't work, because if you are, for example, in a users view, your browser url could be something `http://communi.ty/user/10`, then when you press the link to the group "NodeJS developers" your browser will request `http://communi.ty/user/group/10`, then the URL does not match with the "group" route of the Router.

In front of it, you can think to use absolute URLs, however if you use that Router in another application or your mount it in another path, then they won't work. Perhaps, you decide that the views are not portable and they have to be implemented on each application, well... that make sense, views are very bound to the application and they cannot probably be reused but you still have the issue that if you mount the Router under another path in the same application, because you are prototyping then you could realise that a new path fits better, then you would have to update your views without haven't ported your router to another application; even worse, in the JSON case that I mentioned above, using relative URLs will require to update the Router, then it wouldn't be reusable.

The solutions is easy, always compute the links with their absolute URL in the route and pass them to the render as variables, and likewise when you send JSON.

You can get the absolute URL even though you are in a Router's route with <a href="http://expressjs.com/4x/api.html#req.originalUrl" target="_blank">`req.originalUrl`</a> and then you can compute the URLs to the other routes; in this example, stripping "/group/*" and "/user/*" and after appending whatever is needed in each case would be enough to get the expected result.


## Conclusion

As software developers, we are refactoring all the time, software always can be better or it has to change for any reason; sometimes we have to prototype and move the prototypes to a product as soon as possible hence we don't have time enough to spend thinking all the cases that our applications will face, even we'd have it, we'd probably fail, for that reason I guess that agile methodologies appeared; then in front of of this situation, we can craft software as better as we can and apply tiny things that we learn throughout our experience to make those refactoring less painful.

Thanks for reading it.

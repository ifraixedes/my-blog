+++
title = "Private Members on ES6 Classes with Proxies"
description = "ES6 classes doesn't have any official mechanism to define private members, however using proxies we can get an approach of having them"
date = "2015-05-06T11:00:00+00:00"
tags = ["iojs", "javascript", "es6", "hack"]
categories = [
  "Software Development"
]

aliases = [
  "/post/private-members-es6-classes-with-proxies",
  "2015/05/06/private-members-es6-classes-with-proxies/"
]
+++

ES6 is getting close to its official release, the <a href="http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts" target="_blank">last Revision, no. 38, is the last Final Draft</a>, however <a href="https://kangax.github.io/compat-table/es6/" target="_blank">Browsers, Runtimes and Compilers/Polyfills</a> haven't implemented all the features that the specification defines.

In this post I'm going to go through over a hacky idea about to create objects with __ES6 Classes with private members without using closures as in ES5__, which is possible as you probably know, nonetheless that topic is very well explained in articles, blog posts, etc., so no point to write another blog post to talk about it, again.


## ES6 Classes

As you may already know, ES6 brings the concept of Object Oriented Programming Classes to Javascript.

__ES6 Classes__ allows to create classes without having to specify functions and its prototype property; the syntax and everything that you need to know can be found in a very thorough post named <a href="http://www.2ality.com/2015/02/es6-classes-final.html" target="_blank">"Classes in ECMAScript 6 (final semantics)"</a> that <a href="http://rauschma.de/" target="_blank">Dr. Axel Rauschmayer</a> has written and where he also concluded <a href="http://www.2ality.com/2015/02/es6-classes-final.html#does_javascript_need_classes%3F" target="_blank">a few clear benefits about ES6 Classes</a>.

If you've already read his blog post or you've also known from any other source, ES6 classes doesn't have property scope as many, if not all, <a href="http://en.wikipedia.org/wiki/Object-oriented_programming" target="_blank">Object Oriented Programming</a> languages have, then you still need the <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures" target="_blank">closures workaround to define private properties in objects</a>, at least if you don't use any of the working approaches that I show bellow.


## ES6 Proxies

 __Proxies is a ES6 mechanism that allows to register traps over an object__, so they allows to specify custom behaviour for fundamental object operations (assignment, iteration, property lookup, etc).

 Proxies is a less mature ES6 feature than classes, at least for the environment that I've used in this post which you'll read below.

 Because you can find several resources to figure out what ES6 proxies allow to do, I'm not going to tell their details here; if you're lazy to look for them, <a href="http://rauschma.de/" target="_blank" rel="nofollow">Axel</a> has also written a <a href="http://www.2ality.com/2014/12/es6-proxies.html" target="_blank">very thorough post about them</a>.


## The hack

After the brief introduction about the two ES6 features that I've used to create a hacky idea to define private members in objects, I'm going to walk through what I've used to get it works and the different approaches that I've tried until I found the one that I felt quite satisfied.


### The used environment

My first thought was to use <a href="http://babeljs.io/" target="_blank" rel="nofollow">Babel</a> to be able to transpile ES6 to ES5 and be able to use this hack everywhere, however it couldn't be because, according to Babel, <a href="http://babeljs.io/docs/learn-es6/#proxies" target="_blank">no way to transpile ES6 proxies to ES5</a>.

So my alternative has been to use <a href="https://iojs.org/" target="_blank" rel="nofollow">iojs</a> with <a href="https://www.npmjs.com/package/harmony-reflect" target="_blank">harmony-reflect</a> module.

Therefore if you want to run any piece of code of this post, then install `iojs-1.8.1` (version which I've used so I really know for sure that it works), install harmony-reflect <a href="https://www.npmjs.com" target="_blank" rel="nofollow">npm</a> module and execute iojs with `--harmony` and `--harmony_proxies` flags (`iojs --harmony --harmony_proxies the-script.js`).


### Class declaration

I've created a very simple class to show the proof of concept of this hack, nothing clever here, however it's useful to get a better understanding wha I'll show you below.

{{< highlight js >}}
'use strict';

require('harmony-reflect');

// Usual class
class Person {
  constructor(name) {
    this._name = name;
  }
  get name() {
    return this._name;
  }
  set name(name) {
    this._name = name;
  }
  greeting(person) {
    return `hi ${person.name}`;
  }
}

// Create an object from an usual class
let ivanPerson = new Person('Ivan');

// Call the getter of _name
console.log(`name getter from a Person instance: ${ivanPerson.name}`);
// _name property is accessible because JS classes doesn't have access scope
console.log(`_name property from a Person instance ${ivanPerson._name}`);
 {{< /highlight >}}

 Very clear what I've coded, moreover with the comments I don't think that it needs any clarification; note the `require('harmony-reflect')` import, otherwise the following lines of code won't work.


### The possible approaches

Before going to see some of the approaches that I've come up, you must know that I've chosen as a convention that any property (values or method) whose name starts with `_` is private, even though my proof of concept only show the access to a property value with a defined getter, it also works for properties values without getters and property methods.


#### Proxify the Base Class (not working solution)

I already knew that it wasn't probably going to work, because if you know a little bit how Classes works or you inspect a very slightly them, you'll see that __a ES6 Class it's an object with Function prototype__, besides, Proxies only trap the calls to the target object, so wrapping a Class with a Proxy will protect the Class, not the instances create from them; however I gave a try because never hurt and you make sure that your thoughts are right.

{{< highlight js >}}
// Let's try to protect class object
let ProtectedPerson = new Proxy(Person, {
  get(target, name) {
    console.log('calling getter in Person wrapped by a proxy');
    if (name.startsWith('_')) {
      throw new Error('Accessing to a private property is not allowed');
    } else {
      return target[name];
    }
  }
});

// Let's create an instance of the class that we wrapped with a proxy
let ivanProtectedPerson = new ProtectedPerson('Ivan');

// Call the getter of _name
console.log(`name getter from a protected person instance: ${ivanProtectedPerson.name}`);
// _name still accessible due proxy wrapped class Person;
// a class is a Function with prototype object, proxy trap calls to the
// class itself hence the target is the class, not its instances
console.log(`_name property from a protected person instance: ${ivanProtectedPerson._name}`);
{{< /highlight >}}

Thoughts confirmed, `_name` on any instance of `Person` still accessible; what the Proxy traps, are the properties of `Person` class which is a instance of `Function`; you can test it yourself if you don't believe me.


#### Proxify the Class instances

The obvious solution is to define a proxy for each `Person` instance, and it works as a charm, let's see the example:

{{< highlight js >}}
// let's protect a base class instance
let ivanPersonProtected = new Proxy(ivanPerson, {
  get(target, name) {
    if (name.startsWith('_')) {
      throw new Error('Accessing to a private property is not allowed');
    } else {
      return target[name];
    }
  }
})

// Call the getter of _name
console.log(`name getter from a person instance which has been protected: ${ivanPersonProtected.name}`);
try {
  // _name property call gets trapped by the proxy
  console.log(`_name property from a person instance which has been protected: ${ivanPersonProtected._name}`);
} catch (e) {
  if (e.message === 'Accessing to a private property is not allowed') {
    console.log('Proxy did its job!!');
  } else {
    throw e;
  }
}
{{< /highlight >}}

The solution works and it's great, however it's repetitive having to create a proxy each time that an instance of `Person` is created; it's true that we can create a function which wrap the two operations and it'll solve the problem, however I would like to use Classes to create instances as expected, so using `new` operator, for that reason I squashed my brain a little bit more and I've found the next solution which works and I guess that it's the most interesting part of this post.


#### Proxify `this` instances

The idea has been based in the concept of how Javascript's constructors (i.e. Functions) behave.

Javascript's constructors  return `this` object if they don't have a return statement in its body (usual and expected the most of the times) otherwise they return the value of the `return` statement.

In ES6, Classes are Functions and constructors are allowed to have return statements and their behaviour, is totally the same than in ES5, according Axel, they have the same behaviour for [backwards compatibility](http://www.2ality.com/2015/02/es6-classes-final.html#safety_checks).

Bear in mind that concept, we can do the following:

{{< highlight js >}}
// However it's painful, having to wrap in a proxy every single instance of a class
// is a repeatable tiring task, so let's try to creata a class that protect its
// object instances

class PersonProtected {
  constructor(name) {
    this._name = name;
    // In es6 it also works as in es5: remember es6 class is nothing more than a Function
    // and `new` call the function defined by `constructor`;
    // in es5 works because when you call a `new` on a function the value retuned is the
    // value returned inside the function if it's an object otherwise returns `this`;
    // in es6 remains the same for backward compatibility
    return new Proxy(this, {
      get(target, name) {
        if (name.startsWith('_')) {
          throw new Error('Accessing to a private property is not allowed');
        } else {
          return target[name];
        }
      }
    });
  }
  get name() {
    return this._name;
  }
  set name(name) {
    this._name = name;
  }
  greeting(person) {
    return `hi ${person.name}`;
  }
}

let ivanPersonProtectedInstance = new PersonProtected('Ivan');

// Call the getter of _name
console.log(`name getter from a PersonProtected instance: ${ivanPersonProtectedInstance.name}`);
try {
  // _name property call gets trapped by the proxy
  console.log(`_name property from a PersonProtected instance: ${ivanPersonProtectedInstance._name}`);
} catch (e) {
  if (e.message === 'Accessing to a private property is not allowed') {
    console.log('Class which proxy `this` did its job!!');
  } else {
    throw e;
  }
}
{{< /highlight >}}

As you can see in the previous snippet of code, it's as simple as moving the Proxy call inside the constructor and pass `this` as a target and returns the returned object by the Proxy call, so it takes advantage of the strange behaviour of Javascript's constructors which has been kept in ES6 Classes.

It's clean, works and we can keep the Class instantiation syntax as it is, with the ability to define private properties just starting their names with `_`, convention that I've chosen.


### Conclusion

ES6 Classes don't have any mechanism to define private properties, however with the help of Proxies and the backwards compatibility kept with ES5 we can create a clean mechanism based in naming conventions to define private objects' properties.

Remind you that it's a hack, Proxies aren't even available in iojs, besides I have no clue the performance about this approach, so it isn't for production if you lover yourself.

All the code of the post, just in case that you'd like to check it out, is under this <a href="https://gist.github.com/ifraixedes/e9311748c961f1dbb93e" target="_blank">gist</a>


Do you have any other idea to get something similar with the features that ES6 brings? if you have, please share them in the comments.

+++
title = "Performance Execution on JS Objects Created in Different Ways"
description = "In JS (javascript) there are different approaches to create objects and each one performs different when the object properties values are read and write"
date = "2015-01-26T11:00:00+00:00"
tags = ["NodeJS", "iojs", "web browser", "javascript", "performance", "research", "soft dev"]
categories = [
  "Performance"
]

aliases = [
  "/post/performance-js-objects-created-different-ways",
  "/2015/02/09/performance-js-objects-created-different-ways/"
]
+++

In JS (javascript) there are different approaches to create objects which differ in somehow not only syntax but in small things that you can achieve.

My aim in this post is not to talk of those different approaches in the sense how they differ in terms of <a href="http://en.wikipedia.org/wiki/Information_hiding" target="_blank">encapsulation</a>, <a href="http://en.wikipedia.org/wiki/Computer_programming#Readability_of_source_code" target="_blank">code readability</a>, <a href="http://en.wikipedia.org/wiki/Programming_paradigm" target="_blank">programming paradigm</a> approach, best practices, likes and preferences; I think that there are enough posts, articles, resources, etc, with full details, well explained and good quality about this topic on Internet.

I'm going to show some **basic performance tests** that I've done with an object created with those different approaches; just to have fund and because I was curious about it.

## Test implementations

I've implemented a silly example of those different approaches and after I executed a performance test in different platforms (Browsers, <a href="http://nodejs.org/" target="_blank" rel="nofollow">Node</a> and <a href="https://iojs.org/" target="_blank" rel="nofollow">iojs</a>); let's see first the used object creation approaches implementations:

-	Usual Constructor Function

{{< highlight js >}}
function ContentPrototype() {
  this.text = null;
}

ContentPrototype.prototype.set = function (text) {
  if (typeof text !== 'string') {
      throw new Error('Content mus be a string');
  }

  this.text = text;
};

ContentPrototype.prototype.get = function (text) {
  return this.text;
};
{{< /highlight >}}

-	Usual Constructor Function **without** Object Prototype

{{< highlight js >}}
function ContentPrototypeNull() {
  this.text = null;
}

ContentPrototypeNull.prototype.__proto__ = null;
ContentPrototypeNull.prototype.set = function (text) {
  if (typeof text !== 'string') {
      throw new Error('Content mus be a string');
  }

  this.text = text;
};

ContentPrototypeNull.prototype.get = function (text) {
  return this.text;
};
{{< /highlight >}}

-	Constructor Function which uses `Object.defineProperty`

{{< highlight js >}}
function ContentProperty() {
  var text = null;

  Object.defineProperty(this, 'content', {
    set: function (c) {
      if (typeof c !== 'string') {
          throw new Error('Content mus be a string');
      }

      text = c;
    },
    get: function () {
      return text;
    }
  });
}
{{< /highlight >}}

-	Constructor Function which uses `Object.defineProperty` **without** Object Prototype

{{< highlight js >}}
function ContentPropertyNull() {
  var text = null;

  Object.defineProperty(this, 'content', {
    set: function (c) {
      if (typeof c !== 'string') {
          throw new Error('Content mus be a string');
      }

      text = c;
    },
    get: function () {
      return text;
    }
  });
}

ContentPropertyNull.prototype.__proto__ = null;
{{< /highlight >}}

-	Function which returns an Usual Object Literal

{{< highlight js >}}
function contentFunction() {
  var text = null;

  return {
      set: function (c) {
        if (typeof c !== 'string') {
            throw new Error('Content mus be a string');
        }

        text = c;
      },
      get: function () {
        return text;
      }
  };
}
{{< /highlight >}}

-	Function which returns an Object Literal **without** Object Prototype

{{< highlight js >}}
function contentFunctionNull() {
  var text = null;
  var obj = Object.create(null);

  obj.set = function (c) {
    if (typeof c !== 'string') {
      throw new Error('Content mus be a string');
    }

    text = c;
  };

  obj.get = function () {
    return text;
  };

  return obj;
}
{{< /highlight >}}

-	Function which returns an Object created by `Object.create` defining properties

{{< highlight js >}}
function contentFunctionObjCreate() {
  var text = null;

  return Object.create(Object, {
    content: {
      set: function (c) {
        if (typeof c !== 'string') {
          throw new Error('Content mus be a string');
        }

        text = c;
      },
      get: function () {
        return text;
      }
    }
  });
}
{{< /highlight >}}

-	Function which returns an Object created by `Object.create` defining properties **without** Object Prototype

{{< highlight js >}}
function contentFunctionObjCreateNull() {
  var text = null;

  return Object.create(null, {
    content: {
      set: function (c) {
        if (typeof c !== 'string') {
          throw new Error('Content mus be a string');
        }

        text = c;
      },
      get: function () {
        return text;
      }
    }
  });
}
{{< /highlight >}}

After I wrote those examples, I also wrote little test for each one to check that they have the same behaviour in both operations (read and write), however I haven't added here to avoid noise.

## Performance Execution Tests

Thereafter I've created two performance test scripts to run in Node and iojs in my laptop (8 cores):

1.	My own simple implementation which basically execute each operation a bunch of times in a loop, stores the individual values and calculates the time execution average; you can find it in this <a href="https://gist.github.com/ifraixedes/d9af03a3b67a8aba4a5c#file-simple-average-time-metrics-js" target="_blank" rel="nofollow">gist</a>
2.	Another one which uses <a href="http://benchmarkjs.com/" target="_blank" rel="nofollow">benchmark.js</a> with <a href="https://github.com/wadey/node-microtime" target="_blank" rel="nofollow">microtime</a> whose script you can find in this <a href="https://gist.github.com/ifraixedes/d9af03a3b67a8aba4a5c#file-benchmark-metics-js" target="_blank" rel="nofollow">gist</a>

In the case of the browsers, I've used <a href="http://jsperf.com/" target="_blank" rel="nofollow">jsperf</a>.

Then I've executed the performance execution test in the mentioned platforms to see how the objects, created with each different approach, perform in terms of writing and reading an object's property value.

### iojs

I used the last iojs version available so far, which is 1.1.0.

My own script returned these results

<img alt="iojs v1.1.0 simple average time" src="https://s-media-cache-ak0.pinimg.com/originals/65/d6/66/65d666c5bf4aa1063ba2b42e299ceba8.jpg" class="graphic-medium graphic-medium-centre">

Benchmarjs script returned these ones

<img alt="iojs v1.1.0 benchmark" src="https://s-media-cache-ak0.pinimg.com/originals/9a/76/18/9a7618386ca59477e6069f1b086f1be5.jpg" class="graphic-medium graphic-medium-centre">

### Node

I used the last production Node version available so far, which is 0.12.0, YES v0.12 has been finally released.

My own script returned these results

<img alt="node v0.12.0 simple average time" src="https://s-media-cache-ak0.pinimg.com/originals/f1/b1/9a/f1b19a2b89f0daf73ee83ebbcc3c5165.jpg" class="graphic-medium graphic-medium-centre">


Benchmarjs script returned these ones

<img alt="node v0.12.0 benchmark" src="https://s-media-cache-ak0.pinimg.com/originals/de/36/a1/de36a16e9216edaf3f6e4afaaafa0b81.jpg" class="graphic-medium graphic-medium-centre">

### Browser

In the browser you can find two tests, which are the same than the iojs and Node ones, but I had to split them in two, one to run the writing property operation and another for the reading one, to offer better results readability.

I've run each one in three browsers, Chrome, Firefox and Safari and two different versions in the case of Chrome, one of them the current Chrome Canary; however **I encourage you to run with your browsers** to have more samples of the same borwser/version and/or new ones from different browsers/versions.

The <a href="http://jsperf.com/proto-defineprop-obj-set/2" target="_blank">writing property value test</a> showed me the next chart which I took an screenshot

<img alt="browsers writing property jsperf chart" src="https://s-media-cache-ak0.pinimg.com/originals/84/97/7e/84977e5a4415963462f7aa98c72f9e54.jpg" class="graphic-medium graphic-medium-centre">

The <a href="http://jsperf.com/prototype-vs-defineproperty-vs-object-get/3" target="_blank">reading property value test</a> and its screenshot that I took

<img alt="browsers reading property jsperf chart" src="https://s-media-cache-ak0.pinimg.com/originals/d6/21/d2/d621d29ce5176a6a119d4b65abca3e84.jpg" class="graphic-medium graphic-medium-centre">

## My Analysis

Due that Node and iojs benchmark don't make more difference between running the benchmark test asynchronous and synchronous, I've just needed to take one of them in both cases to compare.

Having the comparison of writing and reading an object property value between them using benchmark script is not quite clear about the differences, more or less they perform similar and one is a little bit faster than other in some cases and vice versa.

If I have to asses them using **my own script**, I can see that overall, **Node performs around 5% faster than iojs**.

However both are consistent in what create object approaches perform faster in terms of writing and reading values.

We can see that Node and iojs, writes and reads perform better with objects whose properties have been defined using `Object.defineProperty` or defining them through `Object.create`; the approach that performs worse is any kind of constructors and in the middle we have the object literals; we cannot see much difference between inherit from Object prototype than without.

Having the **difference between the best approach and the worst one** we can get gains around **40% in writing** and **200% in reading**, so not bad to have thought.

In the case of the browsers, we've got that the performance between the different approaches is the same for writing and reading.

Between them, **Firefox and Safari perform very good with object literals**; Firefox does, in second place with constructors and so bad with objects created with `Object.create` and/or having the properties set by `Object.defineProperty` and Safari the opposite.

Surprisingly **Chrome performs equally with all of them less object literals**, and **Chrome Canary** increases those with constructors, maintains the same with object literals and **decrease so much with objects created by `Object.create`** which define their properties with it.

In the browser, you can see easily, thanks to the charts showed by <a href="http://jsperf.com/" target="_blank" rel="nofollow">jsperf</a>, the roughly percentage of gain between approaches and browsers.

## Conclusion

Having these result you can decide what approach to use in your implementations to boost the performance, however you may only get a benefit of it if you are writing and reading values on objects properties that you've created their implementation; nonetheless I only assessed CPU so, profiling the amount of memory spend with the different approaches when you have to create a lot of them, would also be important to keep the best balance.

Moreover in the time being the results are those, but mostly, browsers are updated fast and these results may change in the near future.

From my point of view, **it isn't worthwhile to focus in performance** and **forget <a href="http://en.wikipedia.org/wiki/Computer_programming#Readability_of_source_code" target="_blank">code readability</a>, <a href="http://en.wikipedia.org/wiki/Programming_paradigm" target="_blank">programming paradigm</a> and best practices** to keep the implementation simple and clear to ease the maintainability; hence after I've written it, I suggest to read these two rules of the Unix philosophy:

1. <a href="http://www.faqs.org/docs/artu/ch01s06.html#id2877610" target="_blank">Rule of Clarity: Clarity is better than cleverness</a>
2. <a href="http://www.faqs.org/docs/artu/ch01s06.html#rule_of_optimization" target="_blank">Rule of Optimization: Prototype before polishing. Get it working before you optimize it</a>

I hope that you've enjoyed.

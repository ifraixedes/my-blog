+++
title = "How Arrow Functions Perform on Node?"
date = "2016-02-18T19:18:37Z"
description = "NodeJS is offering, natively the arrow functions since while ago, but do you know how they perform versus usual functions? in this post you'll read how"
tags = ["performance", "nodejs", "javascript"]
categories = [
  "software development"
]
social_image = "https://s-media-cache-ak0.pinimg.com/originals/99/49/77/994977c48fde58ac674a2d05ba5a5efb.png"
+++

ES2015 Arrow functions are offered natively on NodeJS since while ago; they look as __a convenient way to write anonymous functions than using function expressions__, because they have a {{<ext-link "shorter syntax, but they also bind `this` value lexically" "https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions">}}, which is the thing that got my attention to do this short performance test to know if they run slower, faster or equally than function expressions.

As I mentioned in several posts, performance may be very important, but the most of the times, it's quite less important than other things, as having a clean, simple and easy to understand and maintain code base which may performs not that fast as another one without those properties, or other reasons as using implementations (a.k.a frameworks) which are more generic so they allow to build fast, with less resources and many people can start to work quick because are well know by a certain amount of them.

I always consider the things mentioned above, however I'm not lazy if I have to type a bit more to get better performance without compromising any other important property, so in this case I would say that if arrow functions perform slower than function expressions, I would follow using function expressions because I don't really mind to type the word `function`, moreover that I think that in some situations the line of code is more clear having the keyword `function` than the shorter syntax of the arrow functions, nonetheless that's out of the scope of this post.

## Performance test

Note, my purpose here is only __to analyze those situations where the anonymous functions used don't access to `this` pointer__, because the most of the times that I've seen replacing functions expressions by arrow functions are on those situations and my worries are if arrow functions perform slower due the fact that they lexically bind `this` pointer.

I'm not analyzing those ones which the functions access to the lexical scope where functions expression should do through `bind` or assigning lexical `this` to a variable defined in the lexical scope, which at least, {{<ext-link "a few years ago, `bind` didn't look as a good options from the performance point of view and we should see what happen now with the arrow functions" "http://stackoverflow.com/questions/17638305/why-is-bind-slower-than-a-closure">}}, however that's out of the main goal of this post.

To know the different running times between then I wrote a small script using {{<ext-link "Benchmark.js" "https://benchmarkjs.com/">}} with {{<ext-link "Microtime" "https://github.com/wadey/node-microtime">}} which looks like


{{<highlight js>}}
'use strict';

let benchmark = require('benchmark');

let obj = {
  execFe: function(a, b, c) {
    return (function (a, b, c) {
      return (a + b) * c;
    }(a, b, c));
  },
  execAf: function(a, b, c) {
    return ((a, b, c) => (a + b) * c)(a, b, c);
  }
};

module.exports = obj;

let suite = new benchmark.Suite;

suite.add('arrow function', function () {
  obj.execAf(5, 10, 60);
})
.add('normal function', function () {
  obj.execFe(5, 10, 60);
})
.on('complete', function () {
  this.forEach(b => {
    console.log(`${b.name} -> mean: ${b.stats.mean}`);
  });

  let b = this[0];
  let m = this[1];
  let diff = b.stats.mean - m.stats.mean;
  console.log(`${b.name} is ${(diff < 0) ? 'faster' : 'slower'} than ${m.name} by ${(diff < 0) ? -diff : diff} seconds`);
  console.log(`Fastest is: ${this.filter('fastest').map('name')}`);
})
.run({ async: false });

{{</highlight>}}

You can also find it on the {{<ext-link "gist which I've created for its purpose" "https://gist.github.com/ifraixedes/e697740cd43bec83cd37">}}

## Results

I ran it several times (with NodeJS v5.6.0) and I didn't find any performance run time difference between the arrow functions and the functions expressions. Each time that I executed the test I got different result, some of them the arrow function was faster and in others the function expressions, besides that the time difference between them was so tiny, so that's more the usual differences that you can find in benchmark libraries than a real fact.

Below you can see the result of two of the executions that I did

```
arrow function -> mean: 3.1797950413531386e-8
normal function -> mean: 3.236298102991821e-8
arrow function is faster than normal function by 5.650306163868237e-10 seconds
Fastest is: arrow function,normal function
```

```
arrow function -> mean: 3.478900530420292e-8
normal function -> mean: 3.4248199017909106e-8
arrow function is slower than normal function by 5.408062862938166e-10 seconds
Fastest is: normal function,arrow function
```

As you can see, the difference is negligible.

## Conclusions

Arrow functions and function expressions have the same performance execution, at least in Node.

Best!

---

I've updated this post on 23/03/2016 to clarify what I wanted to analyze related arrow functions

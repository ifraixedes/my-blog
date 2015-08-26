+++
title = "JS Test with Mocha, Chai & Sinon Driven by Example"
description = "Read about how to test in iojs/NodeJS with 100% coverage a simple AWS Lambda handler which third party dependencies with Mocha, Chai & Sinon"
date = "2015-04-06T11:00:00+00:00"
tags = ["testing", "iojs and NodeJS", "javascript", "soft dev", "best practices"]
categories = [
  "Software Development",
  "Best Practices"
]
+++

I've decided to write this post after I've helped a friend of mine, <a href="https://twitter.com/hyprstack" target="_blank" rel="nofollow">@hyprstack</a>, to write a 100% coverage test in one of his implementations using "spies" and "stubs".

As you probably well know __"testing"__ is one of the general software development __best practices__, they're highly recommended for any piece of code that has been create beyond a hack and whenever you're a little bit worry the consequences that your code can produce if it doesn't do what you expect.

Anyway, my friend was doing an implementation of AWS handler for <a href="http://aws.amazon.com/lambda/" target="_blank" rel="nofollow">AWS Lambda</a> which basically is triggered when a file it's uploaded to S3. The handler (function) has to download the file (if it has a supported file image extension), resize the image file in several defined sizes and upload those new files to S3.

It may not seem quite complicated, however the most of the operations are performed by third party libraries (<a href="https://npmjs.org" target="_blank" rel="nofollow">npm modules</a>), hence its test must be to "stub" no to rely on S3 as external service and avoid to mess around image files to simulate the resizing, therefore <a href="https://github.com/aheckmann/gm" target="_blank" rel="nofollow">gm (GraphicsMagick & ImageMagick) node module</a> must be stubbed as well.

__You can refer to this <a href="http://stackoverflow.com/questions/28881483/nodejs-testing-aws-with-mocha/28915395#28915395" target="_blank" rel="nofollow">Stack Overflow question</a> for more details__ about my friends' doubt.


## The execution environment

AWS Lambda is __naturally iojs/NodeJS asynchronous pattern__, basically everything here run in iojs/NodeJS.

As you may know the asynchronous pattern isn't easy to fully understand when you're starting with it, or at least, reaching the point where combining asynchronous callbacks to make it works together are not something that you beat in one fell swoop in the first days, and if you add an extra bit of abstraction by a workflow library, the things can be easily messed up, as you can see as example in that Stack Overflow question; however nothing to worry, I made similar mistakes when I was starting.

This is one point in favour to be more interested in testing your code as my friend is.


## Testing with Mocha, Chai and Sinon

<a href="http://mochajs.org/" target="_blank">__Mocha__</a> as test framework running, <a href="http://chaijs.com/" target="_blank">__Chai__ as assertion library and <a href="http://sinonjs.org/" target="_blank">Sinon</a> as test "spies", "stubs" and "mocks" are a __good combination to mix together__ if you need all those things.

I've used a lot Mocha and Chai, but Sinon, just a bit, due the most of things that I've been developing haven't need to stub a third party dependency to bypass them for similar reasons that I mentioned in the introduction of this post.

Stubs are useful, but they're not the easiest thing to assimilate at the first time that you use or read the documentation; again they can be easy to understand, but as the most the things, more the difficult when you have to use properly in a real scenario.


## The implementation

I'm not going to mention what the implementation does, I think that it's easier to take a look to a right implementation that I've created in this <a href="https://gist.github.com/ifraixedes/3330ce0edf9286234b04" target="_blank" rel="nofollow">gist</a> in case that you're more interested and have better understanding about the following part of the post, hence I just leave here, the pice of code that it can be some more difficult because the issue was still there with the first Stack Overflow reply

{{< highlight js >}}
async.waterfall([
    function download(next) {
      s3.getObject({
        Bucket: s3Bucket,
        Key: s3Key
      }, next);
    },
    function transform(response, next) {
      async.map(sizesConfigs, function (sizeConfig, mapNext) {
        gm(response.Body, s3Key)
        .resize(sizeConfig.width)
        .toBuffer(imageType, function (err, buffer) {
          if (err) {
            mapNext(err);
            return;
          }

          s3.putObject({
            Bucket: s3Bucket,
            Key: 'dst/' + sizeConfig.destinationPath + '/' + s3Key,
            Body: buffer,
            ContentType: 'image/' + imageType
          }, mapNext)
        });
      }, next);
    }
  ], function (err) {
    if (err) {
      console.error('Error processing image, details %s', err.message);
      context.done(err);
    } else {
      context.done(null, 'Successfully resized images bucket: ' + s3Bucket + ' / key: ' + s3Key);
    }
  });
{{< /highlight >}}

The first implementation and the second one provided in the first Stack Overflow reply, messed up with the <a href="https://github.com/caolan/async#waterfall" target="_blank" rel="nofollow">async.waterfall</a>; and this uses, as you can see, inside one of the waterfall functions <a href="https://github.com/caolan/async#map" target="_blank" rel="nofollow">async.map</a>; for a year an half ago, I'm not using async that much, I use [promises](http://en.wikipedia.org/wiki/Futures_and_promises), however I've tried to introduce the less changes as possible to make it works and clean up a little bit to make the implementation simpler to ease its readability.

By the way, my friend did another implementation in my first feed back and it solved the issue, however he was confused due the back and forth feedback, and he did another implementation without using async and using in his own, which I think that it's good to learn and understand how async works internally, but the implementation looks neater with `async` beyond to be closer to the original one.


## The test

I've decided to implement the test from scratch, it was not ideal for me to take the test that he and the Stack Overflow responser did and refactor to aim the 100% coverage.

I'm not going to expose here the test, you can see it, in the <a href="https://gist.github.com/ifraixedes/3330ce0edf9286234b04" target="_blank" rel="nofollow">gist</a>; I'm going to mention what I've learn about Sinon when you have to use for several third party dependencies, together with <a href="https://github.com/thlorenz/proxyquire" target="_blank" rel="nofollow">proxyquire</a> which I had never used before and it's good not to have to expose or inject third party modules in your modules and having the chance of still using them as usual when you want to stub them.

Whenever you are using this "testing stack" (Mocha, Chai and Sinon), __<a href="https://github.com/domenic/sinon-chai" target="_blank" rel="nofollow">sinon-chai</a> is king__, it simplifies and make the test assertions more readable when obviously you use both; my friend was already using it, so I haven't introduced it to the new test, but I've though that it's a good thing to mention.

If you read the test, you'll see that there're several stubs (and/or spies) that you must use to be able to test the handler. The tests cases which involve to test all the execution workflow require to test the calls (passsed parameters) and define at callback call for each; they drove me mad at the beginning and in some point I decided to extract the `transform` function in the `asyn.waterfall` chain to an individual script (a.k.a node module) to be able to create just only one "stub" for the part with the `proxyquire` help and write a second test to test only the `transform` module; however I've failed, I couldn't make `proxyquire` work with it, it was still executing the original function than the `stub` and at the end, I returned back to the current solution (all in one script/node module), however I've also thought the easy second solution, to add the `transform` function to AWS handler module, however I didn't like to have to export both in the same module and as you can see, I returned back to the first and current approach.

With the current approach, I've started to create all the "stubs" and as I thought at the beginning, it was going to be hard to add every stub with the parameters that each one should receive to test all the calls to `gm` module, moreover that a simple mistake (typo, etc.) was not going not to call the chain, making difficult to discover from where the error comes (implementation or test stub and which one), moreover to have to put a lot of expectations in the wrong place `begin` Mocha function.

Thinking for a moment, I've realised that using Sinon with Chai, it's not needed, to have to define parameters, just defining the position of the callback parameters and the values pass in the "stubs" are enough and move to chai the assertions checking in the right place, `it` Mocha function, so it's more right, simpler and readable, hence better.


## Conclusion

Test your code is king, sometime is harder than writing the implementation, in this case I've spent more time in the test than in the implementation, moreover if you want to have 100% coverage; having the coverage is good to know which parts of your implementations are tested and which ones not, so added whenever you can, as my friend did.

If you're starting with iojs/NodeJS, I think that the best way is to understand well the asynchronous programming pattern, is through the callback node convention, they're the basis of node and even though the promise approach are increasing fast, I think that it's worth to learn the basis first.

Testing your code when you're learning it's as good complement for your implementations, you'll learn a lot, and moving forward with spies, stubs and mock it's good to struggle, learn and when you beat them, you'll be ready to use whenever you need then, furthermore to be more experienced.

If you're learning node, don't miss to check <a href="http://nodeschool.io/" target="_blank">Node School</a> in case that you haven't heard about it.


I hope that this read has been useful for you.
Best!

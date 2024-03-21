+++
date = "2016-02-25T09:14:11Z"
description = "KoaJS v2 is production ready although it remains in alpha, why? go ahead and read this post to know and see how to use BabelJS with it"
title = "KoaJS v2, BabelJS & npm scripts"
social_image = "https://camo.githubusercontent.com/674563115c4e0d4e5d99440b916952ad795c498e/68747470733a2f2f646c2e64726f70626f7875736572636f6e74656e742e636f6d2f752f363339363931332f6b6f612f6c6f676f2e706e67"
tags = ["nodejs", "javascript", "eslint"]
categories = [
  "software development"
]
+++

Koa v2 has changed completely the middleware function signature, used in v1.x, to use {{<ext-link "ES2016 async/await functions" "http://tc39.github.io/ecmascript-asyncawait/">}} over {{<ext-link "generators" "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*">}}.

It's still in alpha even thought the Koa's team says that it's production ready, because it's currently used in some systems which are in production, however {{<ext-link "the issues is that NodeJS doesn't still support async/await function natively" "https://github.com/koajs/koa/issues/533">}}, hence you need a transpiler to be able to use async/await functions.

One, and probably the most famous, ES2015 tranpilers which also supports some of the features proposed for ES2016, and between then Async/Await functions, is {{<ext-link "BabelJS" "http://babeljs.io/">}}; it's also the one, most mentioned, in the Koa's issue linked above, but it doesn't look fancy having to transpile server code because it can be very painful, however Koa v2 doesn't only accept async/await functions as a middleware, it also accepts function which return Promises ({{<ext-link "which are already supported natively by last productions releases (LTS and no LTS)" "https://nodejs.org/en/docs/es6/#which-features-ship-with-node-js-by-default-no-runtime-flag-required">}}) because at the end an async/await function returns a Promise to the caller.

## What we can do now

If you don't want to use BabelJS to write server code because it may be a pain the ass having in your development pipeline, then {{<ext-link "you can still use Koa v2 using functions which return Promises" "https://github.com/koajs/koa/tree/2.0.0-alpha.3#example">}}; you don't have to worry in having to rewrite those middlewares again when async/await functions be natively supported by NodeJS, you could use those ones and the new ones that implemented with async/async function seamless.

If you don't mind to use BabelJS in your server code, because it's a new development which isn't going to be a very big codebase, or because the implementation progress is going to be slow for whatever reason or you have any other reason which isn't relevant for the purpose of this post then go for it and when NodeJS support async/away function natively then kill BabelJS.


## Koa v2, BabelJS, with Nodemon & npm scripts

In case that you want to use BabelJS, and you don't have any prepared boilerplate to develop a server side with Babel with server restarting automatically on each change, then you can follow this steps to install it, configure it and run it with npm.

Assuming that you have ran `npm init` in a folder and you're in your terminal on it execute

1. Install Koa v2 `npm install --save koa@next`
2. Install Babel and the presets and plugins needed to transpile ES6/7 features not supported natively by NodeJS; I recommend only use the async/await functions to be able to run your code as soon as async/await function are supported by NodeJS and may not others as spread paramaters, etc.
`npm install --save-dev babel-core babel-cli babel-preset-es2015-node5  babel-plugin-transform-async-to-generator`
3. Install nodemon `npm install --save-dev nodemon`

Then setup babel creating a `.babelrc` file with this content:

{{<highlight json>}}
{
  "env": {
    "development": {
      "presets": ["es2015-node5"],
      "plugins": ["transform-async-to-generator"]
    }
  }
}
{{</highlight>}}

Then you can create, into the property name `scripts` of `package.json`, something like this:

{{<highlight json>}}
{
  "scripts": {
    "dev": "nodemon --exec npm run -s serve",
    "serve": "babel-node -- src/server.js"
  }
}
{{</highlight>}}

In case that you use {{<ext-link "eslint" "http://eslint.org/">}} to lint your code, to avoid linter issues with async/await functions you have to install it and babel plugin

`npm install --save-dev eslint babel-eslint`

then set in `.eslintrc` file the property `"parser": "babel-eslint"`. Bear in mind that you will have to enable the `es6` features too, enabling {{<ext-link "`es6` environtment" "http://eslint.org/docs/user-guide/configuring#specifying-environments">}}.

and you can also setup a watch mode in package.json `scripts` property

{{<highlight json>}}
{
  "lint-w": "nodemon --watch src -e js --exec eslint ."
}
{{</highlight>}}

then you can run the "dev" script target in one terminal tab and in other this one and enjoy developing a Koa v2 app, with asyn/await function with live restart and live linting.


Don't wait until Koa v2 to be announced as production version, use it now with Babel or without, just use it now.

Happy coding!

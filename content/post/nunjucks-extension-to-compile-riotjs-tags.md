+++
date = "2016-03-02T16:05:37Z"
description = "Read why and how I've created a Nunjucks extension to compile RiotJS to use in one of my side projects and also see how it looks"
title = "A Nunjucks extension to compile RiotJS tags"
social_image = "https://s-media-cache-ak0.pinimg.com/originals/54/7e/12/547e12bf2630c38c30bbb8760f148221.png"
tags = ["nodejs", "javascript"]
categories = [
  "software development"
]
+++

On the way to London, meanwhile I was flying, I wanted to take advantage of the dead time without internet to write this short blog post about a {{<ext-link "Nunjucks" "http://mozilla.github.io/nunjucks/">}}, {{<ext-link "RiotJS" "http://riotjs.com/">}} and a Nunjucks extension that I've implemented, for a side project that I'm working on, to render RiotJS tags on the server.

## Nunjucks

{{<ext-link "Nunjucks" "http://mozilla.github.io/nunjucks/">}} is a template engine (yes, another one!) for NodeJS and the browser which is heavily inspired by {{<ext-link "jinja2" "http://jinja.pocoo.org/">}}.

I'd never used before and probably I had heard about it, but I couldn't remember, world is moving fast and almost any time you start a project you can discover lots of new stuff which may be better or a good fit it.

I've used {{<ext-link "Jade" "http://jade-lang.com/">}} and {{<ext-link "Swig" "http://paularmstrong.github.io/swig/">}} (a bit less) several times and for this project I was thinking in using {{<ext-link "Marko" "http://markojs.com/">}}, it has a few nice features as be able to send the content through a stream and a widget system which helps to create reusable components; however I found a bit confusing to start because there are several examples and {{<ext-link "screen casts" "https://www.youtube.com/playlist?list=PLZUgs-F38XXQJBxA58UqnwTVMZS_uza_C">}} but {{<ext-link "quite few functions have been deprecated in the new version" "http://markojs.com/docs/marko/javascript-api/#rendertemplatepath-templatedata-streamwritable">}} and I don't have the need of streaming the content to enhance performance for this project.

At the same time, I didn't want to spend lots of time in choosing and having to solve quite a few issues in how to use it and use it well to get the advantages, so I found Nunjucks was a good choice between taking something that I know and something that it's new for me.

On the other hand, in the time being I don't expect to use that many advanced features that Nunjucks has, and probably I won't take advantage of its power, but it's OK, as it provides me all the things that I need.

## RiotJS

{{<ext-link "RiotJS" "http://riotjs.com/">}} seems probably out of the context of this post and it could be if I wouldn't have decided to also incorporate since the first few commits.

It also could be React, but I feel that it would add too much complexity to the project, it isn't a big project and it shouldn't have a big code base; moreover I would have to spend more time to nail React and all the things around it so I would end up not making the things done.

I've had a experience using RiotJS, for a freelancing project that I was working a few weeks ago, it was a project with very hard deadline and I had to build a mobile friendly site for one existing one, created by one big "consultancy" company which basically they ended up delivering a huge pile of crap; anyway, the only think that matters for this post is that it couldn't be made responsive and in 3 weeks time I had to deep in that horrible code base, make the changes what I needed to serve the content for a {{<ext-link "SPA" "https://en.wikipedia.org/wiki/Single-page_application">}} which at the same time I had to create to be served for requests coming from mobile devices.

I had never used RiotJS before, and due its minimalism I was able to read and understand the docs, and just use it to do what I had to do.

For this side project, I could avoid to add Riot, because it makes sense to render the site in the server, however there are a few parts which I would like to render on the client (e.g. a dashboard), maybe not in the first version, but in one, close to the first one.

Choosing RiotJS, I think that I can take advantage of its {{<ext-link "custom tag" "http://riotjs.com/guide/">}} to create components which at the beginning I can render in the server into Nunjucks templates and at some point, start to render them in the client side in a few pages or parts of them.


## Rendering RiotJS in Nunjucks templates

To do this, I didn't need to create any plugin, I could follow the same approach which you can watch in {{<ext-link "this video using swig" "https://www.youtube.com/watch?v=6ww1UXGJzcs&feature=youtu.be" >}}, but I found that creating a Nunjucks plugin for this purpose was simple and easy and it will save me time to repeat over an over a few statements in any page that I'm going to drop RiotJS tags besides the implementation will be less error prone.

The implementation of {{<ext-link "Nunjucks extension" "http://mozilla.github.io/nunjucks/api.html#custom-tags">}} is as simple as:

{{<highlight js>}}
const path = require('path')
const nunjucks = require('nunjucks')
const riot = require('riot')

function RiotExtension(basePath) {
  this.tags = ['riot']

  this.parse = function (parser, nodes) {
    let tok = parser.nextToken()
    let args = parser.parseSignature(null, true)
    parser.advanceAfterBlockEnd(tok.value)

    parser.parseUntilBlocks('endriot')
    parser.advanceAfterBlockEnd()

    return new nodes.CallExtension(this, 'renderRiot', args, [])
  }

  this.renderRiot = function (ctx, view, opts) {
    /* eslint-disable global-require */
    const tag = require(path.join(basePath, view))
    /* eslint-enable global-require */
    return new nunjucks.runtime.SafeString(riot.render(tag, opts))
  }
}
{{</highlight>}}

On the server, I'm using {{<ext-link "KoaJS" "http://koajs.com/">}} v2, at that time it wasn't support in {{<ext-link "Koa-views" "https://github.com/queckezz/koa-views">}} for Koa v2 without using {{<ext-link "koa-convert" "https://github.com/koajs/convert">}}, maybe {{<ext-link "there will one soon" "https://github.com/queckezz/koa-views/pull/54">}}, and because I prefer to render views without having side effects in Koa `ctx` (context), I basically pass the render function where I need and use it to generate the HTML which I assign to `ctx.body` where I need.

The function which returns a promised render function, looks like

{{<highlight js>}}
const path = require('path')
const nunjucks = require('nunjucks')
const riot = require('riot')

function getRenderFn(viewsPath, opts) {
  riot.settings.brackets = '${ }'

  const njr = nunjucks.configure(viewsPath, opts)
  njr.addExtension('RiotExtension', new RiotExtension(path.join(viewsPath, 'riot')))

  return (view, data) => {
    return new Promise((resolve, reject) => {
      njr.render(view, data, (err, html) => {
        if (err) {
          reject(err)
          return
        }

        resolve(html)
      })
    })
  }
}
{{</highlight>}}

And then, whenever I need to render a Riot tag in a Nunjucks template I only have to do `{% riot "my-tag.html" %}{% endriot %}`

## Conclusion

Nunjuks looks as powerful template engine, on the other hand, I love the minimalism of RiotJS; with the combination of both I can render views in the server including components (Riot tags) than later on, I could reuse in the client side.

Nunjucks and RiotJS isn't the only two that you can combine together, there more famous combinations or just one which allow to create templates which can be rendered in the server and the client, but I've decided to choose those based on my project requirements (technical and non-technical too); my recommendation here, is, do the same, choose the one(s) which fit better to it.

Happy mixing!

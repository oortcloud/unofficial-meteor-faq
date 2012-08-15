Unofficial Meteor FAQ
=====================
Just answering some common questions that aren’t answered in the [official meteor FAQ](http://www.meteor.com/faq/).

##How can I use someone else's javascript in my meteor project?

It depends on how it's packaged. If there's already a smart package, you can use [meteorite](http://possibilities.github.com/meteorite/) to include it. Make sure you check the [atmosphere](http://atmosphere.meteor.com) package repository too.

If it's a simple client side JS script, you can include it in `client/` or `lib/`, although it might be nicer to create a smart package to wrap it, and publish that package on atmosphere. There are some [good instructions](https://atmosphere.meteor.com/wtf/package) on the atmosphere page about how to do that.

##What about node modules?

Node modules are more difficult as they are not included in the bundle created by the  `meteor deploy` command. So if you are publishing to `meteor.com`, you won't be able to use them.

There is [a way around](http://stackoverflow.com/questions/10476170/how-can-i-deploy-node-modules-in-a-meteor-app-on-meteor-com) this, via a hack using the `public/` directory. However YMMV with compiled modules or ones that make heavy use of system calls. Hopefully there will some official means of support in the future.


##How do I stop meteor from reactively overwriting DOM changes from outside meteor?

This comes up a lot when people try to use external JS libs like jquery-ui, jquery-mobile, bootstrap etc. Unfortunately the approach used by these libraries conflicts with meteor's model. Here are some things you could try to make it work:

1. Give your element an `id`.
2. Try to separate your element from the areas of the page that are changing via using separate templates.
3. Do the same using a helper and `Meteor.ui.chunk`.

People have had varying success with the above. However, soon we'll have the [spark](https://github.com/meteor/meteor/tree/spark) package, which has explicit support for this kind of thing.

##How do I animate when meteor changes things under me?

This is a similar problem to above. I outlined some techniques that have worked for me in a [blog post](http://bindle.me/blog/index.php/658/animations-in-meteor-state-of-the-game). 

##How do I animate things adding/being removed from collections?

This isn’t really possible to do (unless you put a delay on the element actually being removed from the collection), because liveui will remove the element from the DOM. However, it's hoped that spark will help with this case too.

##How do I route my app between different views/pages?

Ideally, you'd use an existing JS router and combine it with a reactive variable (e.g. in the session) which defines the visible page. Or you could just try my [reactive router](https://github.com/tmeasday/meteor-router) which does this for you with the backbone router. Worth a look even if you want to use a different router as it could give you some ideas.

##How do I animate/transition such view changes?

I've written an [entire post](http://bindle.me/blog/index.php/679/page-transitions-in-meteor-getleague-com) about this topic.

##How do I get a callback that runs after my template is attached to the DOM?

The upcoming spark branch will provide ‘official’ ways to do this, but for now, the best you can do is this “unofficial” [hack](http://stackoverflow.com/questions/10109788/callback-after-the-dom-was-updated-in-meteor-js).

##How do I know when my subscription is "ready" and not still loading?

Obviously data could keep changing indefinitely, but for the first set of data, you can use meteor’s `onComplete` callback:

```js
Session.set('fooLoading', true); 
Meteor.subscribe('foo', function() { 
  Session.set('fooLoading', false); 
});
```

[I'm hoping](https://github.com/meteor/meteor/pull/273) this will be even easier in the future.

##Where should I put my files?

The example apps in meteor are very simple, and don’t provide much insight. Here’s my current thinking on the best way to do it: (any suggestions/improvements are very welcome!)

```bash
lib/                    # <- any common code for client/server
lib/environment.js      # <- general configuration
lib/vendor              # <- common code from someone else
models/                 # <- definitions of collections and methods on them (could be collections/)

client/index.html       # <- toplevel html
client/index.js         # <- and it's JS
client/application.js   # <- subscriptions, basic Meteor.startup code.
client/environment.js   # <- configuration of any client side packages
client/views/page.html  # <- the templates specific to a single page
client/views/page.js    # <- and the JS to hook it up
client/helpers          # <- any helpers (handlebars or otherwise) that are used often in view files

server/publications.js  # <- Meteor.publish definitions
server/methods.js       # <- Meteor.method definitions
server.environment.js   # <- configuration of server side packages
```


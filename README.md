Unofficial Meteor FAQ
=====================
Just answering some common questions that aren’t answered in the [official meteor FAQ](http://www.meteor.com/faq/).

##How do I update this FAQ?

Send me (Tom Coleman, @tmeasday) a message on github (or otherwise) or fork the FAQ and send a pull request (we can discuss changes and ideas as part of the pull request).

##How do I use a unreleased branch of meteor?
You can git clone it anywhere on your system and then run it with:
```bash
$ path/to/checked/out/meteor/meteor
```

Or you could let [meteorite](http://oortcloud.github.com/meteorite/) do this for you.

##How can I use someone else's javascript in my meteor project?

It depends on how it's packaged. If there's already a smart package, you can use [meteorite](http://oortcloud.github.com/meteorite/) to include it. Make sure you check the [atmosphere](http://atmosphere.meteor.com) package repository too.

If it's a simple client side JS script, you can include it in `client/lib/` or `lib/`, although it might be nicer to create a smart package to wrap it, and publish that package on atmosphere. There are some [good instructions](https://atmosphere.meteor.com/wtf/package) on the atmosphere page about how to do that.

##What about node modules?

Node modules are more difficult as they are not included in the bundle created by the  `meteor deploy` command. So if you are publishing to `meteor.com`, you won't be able to use them.

There is [a way around](http://stackoverflow.com/questions/10476170/how-can-i-deploy-node-modules-in-a-meteor-app-on-meteor-com) this, via a hack using the `public/` directory. However it's also usually necessary to wrap the API in a fiber to allow it to interoperate with meteor, and chances are things are bit complex. You'll want to take a look at Mike Bannister's [node-modules package](https://github.com/possibilities/meteor-node-modules).


##How do I stop meteor from reactively overwriting DOM changes from outside meteor?

You'll want to read the [templates section of the docs](http://docs.meteor.com/#templates_api).

##How do I animate when meteor changes things under me?

This is a similar problem to above. I outlined some techniques that have worked for me in a [blog post](http://bindle.me/blog/index.php/658/animations-in-meteor-state-of-the-game). 

##How do I animate things adding/being removed from collections?

One way to animate things being added is to transition from a `.loading` class when it's rendered. In the relevant template's `rendered` function, add some code like:
```js
Template.item.rendered = function() {
  var $item = $(this.find('.item'));
  Meteor.defer(function() {
    $item.removeClass('loading');
  });
}
```

The `defer` (which corresponds to a `setTimeout` of `0`) is necessary as otherwise the browser won't have had a chance to render the item in the `.loading` state (i.e. hidden / offscreen / whatever).

Removal animations are more difficult, because Meteor will remove the element from the DOM immediately upon the item leaving the collection. Some hacks that you could try (please show me if you get anything working!): 
  - Not deleting the element immediately but waiting for the animation to complete (ughhh, not great interactivity between users).
  - Hooking up your own `observe` calls on the collection so you can handle `removed` differently - ie. making an 'animating' cursor. (Much better, perhaps over-engineering the problem)

##How do I route my app between different views/pages?

Ideally, you'd use an existing JS router and combine it with a reactive variable (e.g. in the session) which defines the visible page. Or you could just try my [reactive router](https://github.com/tmeasday/meteor-router) which does this for you with the backbone router. Worth a look even if you want to use a different router as it could give you some ideas.

##How do I animate/transition such view changes?

I've written an [entire post](http://bindle.me/blog/index.php/679/page-transitions-in-meteor-getleague-com) about this topic.

##How do I get a callback that runs after my template is attached to the DOM?

This is now straightforward with the `rendered` [callback](http://docs.meteor.com/#template_rendered).

##How do I know when my subscription is "ready" and not still loading?

Obviously data could keep changing indefinitely, but for the first set of data, you can use meteor’s `onComplete` callback:

```js
Session.set('fooLoading', true); 
Meteor.subscribe('foo', function() { 
  Session.set('fooLoading', false); 
});
```

[I'm hoping](https://github.com/meteor/meteor/pull/273) this will be even easier in the future.

##How do I hook into an existing, running MongoDB instance?
You can start meteor with the `MONGO_URL` environment var set:
```bash
$ MONGO_URL=mongodb://localhost:27017 meteor
```

Note that pre-existing data in mongo [may be hard to deal with](https://github.com/meteor/meteor/issues/61).

##How do I set environment variables on meteor.com?
I don't believe this is possible.

##What are the best practices for form handling?
There is some good work in Mike Bannister's [form's package](http://forms.meteor.com/), however it's a little out of date right now. I imagine there will be something coming in core eventually.

##What is the best way to do file uploads?
There's brief discussion [on telescope](http://telesc.pe/posts/ae8561f8-02c6-47da-81d3-4758ee6effa3). Also, there's a [filepicker smart package](https://atmosphere.meteor.com/package/filepicker).

##Where should I put my files?

The example apps in meteor are very simple, and don’t provide much insight. Here’s my current thinking on the best way to do it: (any suggestions/improvements are very welcome!)

```bash
lib/                    # <- any common code for client/server. 
lib/environment.js      # <- general configuration
lib/external            # <- common code from someone else
## Note that js files in lib folders are loaded before other js files.

collections/                 # <- definitions of collections and methods on them (could be models/)

client/lib              # <- client specific libraries (also loaded first)
client/lib/environment.js   # <- configuration of any client side packages
client/lib/helpers      # <- any helpers (handlebars or otherwise) that are used often in view files

client/application.js   # <- subscriptions, basic Meteor.startup code.
client/index.html       # <- toplevel html
client/index.js         # <- and its JS
client/views/<page>.html  # <- the templates specific to a single page
client/views/<page>.js    # <- and the JS to hook it up
client/views/<type>/    # <- if you find you have a lot of views of the same object type

server/publications.js  # <- Meteor.publish definitions
server/methods.js       # <- Meteor.method definitions
server/lib/environment.js   # <- configuration of server side packages
```

For larger applications, discrete functionality can be broken up into sub-directories which are themselves organized using the same pattern. The idea here is that eventually module of functionality could be factored out into a separate smart package, and ideally, shared around.

```bash
feature-foo/            # <- all functionality related to feature 'foo'
feature-foo/lib/        # <- common code
feature-foo/models/     # <- model definitions
feature-foo/client/     # <- files only sent to the client
feature-foo/server/     # <- files only available on the server
```
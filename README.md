Unofficial Meteor FAQ
=====================
Just answering some common questions that aren’t answered in the [official meteor FAQ](http://www.meteor.com/faq/). Thanks go to [@dandv](/dandv), [@possibilities](/possibilities), [@IainShigeoka](/IainShigeoka), [@brainclone](/brainclone), [@matb33](/matb33), [@drorm](/@drorm) and [@zVictor](/@zVictor).

###How do I update this FAQ?

Send me (Tom Coleman, @tmeasday) a message on github (or otherwise) or fork the FAQ and send a pull request (we can discuss changes and ideas as part of the pull request).

###How can I find out more about Meteor?

 - Obviously, start at the [Meteor Website](http://meteor.com).
 - [This blog post](http://www.discovermeteor.com/2013/02/10/useful-meteor-resources/) has a list of online resources.
 - [Discover Meteor](http://discovermeteor.com) (written by the author of this list) is a book which takes you through building an app from scratch.

## Customizing Meteor

###How do I use a unreleased branch of meteor?
You can git clone it anywhere on your system and then run it with:
```bash
$ path/to/checked/out/meteor/meteor
```

Or you could let [meteorite](http://oortcloud.github.com/meteorite/) do this for you.

###How can I use someone else's javascript in my meteor project?

It depends on how it's packaged. If there's already a smart package, you can use [meteorite](http://oortcloud.github.com/meteorite/) to include it. Make sure you check the [atmosphere](http://atmosphere.meteor.com) package repository too.

If it's a simple client side JS script, you can include it in `client/lib/` or `lib/`, although it might be nicer to create a smart package to wrap it, and publish that package on atmosphere. There are some [good instructions](https://atmosphere.meteor.com/wtf/package) on the atmosphere page about how to do that.

###What about npm (node) modules?

Meteor by default comes bundled with a set of node modules, which you can access via:
```js
var foo = Npm.require('foo');
```
If you need a npm package that is not included in Meteor, you need to create a Meteor package that wraps it. Avital Oliver created a simple example for the [xml2js npm library](https://github.com/avital/meteor-xml2js-npm-demo).
>Note that you _cannot_ `Npm.require` such npm packages from _within your app_, you can only require them _within the Meteor package that depends on them_. This is due to different Meteor packages potentially depending on different versions of the same npm package.

There is another way you can use NPM modules with your app. 
 * first you need to install [npm](https://atmosphere.meteor.com/package/npm) smart package with - `mrt add npm`
 * then add a file named `packages.json` and add npm modules
 
Read this [guide](http://meteorhacks.com/complete-npm-integration-for-meteor.html) to learn more on [using NPM modules with a meteor app](http://meteorhacks.com/complete-npm-integration-for-meteor.html).

## Reactivity and Rendering

###How do I stop meteor from reactively overwriting DOM changes from outside meteor?

You'll want to read the [templates section of the docs](http://docs.meteor.com/#templates_api).

### How do I ensure control state preservation across live page updating?

Add the `preserve-inputs` package.

From the [docs](http://docs.meteor.com): "This preserves all elements of type input, textarea, button, select, and option that have unique id attributes or that have name attributes that are unique within an enclosing element with an id attribute."

Also, make sure to store related client-side data in the [Session](http://docs.meteor.com/#session) object (not in JavaScript variables)

NOTE: form data will still get cleared across hot-code pushes unfortunately.

###How do I get a callback that runs after my template is attached to the DOM?

This is now straightforward with the `rendered` [callback](http://docs.meteor.com/#template_rendered).

###How do I get reactive HTML in the `<head>` tag?

Unfortunately, this [isn't yet supported](https://github.com/meteor/meteor/issues/266). However, you can work around it, like so:
```js
Meteor.startup(function() {
  Meteor.autorun(function() {
    document.title = Session.get('document-title');
  });
});
```

Note also that this code should work fine with the spiderable package, as long as the `document-title` session var is set when your site initially loads (for instance in a router).

### How can I tell why my code is getting re-run?

If you are using an `autorun` block, you could try this:

```js
Deps.autorun(function(computation) {
  computation.onInvalidate(function() {
    console.trace();
  });
  
  // standard autorun code...
});
```

## Animation
### How do I animate when meteor changes things under me?

This is a similar problem to above. I outlined some techniques that have worked for me in a [blog post](http://bindle.me/blog/index.php/658/animations-in-meteor-state-of-the-game).
### How do I ensure control state preservation across live page updating?

Add the `preserve-inputs` package.

From the [docs](http://docs.meteor.com): "This preserves all elements of type input, textarea, button, select, and option that have unique id attributes or that have name attributes that are unique within an enclosing element with an id attribute."

Also, make sure to store related client-side data in the [Session](http://docs.meteor.com/#session) object (not in JavaScript variables)

NOTE: form data will still get cleared across hot-code pushes unfortunately.

###How do I animate things adding/being removed from collections?

One way to animate things being added is to transition from a `.loading` class when it's rendered. First, add a `.loading` class to to the element in the HTML.

Then, in the relevant template's `rendered` function, remove the class just after it attaches to the document:
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

You may also want to [render items added by the client in a different state, until they're confirmed server side](http://stackoverflow.com/questions/10082537/in-meteor-how-do-i-show-newly-inserted-data-as-greyed-out-until-its-been-confi).


###How do I route my app between different views/pages?

Ideally, you'd use an existing JS router and combine it with a reactive variable (e.g. in the session) which defines the visible page. Or you could just try my [reactive router](https://github.com/tmeasday/meteor-router) which does this for you with the backbone router. Worth a look even if you want to use a different router as it could give you some ideas.

###How do I animate/transition such view changes?

I've written an [entire post](http://bindle.me/blog/index.php/679/page-transitions-in-meteor-getleague-com) about this topic.


## Subscriptions and Methods

### How can I alter every document before it is added to the database?
There isn't support for this right now, but you can use a `deny` to achieve what you want on the server. For example, to timestamp each document as it goes into mongo:

```js
Posts.deny({insert: function(userId, doc) {
  doc.createdAt = new Date().valueOf();
  return false;
}})
```

You can do something similar with `update` to for example simulate a synthetic `updatedAt` field.

Avoid the tempation to use `allow` for this, there's no guarantee it will run (only one `allow` needs to be successful for the `insert` to go ahead).

Alternatively, Mathieu Bouchard (@matb33) has some more complex code to achieve this in a more integrated way [here](https://gist.github.com/matb33/5258260). Ideally in the future core will support something like this.


### How do I know when my subscription is "ready" and not still loading?

When you subscribe to a collection you are returned a handle.  This handle has a reactive ready() method that lets you know when the initial snapshot of the collection has been sent to the client.

```js
globalSubscriptionHandles.push(Meteor.subscribe('foo'));

...

Template.item.areCollectionsReady = function()
{	
	var isReady = globalSubscriptionHandles.every(function(handle)
	{
	    return handle.ready();
	});

	return isReady;
}
```

### Why does `observe` fire a bunch of `added` events for existing documents?

The default behaviour is to do this, because it's often what you want (think about how the `{{#each}}` tag works for instance). You can avoid it with a hidden options though:

```js
Posts.find().observe({
  _suppress_initial: true,
  added: function() {
    // ...
  }
})
```

Note that considering this option is undocumented (and underscore-cased), it could silently change in the future.

### Why do I sometimes see two copies of a document during a latency compensated method?

If you insert a document in the both the real (server-side) method and the simulation (client-side), but then subsequently do more work on the server, there will be a time period where, for the a single browser, there will be two copies of the document in the database.

Meteor dev Nick Martin [wrote](https://github.com/meteor/meteor/issues/881#issuecomment-15882223):

> There are a couple ways to work around this:

> a) pick the same id on the client and the server. The easiest way to do this (and what the default 'insert' method does) is simply pick an id on the client and pass it along with the document to be inserted.

> Doing this will result in their only being one document in the database, so you won't see double. It will still be the case that the server changes to the document don't show up locally until the method is done, though.

> b) make the method finish faster. The latency compensation is triggered when the method finishes. If the thing you are blocking on doesn't do any writes (or its writes don't need to be latency compensated), you can have the method return early and complete its work asynchronously.

> c) do the writes at the end of the method on the server. batch up all the database modifications and wait till after all the blocking stuff is done. this is gross, but should work to reduce the time where the user may see two items.

> We also have an item on the roadmap to make this easy:
> https://trello.com/card/pattern-for-creating-multiple-database-records-from-a-method/508721606e02bb9d570016ae/57



## Deployment
###Where can I host my applications for production?
Meteor gives you a proper infrastructure to deploy your application:
```bash
meteor deploy myapp.meteor.com
```

However, there are some [other options available](http://stackoverflow.com/a/13504325/599991).

###How do I hook into an existing, running MongoDB instance?
You can start meteor with the `MONGO_URL` environment var set:
```bash
$ MONGO_URL=mongodb://localhost:27017 meteor
```

Note that pre-existing data in mongo [may be hard to deal with](https://github.com/meteor/meteor/issues/61).

###How do I set environment variables on meteor.com?
This is not currently possible, unfortunately. You can set `process.env.X = Y` early on in the startup of your app though.

## Best practices

###What are the best practices for form handling?
There is some good work in Mike Bannister's [forms package](http://forms.meteor.com/), however it's a little out of date right now. I imagine there will be something coming in core eventually.

###What is the best way to do file uploads?
There's brief discussion [on telescope](http://telesc.pe/posts/ae8561f8-02c6-47da-81d3-4758ee6effa3). Also, there's a [filepicker smart package](https://atmosphere.meteor.com/package/filepicker).

###What are best practices for security?
First off, you'll want to make sure you are using version 0.5.0 or later, which implements [authentication](http://docs.meteor.com/#accounts_api). Some other points:

1. Files in `server/` do not get served to the client
2. All other JS files do, (although they are minified + concatenated).
3. Any method call can be made by hand from the JS console.

###How do I debug my meteor app?
Client-side you have the console. For server-side debugging, use [node-inspector](/dannycoates/node-inspector) and make sure you have meteor [v0.5.3](/meteor/meteor/blob/master/History.md#v053), which makes things easier thanks to support for `NODE_OPTIONS`:

1. Install node-inspector: 'npm install -g node-inspector'
2. Start meteor: `NODE_OPTIONS='--debug' mrt run`
3. Start node-inspector
4. Go to the URL given by node-inspector in Chrome
5. Debug at will

###What are the best practices for Test-Driven Development?
TDD support isn't official yet in meteor, but (test) files placed in the `tests` subdirectory won't be loaded on the client or server. There are various Node.JS modules that help with testing - see [Meteor test driven development](http://stackoverflow.com/questions/12987525/meteor-test-driven-development) on SO.

###Where should I put my files?

The example apps in meteor are very simple, and don’t provide much insight. Here’s my current thinking on the best way to do it: (any suggestions/improvements are very welcome!)

```bash
lib/                       # <- any common code for client/server.
lib/environment.js         # <- general configuration
lib/methods.js             # <- Meteor.method definitions
lib/external               # <- common code from someone else
## Note that js files in lib folders are loaded before other js files.

collections/               # <- definitions of collections and methods on them (could be models/)

client/lib                 # <- client specific libraries (also loaded first)
client/lib/environment.js  # <- configuration of any client side packages
client/lib/helpers         # <- any helpers (handlebars or otherwise) that are used often in view files

client/application.js      # <- subscriptions, basic Meteor.startup code.
client/index.html          # <- toplevel html
client/index.js            # <- and its JS
client/views/<page>.html   # <- the templates specific to a single page
client/views/<page>.js     # <- and the JS to hook it up
client/views/<type>/       # <- if you find you have a lot of views of the same object type
client/stylesheets/        # <- css / styl / less files

server/publications.js     # <- Meteor.publish definitions
server/lib/environment.js  # <- configuration of server side packages

public/                    # <- static files, such as images, that are served directly.

tests/                     # <- unit test files (won't be loaded on client or server)
```

For larger applications, discrete functionality can be broken up into sub-directories which are themselves organized using the same pattern. The idea here is that eventually module of functionality could be factored out into a separate smart package, and ideally, shared around.

```bash
feature-foo/               # <- all functionality related to feature 'foo'
feature-foo/lib/           # <- common code
feature-foo/models/        # <- model definitions
feature-foo/client/        # <- files only sent to the client
feature-foo/server/        # <- files only available on the server
```

### What IDEs are best suited for meteor?

While many IDEs [support SASS](http://sass-lang.com/editors.html), [CoffeeScript](http://stackoverflow.com/questions/4084167/ide-or-its-add-in-for-coffeescript-programming) etc., at the moment, there are no known IDEs that support meteor. Two efforts are underway:

* [Cloud9](https://github.com/meteor/meteor/issues/214#issuecomment-8892697), the cloud-based IDE with Node.js support
* JetBrains [WebStorm](http://youtrack.jetbrains.com/issue/WI-13858) (Win/Mac/Linux), with existing support for Node.js. If you use Webstorm, make sure to [disable auto-save](devnet.jetbrains.net/message/5469319).Webstorm is proprietary software that offers a [gratis license](http://www.jetbrains.com/webstorm/buy/index.jsp) to people working on open source projects


### What versions of Internet Explorer does meteor support?

There isn't an official list of supported versions, but meteor is known to work on IE8+.

## Troubleshooting errors

### "Uncaught SyntaxError: Unexpected token Y"

Client-side error caused by the server crashing and sending a message starting with "Your app is crashing. Here's the latest log."

### "TypeError: Object #<Object> has no method '_defineMutationMethods'"

Server-side error. Most likely, you forgot to place "new" before a constructor call in the client. [Read more](https://github.com/meteor/meteor/issues/457).

### "Uncaught TypeError: Converting circular structure to JSON"

Check if you're trying to save into `Session` an object with circular references, such as a `Collection`. [Read more](https://github.com/meteor/meteor/issues/139#issuecomment-9975384).

### "Unexpected mongo exit code 100. Restarting."

Mongo was killed without cleaning itself up. Try removing `.meteor/local/db/mongod.lock`. If that fails do an `meteor reset`.

### `@importing` in less files causes errors related to variables not found.

If you're using a collection of less files that need to be imported in a specific order because of variable dependencies (like a custom Twitter Bootstrap installation) you can change the extension of the less files _to be imported_ from `.less` to `.lessimport` and then change your `@import file.less` to `@import file.lessimport`. This will prevent the less compiler from automatically trying to compile all your import files independently, yet still let you use them in the order you want.

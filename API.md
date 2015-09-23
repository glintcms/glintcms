# API

The design of the module's `api` was chosen to get a good balance between "easy to use" and  "easy to extend".

The `api` of the modules is designed after what we call the `associative-provider` model.

You wouldn't use a `TextBlock` directly, but via a `Block` that holds the `TextBlock`
	
```javascript
//e.g.: Block -> TextBlock
var block = Block(TextBlock()).use(Style());
```


# general

## extendability

Have a look at the [extend](EXTEND.md) documentation.

The following `api` document describes how to "use" the `building blocks`, (not how to extend them).


## naming/require
The `building blocks` are called: `wrap, container, etc.` in this document. However, the module names to require them start always with `glint-`.
Also in general the module names are hierarchically structured, separated with a dash `-`, where on the left hand side is the more generic part of the name.

So if you want to use the `adapter` with the `ajax` provider for example, you do it like this:

```javascript
var Adapter = require('glint-adapter');
var AjaxAdapter = require('glint-adapter-ajax');

var adapter = Adapter(AjaxAdapter());
```


## get/set
Getter and Setter can be called like this:

```javascript
// set
var place = obj.place('server');

// get
console.log(obj.place());
// --> server

```

When you provide a value (set), the method returns `this`, so that you can chain other methods.


## chainable methods
Most of the methods are chainable (they return this):

```javascript
// example:
Wrap(o)
  .editable(req.userCan('edit'))
  .i18n(req.i18n)
  .cid(req.params.article)
  .place(o.place)
  .load(res.locals, function(err, result) {
     debug('route loaded', err, req.params.article, result);
     if (err) return next(err);
     res.send(result.page);
  })
```

## events

Most of the `building blocks ` inherit from `EventEmitter` and expose useful events:

```javascript
// Example from the code:
var EventEmitter = require('events').EventEmitter;

/**
 * Expose Block element.
 */
exports = module.exports = Block;
inherits(Block, EventEmitter);
```

The Events can be especially helpful, when designing a `plugin`.

### get/set

Getter and Setter emit an event:
- `emit(name, value)`

### methods

The building blocks methods emit events for every method with the given name:
- `emit(pre-<methodName>, arguments)`
- `emit(<methodName>, arguments)`
- `emit(post-<methodName>, arguments)`


## rendering/place
rendering (load) is done by default on the server.
editing, saving and deleting is always initiated in the browser.
blocks and widgets can be defined to render in the browser when needed.
however you can also override where the components (blocks and widgets) are rendered all together.
you can use this for example to let everything be rendered on the server,
when the site is being called by a bot, search engine, crawler or the like.


**place**
glint is designed to use on the server as well as in the browser (pick you buzzword for it... universal, polymorph, what ever you like).
With the `place` get/set method, you can define, where you would like the control to be rendered.

These are the available options (strings):
- server
- browser 
- both
- force:server
- force:browser
- force:both


**priorities**

    (0:low priority ... 3:high priority)

     0 render on server by default

     1 Block.render('browser') or
       Widget.render('browser')
       -> render these items in the browser

     2 Wrap.render('server') or
       Container.render('server')
       -> render ALL items on the server, e.g. when requested by a search engine.

     3 SpecificBlock.render('force:both') or
       Widget.render('force:both)
       -> when a Specific Block has this flag, it will always be rendered on both sides (server and browser)

     4 Same as priority 3 but with 'force:server' or 'force:client'
       -> render always on the server respectively in the browser


# wrap

A `wrap` can hold several `container`s and `widget`s as well as other `wrap`s.
Let's call them `controls`. With the wrap, you can define how the `controls` are loaded.
First you can define the default object with the method `defaults`.
With the methods `parallel`, `series` and `eventually`, you can define everything from a simple to a quite sophisticated loading execution.
See [flow-builder](https://github.com/intesso/flow-builder)

The `wrap` runs on the server and/or in the `browser` depending on the defined `place`s.

## properties

### api
the `api` property must not be overwritten.

```javascript
Wrap.api === 'wrap'
```

### get/set

You can get/set these properties on the `wrap`.
- key
- id
- selector
- prepend
- append
- el
- place
- editable
- cid

When you set `editable` or `place` on the wrap, the value is also set on all of it`s controls.

`cid` gets/sets the id on the first `container` that the `wrap` holds.


## methods

### constructor

```javascript

// the `wrap` has got the optional key, control arguments. what's added in the constructor get's added with the `parallel` workflow.
var wrap = Wrap(key, control);

// but you can also create the Wrap fist, and then define your workflow.
// in this example the articles and projects `load in parallel` and after they are done,
// the resulting transfer object is handled over to the next steps:
//   in this exampe it's first the contentWidget, and then the layoutWrap after the previous step is done.
var wrap = Wrap();
wrap
  .parallel(container)
  .parallel('articles', articles.selector('.js-articles'))
  .parallel('projects', projects.selector('.js-projects'))
  .series('content', contentWidget.place('force:server'))
  .series(LayoutWrap(o.layout).place('force:server'))
```

### defaults

It let's you define a default object, that's the starting object in every `load` workflow.
This object is the *initial workflow transfer object*

```javascript
// set single key, value
wrap.defaults(key, value);
// set object
wrap.defaults(object);
// get value
wrap.defaults(key);
```

### parallel
When you define several `controls` with the `parallel` method right after each other, they get executed in parallel, and only when all of them have finished (or one of them has an error), the next step is executed.

```javascript
// adds a single control, the clone of the resulting object merged with the transfer object.
wrap.parallel(control;
// adds a sincle control, the clone of the resulting object is insterted into the transfer object with the given `key`.
wrap.parallel(key, control);

// examples
wrap.parallel(container);
wrap.parallel('news', widget1);
wrap.parallel('articles', widget2);
```

### series
`series` method calls, are executed after the previous method has finished.
It has got the same signature as the `parallel` method.

### eventually
`eventually` method calls are started immediately, but are evaluated only at the very end when the whole workflow has finished.
It has got the same signature as the `parallel` method.

### load
Calling the `load` method, starts the defined wrap workflow.


```javascript
// it has got an optional context object. This object is taken as the *initial workflow transfer object* when provided.
// the callback function `callback(err, result)` is called once everything is `load`ed.

wrap.load([context, ] callback);
// or
wrap.load(callback);


// Example:
function(req, res, next) {
  wrap.load(res.locals, function(err, result) {
    if (err) return next(err);
    res.send(result.page);
  });
}
```



# widget
With a widget, you can render/display noneditable content on the server and/or in the browser depending on the defined `place`.

## properties

### api
the `api` property must not be overwritten.

```javascript
Widget.api === 'widget'
```

### get/set

You can get/set these properties on the `widget`.
- key
- id
- selector
- prepend
- append
- el
- place
- template
- data
- render

When you set `editable` or `place` on the wrap, the value is also set on all of it`s controls.

`cid` gets/sets the id on the first `container` that the `wrap` holds.


## methods

### constructor

```javascript
// you can provide the `render` method in the constructor
var widget = Widget(renderFunction);

// or you can provide the `data` and `render` method on the widget instance. Example:
Widget()
  .data(function(fn) {
    adapter.findLatest(o.getLocale(), 3, fn);
  })
  .render(function(options) {
    return ejs.render(o.template || template, options);
  })
```

### data
You can optionally provide a `data` method, if your widget need's to make asynchronous data calls.
It returns `this` and takes a callback function with the parameters: `callback(err, result)`.


```javascript
widget.data(callback);
```

### render
The `render` function is a synchronous function and must return the rendered content.
The data object is provided in the first argument.
It let's you choose what ever rendering engine you want to use (as long as it runs on the server as well as in the browser).

```javascript
wrap..render(function(options) {
  return compiledDotTemplate(options);
})

```


### load
The `load` method is called from the `wrap` that contains this widget during the `load` workflow execution.
Most probably, you never have to call this method directly.
Internally, the `load` method calls the `data` method (if it was provided), and then `render` with the resulting object.



# container
Containers are only used when you use `blocks`.
A container runs on the server and in the browser. On the server, only the `load` method is called (most likely from the surrounding `wrap`),
In the browser, there is more methods:
- load
- edit
- save
- cancel
- delete



## properties

### api
the `api` property must not be overwritten.

```javascript
Container.api === 'container'
```

### get/set

You can get/set these properties on the `container`.
- key
- id
- place
- template
- editable
- blocks
- adapter


## methods

### constructor


```javascript
// you can optionally provide the `blocks` as well as the `adapter` in the constructor

var blocks = {
  title: text().selector('body h1'),
  short: text().selector('[data-id=short]'),
  text: editor().selector('[data-id=text]'),
  meta: Block(MetaBlock())
};

var adapter = Adapter(AjaxAdapter())
  .db(db)
  .type(type)
  .use(Dates())
  .use(Id())

// constructor
var container = Container(blocks, adapter);


// or you can provide them afterwards. Example:
var container = Container();
container
  .blocks(blocks)
  .adapter(adapter)
```


### load
(it runs either on server or browser depending on the `place`)

> Internal sequence:

1. it first calls the `adapter`s `load` method,
2. afterwards all of the `block`s `load` methods


### edit
(runs only in the browser)

> Internal sequence:

1. it first calls the `load` method on this container
2. it then calls the `edit` method on all of the `block`s.


### save
(runs only in the browser)

> Internal sequence:

1. first it calls all of the `block`s `save` methods
2. it then calls the `adapter`s `save` method,
3. aaand finally it calls the `load` method on this container to finish the command.



### cancel
(runs only in the browser)

> Internal sequence:

1. it basically just calls the `load` on this container.


### delete
(runs only in the browser)

> Internal sequence:

1. it first calls the `adapter`s `delete` method,
2. and afterwards it calls the `load` on this container, to finish things up.



# block

The `blocks` are the heart of everything that is editable in *GlintCMS*.

## properties

### api
the `api` property must not be overwritten.

```javascript
Block.api === 'block'
```

### get/set

You can get/set these properties on the `block`.
- id
- selector
- el
- place


## methods


### constructor/methods

```javascript
// you normally instantiate the `block` with the specific `block-provider` that it should hold
var block = Block(blockProvider);

// Example:
var block = Block(TextBlock()).use(Style());

// however you can also add/remove the `block-provider` later with `delegate` and `undelegate`
var block = Block();
block.delegate(TextBlock());

// you can also undelegate a block
block.undelegate(textBlock);
```

The `block` basically delegates the method calls to the specific `block-provider`.
Due to a runtime behaviour (missing `el` "HTMLElement"), the `block` has the ability to buffer method calls in a FIFO, and execute them later.

In addition to the 'getters and setters', it buffers and forwards the following methods:
- load
- edit
- save
- cancel
- hasChanged
- isValid

### plugins

You can extend `block`s with plugins with the `use` method, as well as with the `mixin method`.

#### use

Consult the [extend] documentation or the code.

#### mixin

Consult the [extend] documentation or the code.


```javascript
// Example:
var block.use(Style());

```


# adapter

The `adapter` is the interface to the storage.


## properties

### api
the `api` property must not be overwritten.

```javascript
Adapter.api === 'adapter'
```

### get/set

You can get/set these properties on the `adapter`.
- db
- type
- fn



## methods


### constructor/methods

```javascript
// you normally instantiate the `adapter` with the specific `adapter-provider` that it should delegate it's calls to.
var adapter = Adapter(adapter);

// Example:
var adapter = Adapter(AjaxAdapter());

// however you can also add/remove the `adapter-provider` later with `delegate` and `undelegate`
var adapter = Adapter();
adapter.delegate(AjaxAdapter());

// you can also undelegate a block
adapter.undelegate(ajaxAdapter);
```

The `adapter` basically delegates the method calls to the specific `adapter-provider`.
- find(query, callback)
- load(id, callback)
- save(id, content, callback)
- delete(id, callback)

The function has the form: `callback(err, result)`

the `result` object is the parsed javascript object for the given `id`,
and an array with the matching objects on the `find` callback.

### plugins

You can extend `adapter`s with plugins with the `use` method, as well as with the `mixin method`.

#### use

Consult the [extend] documentation or the code.

#### mixin

the `mixin` is mainly used to extend the adapter's query capability.
Since the `adapter` does not "magically unify the different storage query language", but exposes it directly,
the `adapter` needs a way to handle the different provider's queries.

It does it with the mixin:

```javascript
var mixin = {
  fs: {
    findWithLocale: function(locale, fn) {
      var query = 'this.id.indexOf("__template__") === -1 && this.locale === "' + locale + '"';
      this.find({$where: query}, fn);
    }
  },
  elasticsearch : {
    findWithLocale: function(locale, fn) {
      // pseudocode
      this.find(elasticSearchSpecificQuery, fn)
    }
 }
};

adapter.mixin(mixin);
```

As you can see in the example, the mixin must have an object, with the `adapter-provider` name as the key,
and the functions to `mixin` as the value (nested object).

This way you can support more than just one adapter-plugin,
and if you use some one else's module that does not have the queries for your provider,
just add them yourself and send a pull request.


```javascript
// Example:
var block.use(Style());

```


# trigger

A `trigger` consumes the `container`s api methods and exposes them to the user in a usable form.
Therefore `trigger`s are only useful in the browser, not on the server side.
Although theoretically, you could write a trigger to use on the server side as well, maybe for application integration.

```javascript
// usage example with the trigger implementation: keyboard and sidenav.

var keyboard = require('glint-trigger-keyboard');
var sidenav = require('glint-trigger-sidenav');

wrap.containers.forEach(function(container) {
  keyboard().add(container);
  sidenav().add(container);
});
```

The trigger itself ('glint-trigger') is only a `base class`, the trigger implementors can inherit:

```javascript
// Snippet from the code.

/**
 * Expose `Keyboard`
 */
exports = module.exports = Keyboard;
inherits(Keyboard, Trigger);

```




 

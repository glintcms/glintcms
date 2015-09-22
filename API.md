# API

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

The design of the module's `api` was chosen to get a good balance between "easy to use" and  "easy to extend".

The `api` of the modules is designed after what we call the `associative-provider` model.

You wouldn't use a `TextBlock` directly, but via a `Block` that holds the `TextBlock`
	
```javascript
//e.g.: Block -> TextBlock
var block = Block(TextBlock()).use(Style());
```

The following `api` document describes how to "use" the `building blocks`, (not how to extend them).

*get/set*
Getter and Setter can be called like this:

```javascript
// set
var place = obj.place('server');

// get
console.log(obj.place());
// --> server

```

*rendering*
rendering (load) is done by default on the server.
editing, saving and deleting is always initiated in the browser.
blocks and widgets can be defined to render in the browser when needed.
however you can also override where the components (blocks and widgets) are rendered all together.
you can use this for example to let everything be rendered on the server,
when the site is being called by a bot, search engine, crawler or the like.


*place*
glint is designed to use on the server as well as in the browser (pick you buzzword for it... universal, polymorph, what ever you like).
With the `place` get/set method, you can define, where you would like the control to be rendered.

These are the available options (strings):
- server
- browser 
- both
- force:server
- force:browser
- force:both


priorities:

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


#wrap

A `wrap` can hold several `container`s and `widget`s as well as other `wrap`s.
Let's call them `controls`. With the wrap, you can define how the `controls` are loaded.
First you can define the default object with the method `defaults`.
With the methods `parallel`, `series` and `eventually`, you can define everything from a simple to a quite sophisticated loading execution.
See [flow-builder](https://github.com/intesso/flow-builder)

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

- editable
- place

- cid

When you set `editable` or `place` on the wrap, the value is also set on all of it`s controls.

`cid` gets/sets the id on the first `container` that the `wrap` holds.


## methods

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

## properties

## methods


# container

## properties

## methods


# block

## properties

## methods


# adapter

## properties

## methods


# trigger

## properties

## methods



 

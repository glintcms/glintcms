# API

It was important during the design of the module's `api`, to come up with something that's easy to extend.
And I think you can say, it is a strength of *GlintCMS*, that it is very flexible and easy to extend.

## providers
If you want to integrate third party modules for example for a new editing experience, you can just create a new `block-provider`, and implement the methods:

- `load`
- `edit`
- `save`

Have a look at e.g.: [glint-block-text](https://www.npmjs.com/package/glint-block-text)

You don't have to worry, how the stuff is being saved, the `adapter` takes care of that.


À propos `adapter`, if you want store your data in another `database`, you can create a new `adapter-provider` and implement the methods:

- `find`
- `load`
- `save`
- `delete`

Check out e.g.: [glint-adapter-fs](https://www.npmjs.com/package/glint-adapter-fs)

Then extend your find queries `adapter.mixin()` with the query language of your new `provider`.


## plugins

Another great opportunity is, that you can create plugins for the different `building-blocks` either with the `use` or `mixin` method.

Actually, much of the functionality of *GlintCMS* is build with plugin modules:

Have a look at the examples:

- [block plugin](https://www.npmjs.com/package/glint-plugin-block-style-editable)
- [adapter plugin](https://www.npmjs.com/package/glint-plugin-adapter-dates)
- [wrap plugin](https://www.npmjs.com/package/glint-plugin-wrap-i18n)


and search npm for [glint-plugin](https://www.npmjs.com/search?q=glint-plugin)


### use

The `use` method takes a function as argument that's called with `this`, and returns `this`, to make it chainable.

It's probably easiest to look at the code, and the usage examples just mentioned before.

```javascript
Block.prototype.use = function(plugin) {
  plugin(this);
  return this;
};
```

### mixin

If the plugin is just a `function`, or a couple of functions, you can insert them with the `mixin` mechanism.
It takes an object with the required functions as key, value pairs, and it returns `this`, to make it chainable.

Maybe the code is even more understandable than the explanation :-):

```javascript
Block.prototype.mixin = function(mixins) {
  var self = this;
  Object.keys(mixins).forEach(function(key) {
    self[key] = mixins[key];
  });
  return this;
};
```

### events

Plugins get a lot of power from the events.


## care and share

> And don't forget to share your extensions!

> Just name the modules starting with `glint-` and publish them on [npm](https://www.npmjs.com).


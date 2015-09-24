# extend glintcms

It was important during the design of the module's `api`, to come up with something that's easy to extend.
And I think you can say, it is a strength of *GlintCMS*, that it is very flexible and easy to extend.

# providers

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


# plugins

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


# module development

Developing modules for glint is not difficult. It does not force you using specific libraries, or patterns.

However there is a few things you should consider:


## general considerations
- make sure your module runs on the server as well as in the browser (if it is not intended otherwise).
- The main `glint building blocks` right now don't use ES2015, ECMAScript 6 or ES6 or how ever you call the latest JavaScript version.
  This helps to keep things simple and avoid running into compatibility pitfalls especially with the different browsers.
- it is often better to keep the modules simple for every one, we don't use anything that needs to be compiled like e.g. less, sass or coffeescript.
- if you still use something that needs to be compiled, anything like e.g. less, sass or coffeescript, do it in your build, and provide the compiled js, css.
- also your module should be minimal and not depend on other large modules.
- know what you `require('...')`
- substack wrote a really helpful guide, how you can develop components with browserify: [browserify-handbook](https://github.com/substack/browserify-handbook#reusable-components)

## structure
- there is no restrictions on how to structure a glint module
- the good thing about small modules is, you don't need to care about structure, but size
- try to split it up into different modules, when it becomes too big
- a flat folder structure will do most of the time

## common modules
If you can, it makes sense to depend on a few common modules instead of many different ones.
This list contains modules (dependencies) that are used in many of the modules:

```javascript
{
 "clone": "^1.0.2",
 "debug": "^2.2.0",
 "defaults": "^1.0.2",
 "dot": "^1.0.3",
 "ejs": "^2.3.4",
 "is-browser": "^2.0.1",
 "page": "git://github.com/intesso/page.js.git",
 "utils-merge": "^1.0.0"
}
```

## style
> style is a very personal thing
- the glint modules use 2 spaces indent, and yes they use semicolons
- if your module does it differently, that's fine
- the style should be consistent within a module (check before submitting a pull request).

# care and share
> And don't forget to share your extensions!
> Just name the modules starting with `glint-` and publish them on [npm](https://www.npmjs.com).


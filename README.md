<table>
<tr>
<td>
<img src="glintcms-logo.png" alt="GlintCMS Logo">
</td>
<td>

<h1>GlintCMS Documentation</h1>
<pre>        The invisible Content Management System                 </pre>
<p>
GlintCMS is an in-page, overlay style WYSIWYG (What you see is what you get) CMS. What this means is that to edit a page you simply navigate to the page as you would when you browse your website. Once you arrive at a page you use the overlaid main menu to perform tasks or you simply hover over the main section of the page to edit content or change page settings.
</p>
<p>
The building blocks of GlintCMS are designed that you can build Web-Sites as well as Web-Applications.
</p>

</td>
</tr>
</table>


**Stability**

Probably between experimental and stable. It is used e.g. on [www.intesso.com](http://intesso.com).
The best way to get started is, read along and install a starter project e.g. [glintcms-starter-intesso](https://github.com/glintcms/glintcms-starter-intesso).

You can use it out of the box for small to medium sites.


# design goals and principles

 - it is a [GlintApp](https://github.com/glintapp/glintapp) implementation
 - runs in node.js and in the browser with browserify
 - universal aka (isomorphic) javascript
 - modular | collection of reusable, replacable modules | follows unix philosophy
 - fast, no magic for SEO (like screen scraping of own content with headless browser)
 - for WebSites as well as for WebApps
 - Clean Code over Convention over Configuration
 - mostly framework agnostic
 - easy to learn
 - flexible
 - reusable
 - inline editing
 - minimal management backend
 - minimal build tasks
 - minimal configuration
 - no dynamic require
 - no magic
 - only simple abstractions


# technology

It is written purely in JavaScript and runs on [node.js](https://nodejs.org) and in the browser thanks to [browserify](http://browserify.org) and makes use of many great modules  from [npm](https://www.npmjs.com).

- It should work with all of the newer browsers.
- [Express](http://expressjs.com) is used on the server side
- It runs on Linux and Mac OS, Windows is not yet supported (any help appreciated)
- Tested with node 0.12.7 aaaaand trörööööö 4.y.z

## BYODB

**Bring your own Database**

GlintCMS uses `glint-adapter` for abstracting the data storage. [glintcms-starter-intesso](https://github.com/glintcms/glintcms-starter-intesso) for example uses no database at all, but the file system `fs` instead.

You can write your own adapter provider for the database storage of your choice with minimal effort.


## BYOTE

**Bring your own Template Engine**

GlintCMS works with any template engine that supports node.js and runs in the browser as well.

The existing modules use [doT.js](http://olado.github.io/doT), [ejs](https://www.npmjs.com/package/ejs) and [jade](https://www.npmjs.com/package/jade).

Just keep in mind that not all of them perform as good. We made the best experience with [doT.js](http://olado.github.io/doT).


# use cases
Because *GlintCMS* is designed from the very beginning,
to run efficient on the server as well as in the browser, it is a really good fit for many things.
- Web Application (Single Page Applications)
- Web Sites / Platforms (Heavy server side rendering)


# building blocks

```
                                                                         
   +--------------+        +--------------+        +--------------+
   |              |        |              |        |              |
   |    wrap      | +----> |  container   | +----> |    block     |
   |              |        |              |        |              |
   +--------------+        +--------------+        +--------------+

          +                       +
          |                       |
          v                       v

   +--------------+        +--------------+        + + + + + + +  +
   |              |        |              |
   |    widget    | +----> |   adapter    | +----> +    cache     +
   |              |        |              |
   +--------------+        +--------------+        +  + + + + + + +
                                                                         

```
created with [asciiflow](http://asciiflow.com)

## block
Blocks are the heart of everything that is editable in *GlintCMS*.

A block provider can be a plain text `glint-block-text`, or an image or a rich text e.g. `glint-block-ckeditor` etc.

The `block` itself (`glint-block`) is a unifying interface for the block providers.

It is the block providers responsibility to display its data and to turn it into something editable, when switching to edit mode.

They must basically implement the following sync functions:

```javascript
load(content)
edit()
var content = save()
```

## adapter
The `adapter` (`glint-adapter`) is the unifying interface for the adapter providers like e.g. `glint-adapter-fs` or `glint-adapter-elasticsearch`.

The adapter provider is responsible for storing and retrieving the data/content.

It only solves access to single types, entities, documents or tables (the terms very with the providers technology, let's stick with the term `type`').

It does not combine instances of different types (joins), that's up to the

An adapter provider must implement these async functions:

```javascript
load(.., cb)
save(.., cb)
delete(.., cb)
find(.., cb)
```

## cache
Is optional and is not yet implemented.


## container
The `container` (`glint-container`) holds the different blocks and orchestrates nifty details, like
- what should be loaded (rendered) on the server and what in the browser.
- switching the blocks into the edit mode `edit` and back (`save` or `load` on cancel)
- passing the data between the `adapter` and the `block`s.
- a container has got one adapter and most likely has several `block`s.

If you don't have any editable content on your page, you don't need a container.


## widget
A `widget` (`glint-widget`) can be useful for displaying content, that does not need to be editable in this place.

As an example, you could use it to display the three latest blog entries on the first page.

A `widget` implementation needs to implement the functions:
```javascript
data(cb) // async, optional, if you don't need to load data in an async way
render(fn) // sync, returns the rendered content
```

The `widget` itself exposes the `load` function to integrate with the `wrap` loading mechanism.

## wrap
The `wrap` (`glint-wrap`) wraps it all up.
- it can have several `container`s, `widget`s and `wrap`s.
- it also is involved in what should be loaded (rendered) on the server and what in the browser.



# next steps

- Try out the starter project [glintcms-starter-intesso](https://github.com/glintcms/glintcms-starter-intesso). (it should only take a couple of minutes to get going).
- Have a look at the [API Documentation](API.md)
- As well as the [Extend Documentation](EXTEND.md)
- Have a look at the code of the modules, shout out if you need help.



# stay tuned

... for more detailed usage scenarios.

# author

Andi Neck | [@andineck](https://twitter.com/andineck) | andi.neck@intesso.com | intesso


# get involved

Any Feedback is highly appreciated.
Please create an [Issue](https://github.com/glintcms/glintcms/issues/new) or [PR](https://github.com/glintcms/glintcms/pulls).
I'm happy to add you as a comitter too.


# license

MIT

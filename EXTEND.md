# API

It was important during the design of the module's `api`, to come up with something that's easy to extend.
And I think you can say, it is a strength of *GlintCMS*, that it is very flexible and easy to extend.


If you want to integrate third party modules for example for a new editing experience, you can just create a new `block-provider`, and implement the methods:

    `load`, `edit`, `save`
    check out e.g.: [glint-block-text](https://www.npmjs.com/package/glint-block-text)

You don't have to worry, how the stuff is being saved, the `adapter` takes care of that.


À propos `adapter`, if you want store your data in another `database`, you can create a new `adapter-provider` and implement the methods:

    `find`, `load`, `save`, and `delete`
    check out e.g.: [glint-adapter-fs](https://www.npmjs.com/package/glint-adapter-fs)

Then extend your find queries with the query language of your new `provider`.




> And don't forget to share your extensions!

> Just name the modules starting with `glint-` and publish them on [npm](https://www.npmjs.com).


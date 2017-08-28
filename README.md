# Backbone.Inline.Template

[Quick intro][quick-intro] – [Why?][use-case] – [Setup][setup] – [Framework integration][framework-integration] – [Selecting templates][selecting-templates] – [Cache][cache]<br>
[Limitations][fine-print-limitations] – [Alternative][set-element-as-alternative] – [Build and test][build]

It seems that quite a few people would like to see fully-formed templates in Backbone: templates which contain the root element of the template – the `el`, in Backbone lingo – and not just the inner HTML of that element. It's been a popular feature request [for Backbone][backbone-546], and [for frameworks built on it][marionette-2357]. 

Backbone.Inline.Template is that feature.

## In a nutshell

Assume you want to create a view with an `el` of `<p id="foo" class="bar">`. But you don't want to set up the `el` in your Javascript code, with Backbone's `tagName`, `id` and `className`, as you'd normally have to do. Instead, the `el` should be defined inside the template itself, with HTML markup.

##### Classic Backbone

Here's how:

```html
<script id="some-template" type="text/x-template" data-el-definition="inline">
    <p id="foo" class="bar">
        Lots of fabulous <%= content %> here.
    </p>
</script>
```

Note the data attribute `data-el-definition="inline"`. That's how you tell your application that the `el` is defined inline in the template. (You can [change][selecting-templates] how such templates are marked.)

When you create a view with that template, its `el` will be the element you expect: `<p>`. It is already set up and available when `initialize()` is called, just like any other Backbone `el`.

That's it, really. For your own processing and rendering, proceed as you like. However, you can boost performance and avoid accessing the DOM for retrieving the template. Fetch the template from the [cache of Backbone.Inline.Template][cache] instead. E.g.

```js
// Tell Backbone.Inline.Template which template compiler function you use.
Backbone.InlineTemplate.custom.compiler = _.template;

// Pull the template content from the cache.
var BaseView = Backbone.View.extend( {
  initialize: function () {
    this.template = this.inlineTemplate.getCachedTemplate().compiled;
  }
} );
```

For more on the cache, [see below][cache].

##### Frameworks like Marionette

Now, let's assume that you are using a Backbone framework which takes care of template loading and rendering, but doesn't support `el` inlining. One such framework is Marionette. How do you undercut the framework magic and make it process your template format?

With a single statement. This one:

```js
Backbone.InlineTemplate.updateTemplateSource = true;
```

Now Backbone.Inline.Template sets up your `el` as it did before, but it also modifies the original template in place. By the time your framework kicks in, the HTML defining the `el` is no longer part of the template content. Instead, the framework sees a template it can understand: just the part which provides the inner HTML of the `el`.

For which Backbone frameworks does that work? Presumably for most. Backbone.Inline.Template does its job at a very early stage during the life cycle of a view. So the chances are excellent that your framework of choice picks up the changes which Backbone.Inline.Template has made to the templates.

You'll find the details on framework integration and the `updateTemplateSource` option [below][framework-integration].

##### And finally...

Please have a brief look at the [fine print and limitations][fine-print-limitations], too.

## Why use it?

There are several reasons for wanting to see the `el` of a view inside a template.

**Markup should stay out of Javascript.**
  
The principle itself doesn't really need explaining, it is a basic tenet of clean design. Using Backbone view properties like `tagName`, `className` etc is in direct violation of that principle. 
  
However, there is more than one way to achieve a separation of markup and Javascript code. If that separation is all you want from fully self-contained templates, consider using [Backbone.Declarative.Views][] instead, which does the job very well. In fact, Backbone.Inline.Template is just a plug-in for it, adding an additional layer of template wrangling.

A definite plus of Backbone.Declarative.Views is that it works with zero configuration in virtually any Backbone application. Inlining the `el` might be more intuitive with Backbone.Inline.Template, and some might favour it for reasons of style. But [Backbone frameworks typically require][framework-integration] the templates to be modified in place, which adds a bit of complexity. If you can do without, why not [do it][Backbone.Declarative.Views]. In any event, you have a choice.

**The `el` must be inline to make templates work on the server.**

If you render your HTML on the server and slap on a Backbone application as a client-side progressive enhancement, you probably want to use the exact same templates on both ends. And your server-side app might require you to define the template root element, the `el`, inline. 

Backbone.Inline.Template is your new best friend in that case, and it saves you from the worries which burden [other approaches][set-element-as-alternative].

**The `el` itself, and not just its content, depends on template variables.**

That, too, is a [legitimate requirement][marionette-2357-comment-designermonkey]. But in that case, sadly, Backbone.Inline.Template is [not for you][no-el-vars]. Have a look at its [limitations][fine-print-limitations] to see why. [Alternatives exist][set-element-as-alternative], though at a cost.

## Dependencies and setup

Backbone.Inline.Template depends on [Backbone.Declarative.Views][] and the Backbone stack: [Backbone][], [Underscore][], and [jQuery][] or one of its replacements. Include backbone.inline.template.js after that lot.

If you use other components which extend Backbone.View, load these components after Backbone.Inline.Template.

Backbone.Inline.Template augments the Backbone.View base type, so its functionality is available in every view throughout your code.

When loaded as a module (e.g. AMD, Node), Backbone.Inline.Template does not export a meaningful value. It solely lives in the Backbone namespace.

### With Marionette

Load backbone.inline.template.js after [Marionette][]. Do it in this order: Marionette, Backbone.Declarative.Views, Backbone.Inline.Template.

If you use AMD, please be aware that Marionette is not declared as a dependency in the AMD build of Backbone.Inline.Template, nor in Backbone.Declarative.Views. Declare it yourself by adding the following shim to your config:

```javascript
requirejs.config( {
    shim: {
        'backbone.declarative.views': ['marionette']
    }
} );
```

Note that the shim makes Marionette a [dependency of Backbone.Declarative.Views][Backbone.Declarative.Views-setup-marionette], not of Backbone.Inline.Template.

### Download, Bower, npm

The stable version of Backbone.Inline.Template is available in the `dist` directory ([dev][dist-dev], [prod][dist-prod]). If you use Bower, fetch the files with `bower install backbone.inline.template`. With npm, it is `npm install backbone.inline.template`.

## Framework integration: `updateTemplateSource`

Backbone leaves it up to you how you deal with your templates. Frameworks built on top of Backbone, however, usually have specific expectations about what is in the template, and what isn't. The `el` of the view usually isn't.

So if your templates contain the `el` of the view initially, it has to be gone by the time the framework processes the template. Backbone.Inline.Template can sort that out for you. Set the configuration option 

```js
Backbone.InlineTemplate.updateTemplateSource = true;
```

and your original templates are modified. Once Backbone.Inline.Template is done with them, they just contain the inner HTML of the `el`.

You also need that option if you are working with pre-existing code which expects "normal" templates (ie, templates without the `el` inside).

##### How does it work?

Let's get the magic out of this process and look at an example. Suppose your original template looks like this:

```html
<script id="some-template" type="text/x-template" data-el-definition="inline">
    <p id="foo" class="bar">
        Lots of fabulous <%= content %> here.
    </p>
</script>
```

As soon as you instantiate a view with that template, Backbone.Inline.Template transforms the template into:

```html
<script id="some-template" type="text/x-template" 
    data-tag-name="p" data-id="foo" data-class-name="bar">
    Lots of fabulous <%= content %> here.
</script>
```

The transformed version is presented to the framework when it processes the template. As you can see, the `el` tag is no longer part of the template content – just the inner HTML of the `el` is left. 

Meanwhile, the `el` has morphed into a set of data attributes on the template element (`data-tag-name` etc). Backbone frameworks happily ignore the data attributes. However, they are picked up by [Backbone.Declarative.Views][Backbone.Declarative.Views], the engine behind Backbone.Inline.Template, whenever you instantiate a new view with the template. The `el` of the view is set up accordingly.

##### Limitations of `updateTemplateSource`

The `updateTemplateSource` option is global. If you use it, the transformation applies to all templates which are [marked for processing][selecting-templates] by Backbone.Inline.Template. You can't transform templates on a case-by-case basis.

As you have seen, the transformed template is written back to the original template element (`<script>` or `<template>`). So an actual template element needs to be present in the DOM. With `updateTemplateSource`, you can't use a raw HTML string as a template (you'll get an error if you do). Provide a node.

## Selecting templates for processing

By default, Backbone.Inline.Template does not act on each and every template. A template is recognized as having an inline `el` only if a specific data attribute is set on the template element: `data-el-definition: "inline"`. All other templates are left alone.

If that doesn't suit your needs, you can change the way templates are selected for processing. Do it by overriding 

```js
Backbone.InlineTemplate.hasInlineEl 
```

with a custom function.

##### Providing a function for `hasInlineEl`

Your `hasInlineEl` function is called with a single argument: the template, as a jQuery object. The function can examine the template and must return a boolean.

For instance, assume you want templates to be handled by Backbone.Inline.Template if they have an attribute of `type="text/x-inline-template"`. That can be achieved with

```js
Backbone.InlineTemplate.hasInlineEl = function ( $template ) {
    return $template.attr( "type" ) === "text/x-inline-template";
};
```

In order to treat all templates as having an inline `el`, the function just has to return true:

```js
Backbone.InlineTemplate.hasInlineEl = function () { return true; };
```

A word of caution: Please don't define a custom `hasInlineEl` just for using another data attribute. If you want a data attribute to select your templates, use the one which is provided out of the box (`data-el-definition`).

Why? Because the jQuery methods for data attributes would interfere with the handling of your custom attribute. These methods are buggy and inconsistent across jQuery versions. Backbone.Inline.Templates takes care of the issues for the data attributes which are built in, but custom ones would not be covered by that.

## Cache

Backbone.Inline.Template is a plug-in for [Backbone.Declarative.Views][], which does all the real work. Unsurprisingly, both share the same template cache. Backbone.Inline.Template just provides a number of aliases to access that cache.

In the documentation of Backbone.Declarative.Views, you'll find more [about the cache itself][Backbone.Declarative.Views-cache]. 

Used with Backbone.Inline.Template, the cache captures the state of the template after the `el` has been extracted. So when a template [has been processed][selecting-templates] by Backbone.Inline.Template and you [query the template cache][Backbone.Declarative.Views-cache-access], 

- the `html` property of the [cached object][Backbone.Declarative.Views-cache-entry] refers to the HTML _inside_ the `el`, not the HTML of the entire template (which would have included the inline `el`)
- the `compiled` property, likewise, hands you the template function for the HTML _inside_ the `el`, not the entire template. (You need to have [defined a compiler function][Backbone.Declarative.Views-compiler] for that, of course).

The following aliases are available for the cache of Backbone.Declarative.Views.

- Tell Backbone.Inline.Template [which template compiler function to use][Backbone.Declarative.Views-compiler]:

  ```js
  // E.g. using _.template as your compiler:
  Backbone.InlineTemplate.custom.compiler = _.template;
  // Alias of 
  Backbone.DeclarativeViews.custom.compiler = _.template;
  ```

- Define a [custom template loader][Backbone.Declarative.Views-loader]:

  ```js
  Backbone.InlineTemplate.custom.loadTemplate = function () { /*...*/ };
  // Alias of 
  Backbone.DeclarativeViews.custom.loadTemplate = function () { /*...*/ };
  ```

- [Retrieve a cached template][Backbone.Declarative.Views-cache-access] in the context of a view:

  ```js
  initialize: function () {
    cachedTemplate = this.inlineTemplate.getCachedTemplate();
    // Alias of 
    cachedTemplate = this.declarativeViews.getCachedTemplate();
  }
  ```

- [Retrieve a cached template][Backbone.Declarative.Views-cache-access] independently of a view, by selector:

  ```js
  cachedTemplate = Backbone.InlineTemplate.getCachedTemplate( "#template" );
  // Alias of 
  cachedTemplate = Backbone.DeclarativeViews.getCachedTemplate( "#template" );
  ```

- [Clear a cached template][Backbone.Declarative.Views-clear-cache] in the context of a view:

  ```js
  someView.inlineTemplate.clearCachedTemplate();
  // Alias of 
  someView.declarativeViews.clearCachedTemplate();
  ```

- [Clear a cached template][Backbone.Declarative.Views-clear-cache] independently of a view, by selector:

  ```js
  Backbone.InlineTemplate.clearCachedTemplate( "#template" );
  // Alias of 
  Backbone.DeclarativeViews.clearCachedTemplate( "#template" );
  ```

- [Clear the cache][Backbone.Declarative.Views-clear-cache]:

  ```js
  Backbone.InlineTemplate.clearCache();
  // Alias of 
  Backbone.DeclarativeViews.clearCache();
  ```

## Fine Print and Limitations

The idea behind Backbone.Inline.Template is that it should "just work" with a minimum of intervention. But there are things to keep in mind, and some use cases are beyond its scope. 

### Just one top-level tag inside the template

The top-level HTML element tag inside a template defines the `el` of the view, and there can just be one such `el`. **You must abide by that rule**.

Multiple top-level HTML elements inside a template will confuse the hell out of the template loader and lead to unexpected results, such as an empty template, invalid and disfigured HTML, or an error while creating the view.

So **don't** do this:

```html
<script id="some-template" type="text/x-template" data-el-definition="inline">
    <section id="main" class="bar">
        <p><%= content %></p>
    </section>
    <footer> <!-- WRONG, second top-level element! -->
        <%= boilerplate %>
    </footer>
</script>
```

However, you can add top-level HTML comments to the template. They are simply ignored.

```html
<script id="some-template" type="text/x-template" data-el-definition="inline">
    <!-- 
        A top-level comment like this doesn't break stuff. 
        It simply won't show up in the HTML output. 
    -->
    <section id="main" class="bar">
        <p><%= content %></p>
    </section>
</script>
```

### Don't use template variables in the `el` itself

Setting up the `el` of a view with template variables is a [valid use case][marionette-2357-comment-designermonkey], but unfortunately, Backbone.Inline.Template doesn't support it. 

Backbone creates the `el` as soon as the view is instantiated, even before `initialize()` is called (and [for a reason][backbone-546-comment-jashkenas]). The `el` already exists before any processing of template data can take place. The data may not even be available that early. In any event, it can't be used for the `el`.

So **don't** do this:

```html
<script id="some-template" type="text/x-template" data-el-definition="inline">
    <p class="<%= el_class %>"> <!-- WRONG, template var in el! -->
        <%= content %>          <!-- This part is OK, of course -->
    </p>
</script>
```

If you need to support template variables in the `el` itself, you have to go down a different route and swap out the `el` on render, when the data is there.

Backbone.Inline.Template can't help you with it and is not useful in that scenario. You can find an [alternative approach below][set-element-as-alternative], and additional inspiration e.g. [on Stack Overflow][so-set-element]. But [be aware of the pitfalls][set-element-cons].

### Leave the `el` in place when rendering

It may be quite obvious, but maybe warrants at least a mention: The `el` is already in place when you render, so don't throw it away needlessly. Nothing breaks if you do ([well, mostly][set-element-cons]), but you take a performance hit and have to face all sorts of event binding woes for no reason.

So you want to keep the `el`. What does that imply? You have defined the `el` inside the template. If you compile the template in its original form, the `el` element shows up in the result. You have to remove the `el` element from the template and only inject the remainder into the existing `el` of the view.

There is no need to transform the template yourself, though. Backbone.Inline.Template can remove the `el` for you and redefine the template with what is left. Use the [`updateTemplateSource` option][framework-integration] for that – you may have to turn it on anyway if [your framework expects it][framework-integration].

Or better still, don't even do that, don't (re-)compile the template yourself. There is a much easier and more efficient option: Pull the compiled template from the [built-in cache][cache] instead. There, the duplicate `el` tag [is already removed][cache] for you.

### Boolean `el` attributes

[Boolean attributes][spec-boolean-attributes] of an `el`, like [`hidden`][mdn-hidden], are fully supported. That includes the short notation with just the attribute name, without a value assignment (`<p hidden>`).

### How to use template literals, instead of a selector

For your templates, you don't always have to use a template element in the DOM, chosen with a selector. Instead, you can pass in a complete template as a string. But how do you mark it for processing by Backbone.Inline.Template? Obviously, you can't [set a data attribute][selecting-templates] on a string.

You have two options. For one, you can simply make sure that _all_ templates are processed by Backbone.Inline.Template. That includes template strings. A [custom `hasInlineEl` function][custom-marker-function] does the trick.

Alternatively, you can add a special HTML comment to your template string which selects it for processing:

```html
<!-- data-el-definition="inline" -->
``` 

The comment can be placed anywhere in your template. It is governed by the same rules as [special comments][Backbone.Declarative.Views-template-string] in Backbone.Declarative.Views.

You can't use the special comment if you have defined [a custom `hasInlineEl` function][custom-marker-function]. Only the default data attribute is recognized in the comment (`data-el-definition="inline"`). 

Please be aware that you cannot process template strings with Backbone.Inline.Template if you also configure it to update the template source. The `updateTemplateSource` setting [must be off][updateTemplateSource-limitations].

## Alternative approach: `setElement()`

Backbone.Inline.Template and its cousin, [Backbone.Declarative.Views][], do their respective jobs in an unobtrusive way which is compatible with pretty much everything. But [there are limits][no-el-vars] to what they can do. Once once you hit these constraints, you need to find alternatives.

A well-known approach makes use of [`setElement()`][backbone-setelement], which is native to Backbone views, when rendering a view.

##### The pattern

With `setElement()`, you throw away the existing `el` which is provided by Backbone, and replace it with your own when you render the view. The basic pattern looks like this:

```js
render: function () {
    var $existingEl = this.$el,

        // Fill in the template vars and return the HTML of the view,
        // INCLUDING the outer `el`. It doesn't matter here how you 
        // get that done. The evaluated template is a string of HTML.
        evaluatedTemplate = this.template( this.model.attributes ),

        $replacementEl = $( evaluatedTemplate );
  
    // Make the view reference the new node as its `el`
    this.setElement( $replacementEl );

    // Replace the existing `el` node in the DOM with the new one 
    $existingEl.replaceWith( $replacementEl );
    
    // ... other stuff 
}
```

##### The pros

The `setElement()` pattern is unbeatable in one respect: you can use template variables to set up the `el` of a view.

For instance, assume you need an individual `id` for each `el`. For some reason, you can't set the `id` in your Javascript code with the `id` property of the view. The `id` has to be a template variable, along the lines of 

```html
<li id="item-{{itemNumber}}">
```

To incorporate the template data, you have to recreate the `el` when the data is ready. The pattern above allows you to do it.

That's not exactly a long list of advantages, but for that one use case, `setElement()` is the way to go.

##### The cons

Relying on `setElement()` has a lot of drawbacks. None of these matter if your use case forces you to recreate the `el` (see [the pros][set-element-pros]). But there is a strong case against using `setElement()` if you have a choice.

The challenges broadly fall into two categories: compatibility and implementation detail. Compatibility, both with existing and future code, is probably the trickier of the two.

- Defining the `el` in the template may be convenient and even necessary, but it breaks with Backbone conventions. For the view code you write yourself, that won't be much of an issue. But as soon as you pull third-party Backbone extensions like Marionette into a project, templates of that kind no longer work. You probably have to rewrite the render method of your would-be framework, at the very least, to adapt it.

  Backbone.Inline.Template does its processing very early during the life cycle of a view. That's why it is able to [present a cleaned-up, conventional template][framework-integration] to Backbone frameworks. With `setElement()`, however, you don't have that option. It forces you to keep the original template around until render time. 

- Backbone is not very prescriptive about how it is to be used. It gives developers a lot of flexibility, and expects them to figure out their own way of handling views in particular. As a result, there are many different approaches and implementations.

  Replacing a view's `el` with `setElement()`, on the other hand, requires very specific steps at a very specific time, and may clash with the approach chosen in the application you work on. Expect problems when integrating it into legacy code.

The other group of issues is about pitfalls when rendering views with `setElement()`. The devil here is in the details. These issues are solvable, but require attention (and additional code).

- `setElement()` rebinds events set up with the `events` hash of the view, but not events [set up in any other way][setelement-event-rebinding-jsbin]. You have to remember taking care of them.

- `setElement()` screws up events in nested views. Fortunately, a [good pattern exists][ian-storm-taylor-rendering-views] to deal with it. It is important to be aware of the issue, though.

- Recreating the `el` and moving the delegated events around has a performance cost which matters as the number of render calls goes up (e.g. large lists).

That's quite a few things to consider, and the list is not even complete. [Long discussions][backbone-546] have taken place about defining the `el` inline, inside a template. The problems associated with re-assigning the `el` during render have come up repeatedly in these discussions. The Backbone team [decided against][backbone-546-comment-jashkenas] this approach back in 2012. For Marionette, [the debate][marionette-2357] is still going on – it has already lasted for years. 

## Build process and tests

If you'd like to fix, customize or otherwise improve the project: here are your tools.

### Setup

[npm][] and [Bower][] set up the environment for you.

- The only thing you've got to have on your machine is [Node.js]. Download the installer [here][Node.js].
- Open a command prompt in the project directory.
- Run `npm install`. (Creates the environment.)
- Run `bower install`. (Fetches the dependencies of the script.)

Your test and build environment is ready now. If you want to test against specific versions of Backbone, edit `bower.json` first.

### Running tests, creating a new build

#### Considerations for testing

To run the tests on remote clients (e.g. mobile devices), start a web server with `grunt interactive` and visit `http://[your-host-ip]:9400/web-mocha/` with the client browser. Running the tests in a browser like this is slow, so it might make sense to disable the power-save/sleep/auto-lock timeout on mobile devices. Use `grunt test` (see below) for faster local testing.

#### Tool chain and commands

The test tool chain: [Grunt][] (task runner), [Karma][] (test runner), [Mocha][] (test framework), [Chai][] (assertion library), [Sinon][] (mocking framework). The good news: you don't need to worry about any of this.

A handful of commands manage everything for you:

- Run the tests in a terminal with `grunt test`.
- Run the tests in a browser interactively, live-reloading the page when the source or the tests change: `grunt interactive`.
- If the live reload bothers you, you can also run the tests in a browser without it: `grunt webtest`.
- Run the linter only with `grunt lint` or `grunt hint`. (The linter is part of `grunt test` as well.)
- Build the dist files (also running tests and linter) with `grunt build`, or just `grunt`.
- Build continuously on every save with `grunt ci`.
- Change the version number throughout the project with `grunt setver --to=1.2.3`. Or just increment the revision with `grunt setver --inc`. (Remember to rebuild the project with `grunt` afterwards.)
- `grunt getver` will quickly tell you which version you are at.

Finally, if need be, you can set up a quick demo page to play with the code. First, edit the files in the `demo` directory. Then display `demo/index.html`, live-reloading your changes to the code or the page, with `grunt demo`. Libraries needed for the demo/playground should go into the Bower dev dependencies, in the project-wide `bower.json`, or else be managed by the dedicated `bower.json` in the demo directory.

_The `grunt interactive` and `grunt demo` commands spin up a web server, opening up the **whole project** to access via http._ So please be aware of the security implications. You can restrict that access to localhost in `Gruntfile.js` if you just use browsers on your machine.

### Changing the tool chain configuration

In case anything about the test and build process needs to be changed, have a look at the following config files:

- `karma.conf.js` (changes to dependencies, additional test frameworks)
- `Gruntfile.js`  (changes to the whole process)
- `web-mocha/_index.html` (changes to dependencies, additional test frameworks)

New test files in the `spec` directory are picked up automatically, no need to edit the configuration for that.

## Release notes

### 1.0.1

- Fixed template source failing to update if all `el` properties are defined in a Backbone view
- Updated Backbone.Declarative.Views dependency to 3.1

### 1.0.0

- Removed the separate AMD/Node builds in `dist/amd`. Module systems and browser globals are now supported by the same file, `dist/backbone.inline.template.js` (or `.min.js`)
- Simplified AMD shim for using Marionette
- Version is exposed in `Backbone.InlineTemplate.version`

### v0.3.0

- Completed test suite
- Fixed handling of `el` attributes with empty string values, are no longer dropped
- Updated Backbone.Declarative.Views dependency to 3.x

### v0.2.1

- Added matcher tests
- Fixed matcher handling of multi-line `el` tags
- Fixed matcher handling of omitted slash in self-closing `el` tags
- Fixed matcher handling of top-level HTML comments which contain tags 

### v0.2.0

- Removed `removeInlineElMarker()`
- Renamed `updateOriginalTemplates` option to `updateTemplateSource`

### v0.1.0

- Initial public release

## License

MIT.

Copyright (c) 2016 Michael Heim.

Code in the data provider test helper: (c) 2014 Box, Inc., Apache 2.0 license. [See file][data-provider.js].

[quick-intro]: #in-a-nutshell
[use-case]: #why-use-it
[setup]: #dependencies-and-setup
[framework-integration]: #framework-integration-updatetemplatesource
[updateTemplateSource-limitations]: #limitations-of-updatetemplatesource
[selecting-templates]: #selecting-templates-for-processing
[custom-marker-function]: #providing-a-function-for-hasinlineel
[cache]: #cache
[fine-print-limitations]: #fine-print-and-limitations
[no-el-vars]: #dont-use-template-variables-in-the-el-itself
[set-element-as-alternative]: #alternative-approach-setelement
[set-element-pros]: #the-pros
[set-element-cons]: #the-cons
[build]: #build-process-and-tests

[backbone-546]: https://github.com/jashkenas/backbone/issues/546 "Backbone Issue 546: Don't wrap views if using templates"
[backbone-546-comment-jashkenas]: https://github.com/jashkenas/backbone/issues/546#issuecomment-3604746 "Backbone Issue 546: Don't wrap views if using templates – Comment jashkenas"
[marionette-2357]: https://github.com/marionettejs/backbone.marionette/issues/2357 "Marionette Issue 2357: Use template as el"
[marionette-2357-comment-designermonkey]: https://github.com/marionettejs/backbone.marionette/issues/2357#issuecomment-75448937 "Marionette Issue 2357: Use template as el – Comment designermonkey"
[ian-storm-taylor-rendering-views]: https://ianstormtaylor.com/rendering-views-in-backbonejs-isnt-always-simple "Rendering Views in Backbone.js Isn’t Always Simple – Ian Storm Taylor"
[so-set-element]: http://stackoverflow.com/questions/11594961/backbone-not-this-el-wrapping/11598543#11598543 "Backbone, not `this.el` wrapping, Answer by nikoshr – Stack Overflow"
[setelement-event-rebinding-jsbin]: http://jsbin.com/fakadan/8 "Rendering views with setElement: Do DOM events survive? – JS Bin"
[backbone-setelement]: http://backbonejs.org/#View-setElement "setElement() – Backbone.js"
[spec-boolean-attributes]: https://html.spec.whatwg.org/multipage/infrastructure.html#boolean-attributes "Boolean attributes - HTML Living Standard | WHATWG"
[mdn-hidden]: https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/hidden "hidden – HTML | MDN"

[dist-dev]: https://raw.github.com/hashchange/backbone.inline.template/master/dist/backbone.inline.template.js "backbone.inline.template.js"
[dist-prod]: https://raw.github.com/hashchange/backbone.inline.template/master/dist/backbone.inline.template.min.js "backbone.inline.template.min.js"

[data-provider.js]: https://github.com/hashchange/backbone.inline.template/blob/master/spec/helpers/data-provider.js "Source code of data-provider.js"

[Underscore]: http://underscorejs.org/ "Underscore.js"
[jQuery]: http://jquery.com/ "jQuery"
[Backbone]: http://backbonejs.org/ "Backbone.js"
[Marionette]: https://github.com/marionettejs/backbone.marionette#readme "Marionette: a composite application library for Backbone.js"
[Backbone.Declarative.Views]: https://github.com/hashchange/backbone.declarative.views#readme "Backbone.Declarative.Views"

[Backbone.Declarative.Views-setup-marionette]: https://github.com/hashchange/backbone.declarative.views#with-marionette "Dependencies and Setup: With Marionette – Backbone.Declarative.Views"
[Backbone.Declarative.Views-cache]: https://github.com/hashchange/backbone.declarative.views#performance-use-the-template-cache "Performance: Use the template cache – Backbone.Declarative.Views"
[Backbone.Declarative.Views-compiler]: https://github.com/hashchange/backbone.declarative.views#keeping-compiled-templates-in-the-cache "Keeping compiled templates in the cache – Backbone.Declarative.Views"
[Backbone.Declarative.Views-loader]: https://github.com/hashchange/backbone.declarative.views#using-a-custom-template-loader "Using a custom template loader – Backbone.Declarative.Views"
[Backbone.Declarative.Views-cache-access]: https://github.com/hashchange/backbone.declarative.views#reading-template-data-from-the-cache "Reading template data from the cache – Backbone.Declarative.Views"
[Backbone.Declarative.Views-cache-entry]: https://github.com/hashchange/backbone.declarative.views#what-is-on-offer-in-a-cache-entry "What is on offer in a cache entry? – Backbone.Declarative.Views"
[Backbone.Declarative.Views-clear-cache]: https://github.com/hashchange/backbone.declarative.views#clearing-the-cache "Clearing the cache – Backbone.Declarative.Views"
[Backbone.Declarative.Views-template-string]: https://github.com/hashchange/backbone.declarative.views#setting-the-template-property-to-a-template-string-rather-than-a-selector "Setting the template property to a template string rather than a selector – Backbone.Declarative.Views"

[Node.js]: http://nodejs.org/ "Node.js"
[Bower]: http://bower.io/ "Bower: a package manager for the web"
[npm]: https://npmjs.org/ "npm: Node Packaged Modules"
[Grunt]: http://gruntjs.com/ "Grunt: The JavaScript Task Runner"
[Karma]: http://karma-runner.github.io/ "Karma - Spectacular Test Runner for Javascript"
[Mocha]: http://mochajs.org/ "Mocha - the fun, simple, flexible JavaScript test framework"
[Chai]: http://chaijs.com/ "Chai: a BDD / TDD assertion library"
[Sinon]: http://sinonjs.org/ "Sinon.JS - Versatile standalone test spies, stubs and mocks for JavaScript"
[JSHint]: http://www.jshint.com/ "JSHint, a JavaScript Code Quality Tool"

# JavaScript Styleguide

__This guide is specific to javascript for browsers, and may not be appropriate for node applications__

## The basics

### Use soft tabs with a two space indent

This follows the other conventions used within our projects.

### Don't use CoffeeScript

It's an extra abstraction and introduces extra languages for developers to learn. Using Javascript gives us guaranteed performance characteristics and more well known support paths.

### Strict mode

Add the `"use strict";` statement to the top of your scripts to enable [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode).

## Code style

### Writing self-documenting javascript

Unless care is taken to write communicative code, javascript can quickly become extremely difficult to read, especially when excessive jQuery-style chaining is happening. Being mindful of how communicative one's code is to other developers is of course a given as with all languages, but there are certain practices which can help make code easier to follow in javascript.

#### Avoid loose code

__Loose code is code whose function or effect is not described by the name of the containing function.__

Consider the following two examples:

    function init() {
      $('ol.lemmings li a.linky_link').map(function(linky_linky) {
        $(linky_linky).after('<span>blot</span>');
      });
    }

and

    function init() {
      addBlotsToLemmingLinks();
    }

    function addBlotsToLemmingLinks() {
      $('ol.lemmings li a.linky_link').map(function(linky_linky) {
        $(linky_linky).after('<span>blot</span>');
      });
    }

In the second example, there's no need to read into the jQuery bits to see what the purpose of that piece of code is.

#### Use many small functions

Keeping functions small and using loads of them intrinsicly makes javascript more self-documenting. It may even be beneficial to break a function's code into small private nested functions:

    function showLightbox() {
      buildDomElements();
      appendToDom();
      bindEvents();
      show();

      function buildDomElements() {
        // some code
      }

      function appendToDom() {
        // some code
      }

      function bindEvents() {
        // some code
      }

      function show() {
        // some code
      }
    }

_However_ in these cases, it would be best to first think about whether the function is doing too much.

#### Avoid excessive chaining

jQuery and a few other libraries allow you to chain functions together. This can be nice, but if done excessively can make things very hard to read:

    function doSomethingToTheDom() {
      $('.something').find('.something_else').next('label').css('background-color', 'red').click(function() {doSomething();}).click();
    }

In this case, there's no need for the two finders at the start, and it would be far more readable if split out into several lines:

    function doSomethingToTheDom() {
      var $somethingElse = $('.something .something_else');
      var $label = $somethingElse.next('label');
      $label.css('background-color', 'red');
      $label.click(doSomething);
      doSomething();

      function doSomething() {
        // ...
      }
    }

### Patterns

All code should be wrapped in closures and should declare `use strict`.  The GOVUK namespace should be setup in each file to promote portability.

    (function() {
      "use strict";
      window.GOVUK = window.GOVUK || {};

      // stuff
    }());

Unless there's a possibility of another framework taking over `$`, keeping jQuery referenced in the closure scope is not necessary.

#### Namespacing

This depends on project, but unless the project's javascript layer is thick it's recommended to stick to a single namespace object, `GOVUK`.

#### Singleton pattern

Singletons should be defined as raw JavaScript hashes, and if required should do its initialisation in a function called init.

    (function() {
      "use strict";
      window.GOVUK = window.GOVUK || {};

      window.GOVUK.singletonThing = {
        init: function init() {

        },

        anotherFunction: function anotherFunction() {

        },

        // etc.
      };
    }());

#### Constructor pattern

Constructors should follow the prototype pattern as follows:

    (function() {
      "use strict";
      window.GOVUK = window.GOVUK || {};

      function TheThing(options) {
        // some initialisation code
      }

      TheThing.prototype.someFunction = function someFunction() {
        // do some stuff
      };

      window.GOVUK.TheThing = TheThing;
    }());

Defining functions on the prototype as opposed to defining them privately in the constructor exposes them making the objects easier to test. Although in theory you should never test a private method, it's sometimes helpful to do so in JavaScript - particularly when testing objects which are very tightly coupled to the DOM and often don't have any public API.

Defining the constructor in the wrapper function's scope, then assigning it to the namespace improves readability by keeping names shorter.

### Other style points

Favour named arguments in a hash over sequential arguments. [Connascence of naming is a weaker form of connascence than connascence of position][4].

In general, use of anonymous functions should be avoided. Code made up of anonymous functions is more difficult to profile and debug.  Anonymous functions don't report a name to profilers, stack traces and when calling `arguments.callee.caller`, etc.

bad:

    TheThing.prototype.someFunction = function() {
      //do some stuff
    };
    new TheThing().someFunction.name;  //  ==> ''

good:

    TheThing.prototype.someFunction = function someFunction() {
      //do some stuff
    };
    new TheThing().someFunction.name;  //  ==> 'someFunction'

#### Use a `.js-` prefix for JS-only HTML classes

E.g `js-hidden`, `js-tab` etc. This makes it completely transparent what the class is used for within the HTML.

#### Don't apply styles directly inside Javascript

You should only ever apply CSS classes and style from there. Otherwise you risk clobbering user stylesheets and mixing concerns across different code bases. Also see the previous point.

## File structure and namespacing

The following may not be appropriate for all applications, but works for the most part.

Each JavaScript object should be stored in a root folder expressing which part of the app it's going to be used in (if appropriate). Javascript files should then be further categorised according to the scope of that object's effect.

The reason for this further categorisation is to maintain clarity about the scope of each part of the javascript, and to allow for greater confidence in changing / removing code. If something's a 'view' type of script, I can be confident in removing it when the corresponding view template goes.

    ./frontend/views/
    ./frontend/modules/
    ./frontend/helpers/
    ./admin/views/
    ./admin/modules/
    ./admin/helpers/

__Views__ are view-specific scripts and as with the css, their file path and name should exactly mirror the view template or partial it applies to. The name of the script object should reflect the whole view path (the object in `/admin/editions/index.js` should be called `GOVUK.adminEditionsIndex`).

__Modules__ are re-useable things. An example of a module would be the script for a tabbed content block. Modules should not be initialised globaly, and should only be initialised when needed. If a script is only ever going to be used in one place, don't make it a module.

__Helpers__ are scripts which are loaded everywhere (such as a script which prevents forms from being submited twice). Care should be taken not to flood projects with helpers as this can have a negative effect on page load times, especially on mobile devices.


[1]: https://github.com/alphagov/govuk_frontend_toolkit#conditionals
[2]: https://github.com/alphagov/static
[3]: https://github.com/alphagov/govuk_frontend_toolkit
[4]: http://en.wikipedia.org/wiki/Connascence_%28computer_programming%29#Types_of_connascence


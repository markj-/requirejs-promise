# requirejs-promise

This is a plugin for Require.JS that allows modules to resolve asynchronously via jQuery Promises. The current AMD standard does not specify how to deal with asynchronous dependencies, and as such it is perfectly reasonable that Require.JS does not handle them. That said, I still find them unavoidable from time to time, so here we are.

## Status

This plugin seems to behave as expected for Promise objects generated by jQuery, Q and RSVP. It now seems to work okay with the Require.JS Optimiser.

## Licence

I'm putting this out under the Simplified BSD Licence (see LICENCE). Go nuts.

## Requirements

You'll also (obviously) need Require.JS. Likewise, I'll always be targetting the current version.

As this is just a plugin, it doesn't make sense to have further requirements.

I do intend to eventually extend support to other implementations of Promises (with your help) so jQuery won't always be needed.

## Usage

### Basics

You shouldn't need to include this directly in your HTML as a script element.

Configure Require.JS so that it has a path to resolve, for example:

```javascript
    require.config({
        config: {
            paths: {
                // remember to leave out the .js on the end
                'promise': '/path/to/requirejs-promise'
            }
        }
    });
```

Your asynchronous module will probably look something like this:

```javascript
    define('myAsyncModule', ['jquery'], function($) {
        var deferred = new $.Deferred(),
        myModule = {};

        // we'll use AJAX here, but you could do anything asynchronous
        $.ajax({
            url: '//domain/path/to/script',
            complete: function(jqxhr, textstatus) {
                if ($.inArray(jqxhr.status, [0, 200, 204, 304])) {
                    // TODO: do other stuff for successful AJAX
                    myModule.response = jqxhr.responseText;
                    // if your consumers are expecting something, give it to them
                    deferred.resolve(myModule);
                } else {
                    // TODO: do other stuff for failed AJAX
                    deferred.reject(null);
                }
            }
        });

        return deferred.promise();
    });
```

Then, when it comes time to actually load in an asynchronous Promise-based dependency, do something like this:

```javascript
    require(['promise!myAsyncModule'], function(asyncModule) {
        // TODO: do something cool with asyncModule.response
        ...
    });
```

The goal of this little plugin is to make module usage completely seemless. Module resolution is asynchronous within Require.JS anyway, so (assuming everything goes according to plan) nobody downstream should notice that the dependency had a Promise.

### Require.JS Optimisation

See the test/ directory.

index.html is a QUnit test case using non-optimised sources

index2.html is a QUnit test case using optimised (minified) sources

For hints on what needs to be done to get the Optimiser working, see test/build.js and test/main.js


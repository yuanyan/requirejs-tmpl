requirejs-tmpl
==============

Require.js plugin for templates, it's a tooling not a library.

## Fetures

* Pre-compile templates with r.js, easy build
* Templates are compiled into pure JavaScript, fast enough
* Sub-templates, really usefull
* Using ERB-style template delimiters default, easy custom

## Installation

Via Bower:

```sh
bower install requirejs-tmpl
```

## Usage

The tmpl.js plugin will automatically be loaded if the templ! prefix is used for a dependency. Download the plugin and put
it in the app's [baseUrl](http://requirejs.org/docs/api.html#config-baseUrl)
directory (or use the [paths config](http://requirejs.org/docs/api.html#config-paths) to place it in other areas).

You can specify a template file resource as a dependency like so:

```js
define(["jquery", "tmpl!path/to/template1.html", "tmpl!path/to/template2.html"], function($, template1, template2) {
    var html1 = template1({foo:1});
    var html2 = template1({bar:2});
    // balabala...
});
```

Notice the .html suffixes to specify the extension of the file. The
"path/to/" part of the path will be resolved according to normal module name
resolution: it will use the **baseUrl** and **paths** [configuration
options](http://requirejs.org/docs/api.html#config) to map that name to a path.

The template files are loaded via asynchronous XMLHttpRequest (XHR) calls, so you
can only fetch files from the same domain as the web page (see **XHR
restrictions** below).

However, [the RequireJS optimizer](http://requirejs.org/docs/optimization.html)
will inline any tmpl! references with the actual template file contents into the
modules, so after a build, the modules that have tmpl! dependencies can be used
from other domains.

## Configuration

### XHR restrictions

The tmpl plugin works by using XMLHttpRequest (XHR) to fetch the template for the
resources it handles.

However, XHR calls have some restrictions, due to browser/web security policies:

1) Many browsers do not allow file:// access to just any file. You are better
off serving the application from a local web server than using local file://
URLs. You will likely run into trouble otherwise.

2) There are restrictions for using XHR to access files on another web domain.
While CORS can help enable the server for cross-domain access, doing so must
be done with care (in particular if you also host an API from that domain),
and not all browsers support CORS.

So if the tmpl plugin determines that the request for the resource is on another
domain, it will try to access a ".js" version of the resource by using a
script tag. Script tag GET requests are allowed across domains. The .js version
of the resource should just be a script with a define() call in it that returns
a string for the module value.

Example: if the resource is 'tmpl!example.html' and that resolves to a path
on another web domain, the tmpl plugin will do a script tag load for
'example.html.js'.

The [requirejs optimizer](http://requirejs.org/docs/optimization.html) will
generate these '.js' versions of the template resources if you set this in the
build profile:

    optimizeAllPluginResources: true

In some cases, you may want the tmpl plugin to not try the .js resource, maybe
because you have configured CORS on the other server, and you know that only
browsers that support CORS will be used. In that case you can use the
[module config](http://requirejs.org/docs/api.html#config-moduleconfig)
(requires RequireJS 2+) to override some of the basic logic the plugin uses to
determine if the .js file should be requested:

```js
requirejs.config({
    config: {
        tmpl: {
            useXhr: function (url, protocol, hostname, port) {
                //Override function for determining if XHR should be used.
                //url: the URL being requested
                //protocol: protocol of page tmpl.js is running on
                //hostname: hostname of page tmpl.js is running on
                //port: port of page tmpl.js is running on
                //Use protocol, hostname, and port to compare against the url
                //being requested.
                //Return true or false. true means "use xhr", false means
                //"fetch the .js version of this resource".
            }
        }
    }
});
```

### Custom XHR hooks

There may be cases where you might want to provide the XHR object to use
in the request, or you may just want to add some custom headers to the
XHR object used to make the request. You can use the following hooks:

```js
requirejs.config({
    config: {
        tmpl: {
            onXhr: function (xhr, url) {
                //Called after the XHR has been created and after the
                //xhr.open() call, but before the xhr.send() call.
                //Useful time to set headers.
                //xhr: the xhr object
                //url: the url that is being used with the xhr object.
            },
            createXhr: function () {
                //Overrides the creation of the XHR object. Return an XHR
                //object from this function.
            },
            onXhrComplete: function (xhr, url) {
                //Called whenever an XHR has completed its work. Useful
                //if browser-specific xhr cleanup needs to be done.
            }
        }
    }
});
```

### Template Settings

By default, tmpl plugin uses ERB-style template delimiters, change the
following template settings to use alternative delimiters.

```js
requirejs.config({
    config: {
        tmpl: {
            templateSettings:  {
                evaluate    : /<%([\s\S]+?)%>/g,
                interpolate : /<%=([\s\S]+?)%>/g,
                escape      : /<%-([\s\S]+?)%>/g,
                include     : /<%@([\s\S]+?)%>/g
            }
        }
    }
});
```

If ERB-style delimiters aren't your cup of tea, you can change template settings to use different symbols to set off interpolated code. 
Define an interpolate regex to match expressions that should be interpolated verbatim, 
an escape regex to match expressions that should be inserted after being HTML escaped, 
and an evaluate regex to match expressions that should be evaluated without insertion into the resulting string. 
You may define or omit any combination of the three. For example, to perform [Mustache.js](http://github.com/janl/mustache.js#readme) style templating:

```js
requirejs.config({
    config: {
        tmpl: {
            templateSettings:  {
                interpolate : /\{\{(.+?)\}\}/g
            }
        }
    }
});
```

### [Mod.js](https://github.com/modulejs/modjs) Compile

```js
// Modfile
module.exports = {
    tasks: {
        compile:{
            loader: "requirejs"
            source: "path/to/main.js",
            dest: "path/to/dest.js",
            baseUrl: "./js",
            stubModules: ['tmpl'],
            miniLoader: true
        }
    }
}
```

Note: do not forget config stubModules: ['tmpl'] in Modfile

## License

Requirejs tmpl plugin inspired by [text plugin](https://github.com/requirejs/text)

Templating inspired by [underscore.template](http://underscorejs.org/#template)

MIT License.

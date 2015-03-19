# bedrock-requirejs

A [bedrock][] module that provides a client-side module loader and
autoconfiguration capabilities for bower components.

**bedrock-requirejs** autogenerates a [RequireJS][]-configuration and autoloads
client-side modules based on your project's [bedrock][] configuration and
any [bower][] components you have installed. It also provides an API for
optimizing (bundling and minifying) your client-side JavaScript.

**bedrock-requirejs** is often coupled with [bedrock-views][] and
[bedrock-angular][] to provide frontend UIs.

## Quick Examples

```
npm install bedrock-requirejs
```

```js
var brRequire = require('bedrock-requirejs');

TODO: API examples

// brRequire.buildConfigSync
// brRequire.wrapConfigSync
// brRequire.wrapConfig
// brRequire.readBowerPackagesSync
// brRequire.optimize
```

## Configuration

For documentation on configuration, see [config.js](https://github.com/digitalbazaar/bedrock-requirejs/blob/master/lib/config.js).

## How It Works

**bedrock-requirejs** combines [RequireJS][] and [bower][] to ease developing,
managing, and loading client-side JavaScript. It is configuration driven but
designed such that most configuration options can be automatically computed.

With **bedrock-requirejs**, you should be able to `bower install` a bower
component, restart your **bedrock** server, and then when you next visit
your website, the bower component will be served when requested.

To make this possible, **bedrock-requirejs** will scan your project's
bower components directory when [bedrock][] starts. The value of
`bedrock.config.requirejs.bower.bowerrc.directory` will be used to determine
the bower components directory; it defaults to `bower_components`. Using the
results of the scan and any manually configured options, **bedrock-requirejs**
generates a [RequireJS][] project configuration that it serves via
[bedrock-express][] using the route `/requirejs/main.js`.

Manually configured options can be specified via
`bedrock.config.requirejs.config`, which is a basic [RequireJS][] configuration
object. There are also special `prefix` and `suffix` JavaScript files that are
used when generating `main.js`, that can be replaced, if necessary, via
`bedrock.config.requirejs.configPrefix` and
`bedrock.config.requirejs.configSuffix`. It's unlikely that these files need
to be replaced; if you find yourself thinking about replacing them, make sure
there isn't another feature that could be used instead to accomplish what
you're trying to do.

To load the auto-detected and configured modules, an HTML page includes
[RequireJS][] and sets its main module to `/requirejs/main.js`. If
[RequireJS][] is installed via [bower][], the HTML could look like this:

```html
<script data-main="/requirejs/main.js" src="/bower_components/requirejs/require.js"></script>
```

**bedrock-requirejs** also includes an API for running [RequireJS][]'s
optimizer. This feature can be used to concatenate (in dependency-sorted order)
and minify all JavaScript associated with a project. This means that all of
the JavaScript that can be auto-detected or custom-configured can be merged
into a single. minified file for use in production systems. The [RequireJS][]
configuration that is fed into the optimizer is generated using any custom
rules from `bedrock.config.requirejs.optimize.config`, which represents a
[RequireJS][] optimization configuration object, combined with the
results of auto-scanning the bower components directory. A special, simplified
[RequireJS][] module loader, [almond][] is also injected into the optimized
JavaScript. By including [almond][], there is no need to include `require.js`,
as the single JavaScript file will autoload itself. The resulting JavaScript
file will be written to `bedrock.config.requirejs.optimize.config.out`.
**bedrock-requirejs** will also serve this file, via `/requirejs/main.min.js`.
To run the optimizer:

```js
var brRequire = require('bedrock-requirejs');

brRequire.optimize(function(err) {
  if(err) {
    console.error('JavaScript optimization failed.', err);
  } else {
    console.log('JavaScript optimization complete.');
  }
});
```

To run the optimizer with a special tool that runs for each JavaScript file,
eg: [ng-annotate][], you can do the following:

```js
var brRequire = require('bedrock-requirejs');

brRequire.optimize({
  onBuildRead: function(moduleName, path, contents) {
    var ngAnnotate = require('ng-annotate');
    var result = ngAnnotate(contents, {add: true});
    if(result.errors) {
      console.error('ng-annotate failed for ' +
        'moduleName="' + moduleName + '", path="' + path + '", ' +
        'errors=', result.errors);
      process.exit(1);
    }
    return result.src;
  }, function(err) {
    if(err) {
      console.error('JavaScript optimization failed.', err);
    } else {
      console.log('JavaScript optimization complete.');
    }
  });
```

To include the optimized JavaScript via an HTML script tag:

```html
<script src="/requirejs/main.min.js"></script>
```

Note: If you're using [bedrock-views][], it exposes a subcommand `optimize`
that can run this API for your project for you, and it provides the HTML
pages that will serve either `/requirejs/main.js` or `/requirejs/main.min.js`
based on your [bedrock-views][] configuration options.

**bedrock-requirejs** also serves a simple event emitter API at
`/requirejs/events.js` that `main.js` uses to emit events listeners can use to
execute behavior before or after certain client-side modules are loaded.
There are two phases during which `main.js` will load modules: **early**
and **autoload**. The modules that are loaded during the **early** phase
include all of those listed in `bedrock.config.requirejs.config.deps`. The
modules that are loaded during the **autoload** phase include those listed
in `bedrock.config.requirejs.autoload`. The following events are emitted:

- **bedrock-requirejs.init**
  - Emitted before the **autoload** modules are loaded. This event allows
    any **early** loaded modules to run initialization code after all **early**
    loaded modules have been required but before any of the **autoload**
    modules are required. In order to receive this event, a module must be
    added to the `bedrock.config.requirejs.config.deps` array so it will be
    loaded during the **early** phase. The first parameter passed to any
    listener function will be an object where the keys are the values from
    `bedrock.config.requirejs.config.deps` and where the values are
    their associated module APIs.
- **bedrock-requirejs.ready**
  - Emitted after the **autoload** modules have all been required. Listeners
    can use this event to perform initialization/start up code that must be
    deferred until all **autoload** modules have been loaded. For example,
    a module that allows other modules to register with it before running
    initialization code may find listening for this event to be useful. An
    example of a module that uses this event is [bedrock-angular][], which
    waits for **autoload** modules to register themselves with angular prior to
    bootstrapping the main AngularJS application.


[almond]: https://github.com/jrburke/almond
[bedrock]: https://github.com/digitalbazaar/bedrock
[bedrock-angular]: https://github.com/digitalbazaar/bedrock-angular
[bedrock-express]: https://github.com/digitalbazaar/bedrock-express
[bedrock-views]: https://github.com/digitalbazaar/bedrock-views
[bower]: http://bower.io/
[ng-annotate]: https://github.com/olov/ng-annotate
[RequireJS]: http://requirejs.org/

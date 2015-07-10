# readium-js-viewer

**EPUB reader written in HTML, CSS and Javascript.**

This Readium software component implements the Readium Chrome extension / app for offline reading ( https://chrome.google.com/webstore/detail/readium/fepbnnnkkadjhjahcafoaglimekefifl ),
and the "cloud reader" for online e-books ( http://readium-cloudreader.divshot.io or more recently: http://development.readium.divshot.io ).

Please see https://github.com/readium/readium-shared-js for more information about the underlying rendering engine.

## License

**BSD-3-Clause** ( http://opensource.org/licenses/BSD-3-Clause )

See license.txt ( https://github.com/readium/readium-js-viewer/blob/develop/license.txt )


## Prerequisites

* A decent terminal. On Windows, GitShell works great ( http://git-scm.com ), GitBash works too ( https://msysgit.github.io ), and Cygwin adds useful commands ( https://www.cygwin.com ).
* NodeJS ( https://nodejs.org )


## Development

**Initial setup:**

* `git submodule update --init --recursive` to ensure that the readium-js-viewer chain of dependencies is initialised (readium-js, readium-shared-js and readium-cfi-js)
* `npm run prepare` (to perform required preliminary tasks, like patching code before building)

Note that the above command executes the following:

* `npm install` (to download dependencies defined in `package.json` ... note that the `--production` option can be used to avoid downloading development dependencies, for example when testing only the pre-built `build-output` folder contents)
* `npm update` (to make sure that the dependency tree is up to date)

**Git branch initialization:**

* `git submodule foreach --recursive 'git checkout BRANCH_NAME'`
* or simply `cd` inside each repository / submodule, and manually enter the desired branch name: `git checkout BRANCH_NAME` (Git should automatically track the corresponding branch in the 'origin' remote).


Advanced usage (e.g. TravisCI) - the commands below automate the remote/origin tracking process (this requires a Bash-like shell):

* ``for remote in `git branch -r | grep -v \> | grep -v master`; do git branch --track ${remote#origin/} $remote; done`` to ensure that all Git 'origin' remotes are tracked by local branches.
* ``git checkout `git for-each-ref --format="%(refname:short) %(objectname)" 'refs/heads/' | grep $(git rev-parse HEAD) | cut -d " " -f 1` `` to ensure that Git checks-out actual branch names (as by default Git initializes submodules to match their registered Git SHA1 commit, but in detached HEAD state)

(repeat for each repository / submodule)


**Typical workflow:**

No RequireJS optimization:

* `npm run http` (to launch an http server, automatically opens a web browser instance to the HTML files in the `dev` folder)
* Hack away! (e.g. source code in the `src/js` folder)
* Press F5 (refresh / reload) in the web browser

Or to use optimized Javascript bundles (single or multiple):

* `npm run build` (to update the RequireJS bundles in the build output folder)
* `npm run http:watch` (to launch an http server, automatically opens a web browser instance to the HTML files in the `dev` folder)
* `npm run http` (same as above, but without watching for file changes (no automatic rebuild))

And finally to update the distribution packages (automatically calls the `build` task above, so `npm run build` is redundant):

* `npm run dist` (Chrome extension and cloud reader, including the lite / no-library variant)

The above task takes a lot of time (as it builds distributable packages for *all* ReadiumJS flavours), and is in fact not strictly necessary to test the cloud reader (see `npm run http` above, using the "no optimise" RequireJS option). Thankfully, the packaged code for the Chrome App / Extension can be quickly generated using this build command instead:

* `npm run chromeApp` (generates a ready-to-use Readium packaged app for Chrome, inside the usual `dist/chrome-app` folder)

**Plugins integration:**

When invoking the `npm run build` command, the generated `build-output` folder contains RequireJS module bundles that include the default plugins specified in `readium-js/readium-js-shared/plugins/plugins.cson` (see the `readium-js/readium-js-shared/PLUGINS.md` documentation). Developers can override the default plugins configuration by using an additional file called `plugins-override.cson`. This file is git-ignored (not persistent in the Git repository), which means that Readium's default plugins configuration is never at risk of being mistakenly overridden by developers, whilst giving developers the possibility of creating custom builds on their local machines.

## RequireJS bundle optimisation

Note that by default, compiled RequireJS bundles are minified / mangled / uglify-ed. You can force the build process to generate non-compressed Javascript bundles by setting the `RJS_UGLY` environment variable to "no" or "false" (any other value means "yes" / "true").

This may come-in handy when testing / debugging the Chrome Extension (Packaged App) in "developer mode" directly from the `dist` folder (i.e. without the sourcemaps manually copied into the script folder).

## Tests

Mocha-driven UI tests via Selenium (not PhantomJS, but actual installed browsers accessed via WebDriver):

* `npm run test:firefox`
* `npm run test:chrome`
* `npm run test:chromeApp`

`npm run test` (runs all of the above)

Via SauceLabs:

* `npm run test:sauce:firefox`
* `npm run test:sauce:chrome`
* `npm run test:sauce:chromeApp`

`npm run test:sauce` (runs all of the above)

Travis (Continuous Integration) automatically uses a chromeApp and Firefox test matrix (2x modes), and uses SauceLabs to actually run the test.

## Distribution

See the `dist` folder contents:

* `cloud-reader`
* `cloud-reader-lite` (same as above, without the ebook library feature)
* `chrome-app` (Google Chrome Extension / Packaged App)

The source maps are generated separately, so they are effectively an opt-in feature (simply copy/paste them next to their original Javascript file counterparts, e.g. in the `scripts` folder)

## Cloud reader deployment

The `cloud-reader` distribution folder (see section above) can be uploaded to an HTTP server as-is,
in which case a sibling `/epub_content/` folder is expected to contain exploded or zipped EPUBs,
and the `/epub_content/epub_library.json` file is expected to describe the available ebooks in the online library
(see the existing examples in `readium-js-viewer` repository). Additionally, the `epubs` URL parameter (HTTP GET)
can be used to specify a different location for the JSON file that describes the ebook library contents, for example:
`http://domain.com/index.html?epubs=http://otherdomain.com/ebooks.json` (assuming both HTTP servers are suitably configured with CORS),
or for example `http://domain.com/index.html?epubs=EPUBs/ebooks.json` (assuming a folder named `EPUBs/` exists as a sibling of `index.html`,
and this folder contains the `ebooks.json` file). Finally, the ebook library can be permanently set to a specific location,
by editing `cloud-reader/index.html` and by replacing the value of `epubLibraryPath`:
```javascript
require.config({
config : {
        'readium_js_viewer/ModuleConfig' : {
            'epubLibraryPath`: VALUE
        }
});
```

The `cloud-reader-lite` distribution does not feature an ebook library, so EPUBs must be specified via the URL parameter (HTTP GET), for example:
`http://domain.com/index.html?epub=http://otherdomain.com/ebook.epub` (assuming both HTTP servers are suitably configured with CORS),
or for example `http://domain.com/index.html?epub=EPUBs/ebook.epub` (assuming a folder named `EPUBs/` exists as a sibling of `index.html`,
and this folder contains the `ebook.epub` file
(note that the folder name is arbitrary, and it may in fact follow the default naming convention: `epub_content/`)).


For more information about HTTP CORS, see https://docs.google.com/document/d/1RK_59-75OSE0PA6wexD9rYHQLnNYfoKIpsnH3SmDpUc


## NPM (Node Package Manager)

NOTE THAT THIS FEATURE IS NOT FULLY IMPLEMENTED YET (PLEASE REFERENCE THE GITHUB REPOSITORIES INSTEAD FROM YOUR PACKAGE.JSON)

All packages "owned" and maintained by the Readium Foundation are listed here: https://www.npmjs.com/~readium

Note that although Node and NPM natively use the CommonJS format, Readium modules are currently only defined as AMD (RequireJS).
This explains why Browserify ( http://browserify.org ) is not used by this Readium project.
More information at http://requirejs.org/docs/commonjs.html and http://requirejs.org/docs/node.html

* Make sure `npm install readium-js-viewer` completes successfully ( https://www.npmjs.com/package/readium-js-viewer )
* Execute `npm run http`, which opens a web browser to a basic RequireJS bootstrapper located in the `dev` folder (this is *not* a production-ready minified application)

Note: the `--dev` option after `npm install readium-js-viewer` can be used to force the download of development dependencies,
but this is kind of pointless as the code source and RequireJS build configuration files are missing.
See below if you need to hack the code.


## How to use (RequireJS bundles / AMD modules)

The `build-output` directory contains two distinct folders:

### Single bundle

The `_single-bundle` folder contains `readium-js-viewer_all.js` (and its associated source-map file, as well as a RequireJS bundle index file (which isn't actually needed at runtime, so here just as a reference)),
which aggregates all the required code (external library dependencies included, such as Underscore, jQuery, etc.),
as well as the "Almond" lightweight AMD loader ( https://github.com/jrburke/almond ).

This means that the full RequireJS library ( http://requirejs.org ) is not actually needed to bootstrap the AMD modules at runtime,
as demonstrated by the HTML file in the `dev` folder (trimmed for brevity):

```html
<html>
<head>

<!-- main code bundle, which includes its own Almond AMD loader (no need for the full RequireJS library) -->
<script type="text/javascript" src="../build-output/_single-bundle/readium-js-viewer_all.js"> </script>

<!-- index.js calls into the above library -->
<script type="text/javascript" src="./index.js"> </script>

</head>
<body>
<div id="viewport"> </div>
</body>
</html>
```

### Multiple bundles


The `_multiple-bundles` folder contains several Javascript bundles (and their respective source-map files, as well as RequireJS bundle index files):


* `readium-external-libs.js`: aggregated library dependencies (e.g. Underscore, jQuery, etc.)
* `readium-shared-js.js`: shared Readium code (basically, equivalent to the `js` folder of the "readium-shared-js" submodule)
* `readium-js.js`: the core Readium code (basically, equivalent to the `js` folder of the "readium-js" submodule)
* `readium-js-viewer.js`: this Readium code (mainly, the contents of the `js` folder)
* `readium-plugin-example.js`: simple plugin demo
* `readium-plugin-annotations.js`: the annotation plugin (DOM selection + highlight), which bundle actually contains the "Backbone" library, as this dependency is not already included in the "external libs" bundle.
)

In addition, the folder contains the full `RequireJS.js` library ( http://requirejs.org ), as the above bundles do no include the lightweight "Almond" AMD loader ( https://github.com/jrburke/almond ).

Usage is demonstrated by the HTML file in the `dev` folder (trimmed for brevity):

```html
<html>
<head>

<!-- full RequireJS library -->
<script type="text/javascript" src="../build-output/_multiple-bundles/RequireJS.js"> </script>



<!-- individual bundles: -->

<!-- readium CFI library -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-cfi-js.js"> </script>

<!-- external libraries -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-external-libs.js"> </script>

<!-- readium itself -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-shared-js.js"> </script>

<!-- simple example plugin -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-plugin-example.js"> </script>

<!-- annotations plugin -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-plugin-annotations.js"> </script>

<!-- readium js -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-js.js"> </script>

<!-- readium js viewer -->
<script type="text/javascript" src="../build-output/_multiple-bundles/readium-js-viewer.js"> </script>


<!-- index.js calls into the above libraries -->
<script type="text/javascript" src="./index.js"> </script>

</head>
<body>
<div id="viewport"> </div>
</body>
</html>
```


Note how the "external libs" set of AMD modules can be explicitly described using the `bundles` RequireJS configuration directive
(this eliminates the apparent opacity of such as large container of library dependencies):


```html

<script type="text/javascript">
requirejs.config({
    baseUrl: '../build-output/_multiple-bundles'
});
</script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-cfi-js.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-external-libs.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-shared-js.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-plugin-example.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-plugin-annotations.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-js.js.bundles.js"> </script>

<script type="text/javascript" src="../build-output/_multiple-bundles/readium-js-viewer.js.bundles.js"> </script>

```




## CSON vs. JSON (package.json)

CSON = CoffeeScript-Object-Notation ( https://github.com/bevry/cson )

Running the command `npm run cson2json` will re-generate the `package.json` JSON file.
For more information, see comments in the master `./package/package_base.cson` CSON file.

Why CSON? Because it is a lot more readable than JSON, and therefore easier to maintain.
The syntax is not only less verbose (separators, etc.), more importantly it allows *comments* and *line breaking*!

Although these benefits are not so critical for basic "package" definitions,
here `package.cson/json` declares relatively intricate `script` tasks that are used in the development workflow.
`npm run SCRIPT_NAME` offers a lightweight technique to handle most build tasks,
as NPM CLI utilities are available to perform cross-platform operations (agnostic to the actual command line interface / shell).
For more complex build processes, Grunt / Gulp can be used, but these build systems do not necessarily offer the most readable / maintainable options.

Downside: DO NOT invoke `npm init` or `npm install --save` `--save-dev` `--save-optional`,
as this would overwrite / update the JSON, not the master CSON!

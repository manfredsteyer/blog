# AOT and Tree Shaking with Angular, webpack2 and/or rollup: Step by Step

*Updated in January 2017*: After very good conversations with [Carmen Popoviciu](https://twitter.com/carmenpopoviciu?lang=de), I've updated this article to strike out some important facts and to also explain *why* the steps described here are necessary. 

In the last days, I've adapted my Angular 2 sample for ahead of time (AOT) compilation with the template compiler. Here I'm documenting the necessary steps for such an undertaking as well as my learnings towards this. The [whole sample](https://github.com/manfredsteyer/angular-aot) can be found [here](https://github.com/manfredsteyer/angular-aot).

I'm subdividing this writing into two parts:

1. Part 1: Angular 2 AOT Compilation and Tree Shaking with Webpack2 and/or Rollup (this one)
2. Part 2: Angular 2 AOT Compilation with @ngtools/webpack and Lazy Loading (coming up)

> I want to explicitly thank [Wassim Chegham](https://github.com/manekinekko/angular2-aot-demo/) and [Minko Gechev](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/) for providing great information about this topic. They really helped me getting started with it. Another valuable source for this topic is the [cookbook for AOT and Tree Shaking](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) within the official documentation of Angular 2.

## Motivation

In order to improve performance, Angular compiles templates into TypeScript code. The easiest solution for this is Just in Time (JIT) compilation which for instance takes happen after downloading the files into the browser. To speed up the program start, one can also compile the templates during the build which is called Ahead of Time (AOT) compilation. Thanks to this, we don't need to include the sources of the Angular Compiler into our build which reduces the parts of Angular that need to be loaded into the browser.  

As the generated TypeScript code can be statically analyzed, it it possible to find out which parts of Angular or other libraries aren't used. This is where tools for tree shaking like [webpack2](https://webpack.js.org) or [rollup](http://rollupjs.org) come in. Those tools try to remove the unused code or to put it in another they: They are "shaking off" the loose branches of your source code. 

This post shows how to use the two mentioned techniques by describing an [example](https://github.com/manfredsteyer/angular-aot) which can found in my [GitHub repository](https://github.com/manfredsteyer/angular-aot).

## Configuration

To demonstrate the usage of AOT and tree shaking, this post first of all uses the Angular Compiler to compile the templates to TypeScript. After this, it also compiles this TypeScript code together with the application's TypeScript code down to EcmaScript 5. To enable tree shaking, this samples uses EcmaScript 5 together with the module system of EcmaScript 2015 that defines the well known ``import`` and ``export`` statements. These statements allow for a static code analysis to find unused code and is the foundation for tree shaking tools:

![AOT process](http://i.imgur.com/qgmifSO.png)

Then, the sample uses rollup or webpack2 to perform tree shaking. These tools can also be configured to use UglifyJS to minimize the resulting bundles. At the time of writing, UglifyJS can only deal with EcmaScript 5 which is one reason for converting the code into this version. Another reason for transpiling the code down to EcmaScript 5 is that currently webpack is not supporting a newer version. A third reason is, that currently most libraries are available as EcmaScript 5 files. Fortunately, EcmaScript 5 can be used together with EcmaScript 2015 modules which is the foundation for tree shaking.  

As mentioned [here](http://link-to-other-article), using EcmaScript 2015 or higher at this point would lead to better results as it provides more ways for a static analysis. The result of webpack2's or rollup's work is a tree shaked and minified bundle.

## Refactoring

As AOT compilation does not allow for dynamic references, using ``require`` is not possible. However, thanks to the ``angular2-template-loader`` for Webpack which lines in the templates and styles for components, we can use relative references in JIT mode anyway. In addition to that, the AOT compiler also supports relative paths:

```
@Component({
    selector: 'flug-suchen',
    templateUrl:  './flug-suchen.component.html',
    styleUrls: ['./flug-suchen.component.css'],
})
export class FlugSuchenComponent {
    [...]
}
```

As this sample shows, the compiler is more strict when using AOT. That might lead to some errors when compiling. Using the feedback we get from the compilation errors, we can fix our code. [Oliver Combe](https://github.com/ocombe), the author of the popular [ng2-translate](https://github.com/ocombe/ng2-translate) library, created a nice [list with things that don't work with the AOT Compiler](https://github.com/mgechev/codelyzer/issues/215). Additional information can be found [here](https://gist.github.com/chuckjaz/65dcc2fd5f4f5463e492ed0cb93bca60).

To provide instant feedback during the development, we can leverage the Angular Language Service that has been introduced with Angular 2.3. It targets editors and IDE and helps them to provide Angular specific feedback and code completion for both, typescript files as well as templates. At the time of writing, the upcoming version of WebStorm supports this and there is a preview of a plugin that makes the language service available for Visual Studio Code. 


## Packages

In order to work with AOT compilation as well as with [Webpack2](https://webpack.github.io/) and [Rollup](http://rollupjs.org), one has to install several packages. The next listing contains some dev-dependencies I've used for this:

```
    "angular2-template-loader": "^0.6.0",
    "awesome-typescript-loader": "~3.0.0-beta.18",
    "css-loader": "^0.26.0",
    "file-loader": "^0.9.0",
    "html-webpack-plugin": "^2.21.0",

    "webpack": "^2.2.0",
    "webpack-dev-server": "^2.2.0",
    "webpack-dll-bundles-plugin": "^1.0.0-beta.2"

    "rollup": "^0.41.4",
    "rollup-plugin-commonjs": "^7.0.0",
    "rollup-plugin-node-globals": "^1.1.0",
    "rollup-plugin-node-resolve": "^2.0.0",
    "rollup-plugin-uglify": "^1.0.1",

```

We also need the ``compiler-cli`` and ``plattform-server`` as a dependency:

```
	"@angular/compiler-cli": "~2.4.0",
	"@angular/platform-server": "~2.4.0",
```

The [full list of dev-dependencies](https://github.com/manfredsteyer/angular-aot) can be found in my [sample at GitHub](https://github.com/manfredsteyer/angular-aot).

## tsconfig.json

The AOT compiler uses its own ``tsconfig.json``-file. According to the docs, I've named it ``tsconfig.aot.json``:

```
{
  "compilerOptions": {
    "target": "es5",
    "module": "es2015",
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist/unbundled-aot",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": ["es2015", "dom"],
    "noImplicitAny": true,
    "suppressImplicitAnyIndexErrors": true
  },

 
  "angularCompilerOptions": {
   "genDir": "aot",
   "skipMetadataEmit" : true
 }
}
```

As mentioned above, it's important to use the target ``es5`` together with the module format ``es2015`` to make tree shaking possible. The property ``genDir`` within ``angularCompilerOptions`` defines, where the template compiler should place the files it generated out of HTML files. The sample is also skipping the generation of metadata for the AOT Compiler as this is only needed for reusable libraries. 

The property ``outDir`` defines that the resulting ES5 files should be placed in the folder ``dist/unbundled-aot``. Wepack2 and rollup will use this folder.

## Compiling the templates

The following npm script I've set up within ``package.json`` compiles the HTML templates and creates TypeScript files for it. In addition, it also compiles everything down to ES5 afterwards:

```
"ngc": "ngc -p tsconfig.aot.json",
```

To start this script, one has to use ``npm run ngc``.


## Bootstrapping for AOT

To bootstrap Angular 2 in AOT mode, it has to be started with ``platformBrowser().bootstrapModuleFactory``:

```
// main.aot.ts

import { platformBrowser } from '@angular/platform-browser';
import { AppModuleNgFactory } from '../aot/app/app.module.ngfactory';

import 'rxjs/add/operator/map';
import 'rxjs/add/operator/do';

platformBrowser().bootstrapModuleFactory(AppModuleNgFactory).catch(err => console.error(err));
```

``AppModuleNgFactory`` is a class the AOT compiler created for the root module ``AppModule``. After creating or updating this file, one should run the ngc compiler again.


## Webpack2 Configuration for AOT

The next listing shows the (simple) webpack2 configuration I'm using for the AOT sample. It is pointing to two entry points within the folder ``dist/unbundled-aot``. One of them referenced to the necessary polyfills and the other one points to the compiled version of the above described ``main.aot.ts``:


```
var webpack = require('webpack');
var CompressionPlugin = require("compression-webpack-plugin");

module.exports = {

    profile: true,
    devtool: false,
    entry: {
        'polyfills': './dist/unbundled-aot/app/polyfills.js',
        'app': './dist/unbundled-aot/app/main.aot.js'
    },
    output: {
        path: __dirname + "/dist/aot",
        filename: "[name].js",
        publicPath: "dist/"
    },
    resolve: {
        extensions: [/*'.ts',*/ '.js', '.jpg', '.jpeg', '.gif', '.png', '.css', '.html']
    },
    module: {
        loaders: [
            { test: /\.(jpg|jpeg|gif|png)$/, loader:'file-loader?name=img/[path][name].[ext]' },
            { test: /\.(eof|woff|woff2|svg)$/, loader:'file-loader?name=img/[path][name].[ext]' },
            { test: /\.css$/, loader:'raw-loader' },
            { test: /\.html$/,  loaders: ['html-loader'] }
        ],
        exprContextCritical: false
    },
    plugins: [
        new webpack.LoaderOptionsPlugin({
            minimize: true,
            debug: false
        }),
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            },
            output: {
                comments: false
            },
            sourceMap: false
        }),
        new CompressionPlugin({
            asset: "[path].gz[query]",
            algorithm: "gzip",
            test: /\.js$|\.html$/,
            threshold: 10240,
            minRatio: 0.8
        })
    ],
    node: {
        __filename: true
    },
    devServer: {
        inline:true,
        port: 8080,
        historyApiFallback: true,
        watchOptions: {
            aggregateTimeout: 300,
            poll: 1000
        }
    }
  
};
```

The plugins ``LoaderOptionsPlugin`` and ``UglifyJsPlugin`` reduce the bundle size to a minimum by means of [minification and tree shaking](http://moduscreate.com/webpack-2-tree-shaking-configuration/).

## Webpack2 Configuration for JIT

I've also created a JIT based build. This build I'm using during development because it is much faster than an AOT based build. In principle, the webpack configuration for this looks like the one for AOT. But its entry point ``app`` points to the file ``main.jit.ts`` which is using Angular's JIT-compiler. Another difference is that it is using the ``angular2-template-loader`` which inlines component templates. In addition to that, it takes care of compiling TypeScript leveraging the ``awesome-typescript-loader``.

```
[...]
var webpack = require('webpack');
var CompressionPlugin = require("compression-webpack-plugin");

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;

module.exports = {

    devtool: false,
    entry: {
        'polyfills': './app/polyfills.ts',
        'app': './app/main.jit.ts'
    },
    output: {
        path: __dirname + "/dist/jit",
        filename: "[name].js",
        publicPath: "dist/"
    },
    resolve: {
        extensions: ['.ts', '.js', '.jpg', '.jpeg', '.gif', '.png', '.css', '.html']
    },
    module: {
        loaders: [
            { test: /\.(jpg|jpeg|gif|png)$/, loader:'file-loader?name=img/[path][name].[ext]' },
            { test: /\.(eof|woff|woff2|svg)$/, loader:'file-loader?name=img/[path][name].[ext]' },
            { test: /\.css$/, loader:'raw-loader' },
            { test: /\.html$/,  loaders: ['raw-loader'] },
            { test: /\.ts$/, loaders: ['angular2-template-loader', 'awesome-typescript-loader'], exclude: /node_modules/}
        ],
        exprContextCritical: false
    },
    plugins: [
        
        new webpack.LoaderOptionsPlugin({
            minimize: true,
            debug: false
        }),
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            },
            output: {
                comments: false
            },
            sourceMap: false
        }),
        new CompressionPlugin({
            asset: "[path].gz[query]",
            algorithm: "gzip",
            test: /\.js$|\.html$/,
            threshold: 10240,
            minRatio: 0.8
        })
    ],
    node: {
        __filename: true
    },
    devServer: {
        inline:true,
        port: 8080,
        historyApiFallback: true,
        watchOptions: {
            aggregateTimeout: 300,
            poll: 1000
        }
    }
  
};
[...]
``` 


## Rollup Configuration for AOT

To find out, whether rollup brings improvements over just using webpack2, I've used the rollup configuration of the [cookbook for AOT and rollup from angular.io](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html):

```
// rollup.config.js

import nodeResolve from 'rollup-plugin-node-resolve'
import uglify      from 'rollup-plugin-uglify'

export default {
    entry: 'dist/unbundled-aot/app/main.aot.js',
    dest: 'dist/build.js', // output a single application bundle
    sourceMap: false,
    treeshake: true,
    format: 'iife',
    onwarn: function(warning) {
        // Skip certain warnings
        if ( warning.code === 'THIS_IS_UNDEFINED' ) { return; }
        if ( warning.indexOf("The 'this' keyword is equivalent to 'undefined'") > -1 ) { return; }
        console.warn( warning.message );
    },    
    plugins: [
        nodeResolve({jsnext: true, module: true}),
        commonjs({
            include: 'node_modules/rxjs/**',
        }),
        uglify()
    ]
}

```

Again, this configuration is pointing to the EcmaScript 5 files using the EcmaScript 2015 Module System within ``dist/unbundled-aot``. 

## Build Scripts

To build the solution for all three scenarios, I'm using the following npm scripts:

```
"build-all": "npm run webpack:jit && npm run webpack:aot && npm run rollup:aot",
"webpack:aot": "ngc -p tsconfig.aot.json && webpack --config webpack.aot.config.js",
"webpack:jit": "webpack --config webpack.jit.config.js",
"rollup:aot": "ngc -p tsconfig.aot.json && rollup -c rollup.js",
```

The command ``npn run build-all`` is starting all three builds.   

## Results

The bundle for the JIT mode had 848 kB:

          Asset     Size  Chunks                    Chunk Names
         app.js   848 kB       0  [emitted]  [big]  app
   polyfills.js   103 kB       1  [emitted]         polyfills
polyfills.js.gz  33.5 kB          [emitted]
      app.js.gz   199 kB          [emitted]

When it comes to AOT mode, the bundle could be reduced:

          Asset     Size  Chunks                    Chunk Names
         app.js   531 kB       0  [emitted]  [big]  app
   polyfills.js   103 kB       1  [emitted]         polyfills
polyfills.js.gz  33.5 kB          [emitted]
      app.js.gz   112 kB          [emitted]

The AOT bundle created with rollup was even smaller: 

```
	470.978 build.js
```

After gzipping it, the resulting file took 94 kB:

```
	94.337 build.js.gz
```

## Starting the samples

To start the three samples, I've created three HTML files: ``index.jit.html`` for the JIT mode, ``index.aot.html`` for the AOT base code build with webpack bundles and ``index.rollup.aot.html`` for its counterpart build with rollup. Those files are also included in the provided sample.


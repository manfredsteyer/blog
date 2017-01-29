# 5 things you wouldn't necessarily expect when using AOT and Tree Shaking in Angular

AOT speeds up the application especially the startup because templates are already compiled during build. In addition to this, tree shaking reduces the needed file size by removing unused code. But there are some things, one wouldn't necessarily expect. This article presents five of them, we discovered and researched when working with it.

> This article has been written after some very insightful conversations with [Carmen Popoviciu](https://twitter.com/carmenpopoviciu?lang=de). She pointed out issues regarding AOT and Tree Shaking and helped me to discover them and their origins.

## The most ovious: EcmaScript Modules

Let's start here with the most ovious and least surprising fact: You need sources based on EcmaScript 2015 for tree shaking. At least you need to use EcmaScript Modules introduced with EcmaScript 2015. Those modules use the well known ``import`` and ``export`` statements which allow for a static code analysis which is the foundation for finding unused code.  

It is possible to use a combination of EcmaScript 5 and EcmaScript Modules to support a wider range of tools like webpack 2 which - at the time of writing - does not support EcmaScript 2015 but leverages ``import`` and ``export`` statements for tree shaking. 

Currently, the product team ships the Angular modules using this exact combination (besides additional UMD bundles). As the next section shows, this brings also a drawback as there are some limitations when it comes to analyzing EcmaScript 5 code for tree shaking.   


## Current Limitations for Tree Shaking

Examples for Tree Shaking clearly show that it brings benefits in terms of file size. But at the time of writing, there is much room for improvement. This mostly has to do with the fact that tools for tree shaking like rollup and webpack2 cannot make sure in every situation that removing unused code is safe. This is especially the case in situations, where this code could introduce side effects like changing a global variable. Therefore, those tools don't remove as many code as possible. 

As a matter of fact, this fact holds true for the method ``Object.defineProperty``, which mutates the first passed object. Unfortunately, this very method is used when transpiling class members down to EcmaScript 5 and therefore affects several parts of Angular as well as Libraries like Angular Material.

One can find such parts of the code by activating the output of warnings within the webpack configuration for instance. This makes UglifyJS to list every part of the bundle which it doesn't remove because of the mentioned reason:

```
new webpack.optimize.UglifyJsPlugin({
    compress: {
        warnings: true
    },
    output: {
        comments: true
    },
    sourceMap: false
}),
```

Another interesting experiment is to import a library like Angular Material into the ``AppModule`` without using it at all. Normally, one would expect that it is tree shaken off but for the reason discussed here this leads to a bigger bundle. 

Further information about this can be found inside the issues of [webpack](https://github.com/webpack/webpack/issues/2867) and [rollup](https://github.com/rollup/rollup/issues/1130).

## The myth about smaller bundles

In general, one would expect smaller bundles as the result of AOT and Tree Shaking. While this should be true for tree shaking, AOT compiles templates down to JavaScript code that is bigger. In small projects this isn't necessarily noticed, as the usage of AOT allows for omitting the code for JIT compilation. But with a growing number of templates the increase in file size due to the larger emitted JavaScript code outweighs this.

## SASS and other Stylesheet Languages

As we found out, there is also an issue when using the AOT compiler together with SASS and SCSS. It cannot deal with it and yields errors. The webpack-plugin discussed [here](http://link-is-coming---currently-my-article-about-this-is-just-german) automatically deals with this situation when there is a webpack loader for it.

## Libraries need to support AOT and Tree Shaking

In order to shake unused code of libraries off and to use them with AOT, they also have to support it. This not only means that they have to provide the code using EcmaScript Modules. They also have to provide some meta data, especially meta data for Angular's AOT compiler.

No one less than [Minko Gechev](http://blog.mgechev.com/2017/01/21/distributing-an-angular-library-aot-ngc-types/) wrote a wonderful [article](http://blog.mgechev.com/2017/01/21/distributing-an-angular-library-aot-ngc-types/) about this. At the end, it also provides a check list with all the things one should consider. 


---
layout: post
title: Adding WebPack to an MVC website
categories: ASP.NET MVC AngularJs Webpack
---

This post covers setting up js and css bundling in a boilerplate Asp.net MVC project (not core).

[Sample Github Repository](https://github.com/Togusa09/MvcWebpackAngularJs)

[WebPack Documentation](https://webpack.js.org/)

## 1 - Setup NPM

This step installs webpack and configures it to build an empty javascript file.

- Open Terminal window for the web project
- `npm init -y`
- `npm install webpack webpack-cli --save-dev`
- create an empty `app.js` in the scripts folder, assuming the MVC project convention for script location. The location isn't too important, you just need somewhere to put your scripts.

- create `webpack.config.js`
{% highlight javascript %}
var path = require('path');

module.exports = {
    mode: 'development',
    // Path to the 'primary' script of your project, that references all other built resources
    entry: './scripts/app.js',
    output: {
        // Output directory and file name
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
};
{% endhighlight %}

- Replace the `scripts` section in the package.json file with the following:
{% highlight javascript %}
"scripts": {
    "dev": "webpack --mode development --watch",
    "build": "webpack --mode production"
  }
{% endhighlight %}
- `npm run dev` will do a development build of the scripts, and watch for any changes
- `npm run build` will do a production build of the scripts

This will now run a build and create `dist/bundle.js`. It will be empty as there's no javascript in `app.js`.

[Webpack Config Documentation](https://webpack.js.org/configuration/)
[Webpack CLI Documentation](https://webpack.js.org/api/cli/)

## 2 - Bundling in 3rd party libraries

This step removes the default javascript bundling for the default libraries (jQuery and bootstrap). They are then re-added via NPM and included in the script created by webpack.

- Remove the jquery, jqueryval and bootstrap javascript bundles from `App_start/BundleConfig.cs` and `_Layout.cshtml`. Any css bundles can be left in place for the moment.
- Add a script reference to `dist/bundle.js` in the `_Layout.cshtml`
- `npm install jquery jquery-validation bootstrap@3 --save` (MVC projects have bootstrap 3 by default)
- Add the following to the `app.js`:

{% highlight javascript %}
// Reference the npm packages so that they're bundled into the build
import 'jquery';
import 'jquery-validation';
import 'bootstrap';

// This willchange the application name in the navbar
$(document).ready(function () {
    $('.navbar-brand').text('test2');
});
{% endhighlight %}

- Add the following to the webpack.config.js. This is required to handle the setup of the jQuery globals so that they can be used in scripts
{% highlight javascript %}
plugins: [
        new webpack.ProvidePlugin({
            $: 'jquery',
            jQuery: 'jquery',
            'window.jQuery': 'jquery'
        })
    ]
{% endhighlight %}

[Provide Plugin example for jQuery](https://webpack.js.org/plugins/provide-plugin/)
[Using Bootstrap with Webpack](https://getbootstrap.com/docs/4.0/getting-started/webpack/) Note: This is using different versions of webpack/bootstrap and doesn't match exactly

## 3 - Dev/Production Configs

This step splits the webpack configuration to allow for different options for production and development

- Rename `webpack.config.js` to `webpack.common.js`
- npm install webpack-merge --save-dev
- create `webpack.dev.js`
{% highlight javascript %}
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development'
});
{% endhighlight %}

- create `webpack.production.js`
{% highlight javascript %}
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'production'
});
{% endhighlight %}

- Update scripts section of `package.json` to:
{% highlight javascript %}
"dev": "webpack --config webpack.dev.js --watch",
"build": "webpack --config webpack.prod.js"
{% endhighlight %}

[Webpack production config guide](https://webpack.js.org/guides/production/)
[Webpack-merge plugin](https://github.com/survivejs/webpack-merge)

## 4 - Bundling CSS

This step includes the site CSS in the webpack process, and shows two different ways that it can be bundled. The two ways this can be done are inclusion in the loaded javascript module, or output as a seperate file. For this I've opted for bundling into the module for dev and creating a seperate file for the css bundle in production.

- `npm install css-loader file-loader style-loader mini-css-extract-plugin optimize-css-assets-webpack-plugin --save-dev`
  - `mini-css-extract-plugin` extracts the the css to it's own file
  - `optimize-css-assets-webpack-plugin` minifies the css
- Remove the style bundle from the BundleConfig.cs
- Replace the bundle reference in `_Layout.cshtml` with a refernce to `dist/styles.css`
- Add imports for styles into the `app.js`
{% highlight javascript %}
import 'bootstrap/dist/css/bootstrap.min.css';
import '../Content/Site.css';
{% endhighlight %}
- Update `webpack.common.js` to handle the font and image assets referenced in the styles:
{% highlight javascript %}
module: {
        rules: [
            {
                // This rule exports the files to the dist/assets folder and updates any paths in the css
                test: /\.(png|jpg|jpeg|gif|woff|woff2|ttf|eot|svg)(\?.*)?$/,
                loader: 'file-loader?name=../dist/assets/[name].[ext]'
            },
          ]
    }
{% endhighlight %}
- Update `webpack.dev.js` to bundle in the styles
{% highlight javascript %}
module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader?sourceMap']
            }
        ]
}
{% endhighlight %}
- Update `webpack.prod.js` to generate a seperate bundle file for the css
{% highlight javascript %}
optimization: {
        splitChunks: {
          cacheGroups: {
            styles: {
              name: 'styles',
              test: /\.css$/,
              chunks: 'all',
              enforce: true,
            },
          },
        },
        // When overriding the minimizers, the default JS minimizer needs to be re-added
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
      },
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].css',
            chunkFilename: '[name].css'
        })
    ],
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, 'css-loader'],
            },
        ]
    }
{% endhighlight %}

[css-loader](https://github.com/webpack-contrib/css-loader)
[style-loader](https://github.com/webpack-contrib/style-loader)
[file-loader](https://github.com/webpack-contrib/file-loader)
[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
[Webpack docs for mini-css-extract-plugin](https://webpack.js.org/plugins/mini-css-extract-plugin/)

## 5 - Injecting script names

Webpack can inject the names of the files it generates into a html (or cshtml) file. This allows the use of dynamic files names based on the file hash or compile time to ensure that the client is loading the latest scripts. This example just inserts into the body or head of the template, but it is possible to define specific locations for insertion: [Custom Insertion](https://github.com/jantimon/html-webpack-plugin/tree/master/examples/custom-insertion-position)

- Rename `_Layout.cshtml` to `_Layout_Template.cshtml` (This does create the risk of another developer editing the wrong file)
- npm install html-webpack-plugin --save-dev
- Add the plugin configuration to `webpack.common.js`
{% highlight javascript %}
new HtmlWebpackPlugin({
    inject: "body",
    filename: "../Views/Shared/_Layout.cshtml",
    template: "./Views/Shared/_Layout_Template.cshtml"
})
{% endhighlight %}

[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)
[Webpack docs for html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/)

## Step 6 - Adding an AngularJs application

- Install AngularJs npm install angular@1.7.8
- Create `app.module.js` in the same directory as `app.js`. This allows us to import the module into other javascript files.
{% highlight javascript %}
import angular from 'angular';

export let app = angular.module("testApp", []);
{% endhighlight %}
- Create `scripts/controllers/homeController.js`
{% highlight javascript %}
import { app } from "../app.module";

var homeContoller = function(){
    this.testProperty = "Test Value";
    this.testMethod = function(){
        return "Test Method Value";
    }
};

app.controller('homeController', homeContoller);
{% endhighlight %}
- Add into `Views\Home\Index.html`, so that it appears on the home page

{% highlight javascript %}
<div class="row" ng-app="testApp">
    <div class="col-md-4" ng-controller="homeController as home">
        <p>{{home.testProperty}}</p>
        <p>{{home.testMethod()}}</p>
    </div>
</div>
{% endhighlight %}
- Import the controller script into the `app.js`. It is also possible to create a file specifically for things like controller imports ie. `app.controllers.js`, then import that in `app.js`;
{% highlight javascript %}
import './app.module';
import './Controllers/homeController';
{% endhighlight %}

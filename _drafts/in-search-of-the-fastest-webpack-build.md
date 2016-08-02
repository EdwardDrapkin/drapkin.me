---
layout: post
title: React with Hot Module Replacement
summary: Part one of my webpack series
categories: javascript
thumbnail: bolt
tags:
 - javascript
 - webpack
 - optimization
---

# Getting Started

We're going to setup a basic site with a developer-friendly webpack configuration.

### Greeting the world

Let's start by installing a basic set of packages we'll need and rendering a simple React example.

```shell
$ npm i --save-dev react react-dom
```

Next let's throw together a quick hello world to render:

#### index.js

```javascript
import React from 'react';
import {render} from 'react-dom';

function HelloWorld(props) {
    return <div>Hello World!</div>;
}

render(<HelloWorld />, document.getElementById('demo'));
```

#### index.html
```html
<html>
<body>
    <div id="demo"></div>
    <script type="text/javascript" src="public/index.js"></script>
</body>
</html>
```

And here's where the fun begins!  Install webpack, babel, and the set of presets we're going to be using.

```shell
$ npm i --save-dev webpack@^2.1.0-beta webpack-dev-server@^2.1.0-beta  babel-loader babel-preset-es2015 babel-preset-react babel-preset-modern-browsers
```

Rather than use a single webpack configuration file, we'll need several, so go ahead and create a `webpack.config.js` directory with an index.js entry.  We'll do more here later.

NOTE: If you want to write es6-style webpack config, you can [do that](http://stackoverflow.com/questions/31903692/how-to-use-es6-in-webpack-config).

#### webpack.config.js/index.js

```javascript
var path = require('path');

module.exports = {
    entry: {
        index: path.resolve('index.js')
    },

    module: {
        loaders: [
            {
                test: /.jsx?/,
                exclude: /node_modules/,
                loader: 'babel-loader?cacheDirectory=' + path.resolve('.babelcache/' + process.env.NODE_ENV)
            }
        ]
    },

    output: {
        path: path.resolve('public'),
        publicPath: '/public/',
        filename: '[name].js'
    },

    resolve: {
        extensions: ['.js', '.jsx'],
        modules: [
            path.resolve('node_modules')
        ]
    }
};
```

This is a fairly straightforward webpack configuration, with one exception: we're providing babel with a NODE_ENV-specific cache directory.  Why are we doing this?  Because we are going to use the `modern-browsers/webpack2` preset for development, while we use the es2015 preset for production.  The `modern-browsers/webpack2` preset includes fewer transformations and allows us to take advantage of webpack tree shaking, but is likely to cause errors on all but the most recent of browsers, which is fine for developers, but probably not a great production plan.  We'll deal with production at the end, so let's just worry about babel's devel environment for now:

#### .babelrc

```json
{
  "sourceMaps": "both",
  "compact": false,
  "comments": true,
  "env": {
    "devel": {
      "presets": [
        "react",
        "modern-browsers/webpack2"
      ]
    }
  }
}

```

Go ahead and run the build, fire up index.html somewhere, and you'll see a successful build!

```shell
$ NODE_ENV=devel webpack
```

### Let's get steamy
Now that we're compiling an application, we're gonna need some tools to work on it, and we all know what that means: Hot Modules!

First, let's configure the dev server.

#### webpack.config.js/devserver.js
```javascript
var config = require('./index.js');

module.exports = {
    compress: true,
    hot: true,
    https: true, // SSL everywhere, even here!
    host: 'localhost', // if you're running this inside a VM, change to your guest's external facing hostname
    port: 9090,
    publicPath: config.output.publicPath,
    historyApiFallback: true,
    watchOptions: {
        aggregateTimeout: 300,
        poll: 1000
    }
};
```

And invoke the dev server:

#### server.js
```javascript
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config.js/index.js');
var dsconfig = require('./webpack.config.js/devserver.js');

new WebpackDevServer(webpack(config), dsconfig)
.listen(dsconfig.port, dsconfig.host, function(err, result) {
    if(err) {
        return console.log(err);
    }

    console.log('Listening on ' + dsconfig.host + ':' + dsconfig.port);
});
```

And make sure the dev server works as we expect:

```shell
$ NODE_ENV=devel node server.js
```

Now, just point your browser at [WDS](https://localhost:9090/) and make a change and you should see a live reload.  We'll have to install the react-hot-loader package next:

```shell
$ npm i --save-dev react-hot-loader@^3.0.0-beta
```

And configure it in webpack.  We have to change our entry and add our first plugins.

#### webpack.config.js/index.js
```javascript
entry: {
     index: [
         'react-hot-loader/patch',
         'webpack-dev-server/client?http://localhost:9090', // make sure you update this if you changed devserver.js
         'webpack/hot/only-dev-server',
         path.resolve('index.js')
     ]
},

plugins: [
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin()
],
```

We have to mount an AppContainer instead of our root component, so a tiny bit of refactoring in index.js is necessary:

#### index.js
```javascript
import React from 'react';
import {render} from 'react-dom';
import HelloWorld from './helloworld.js';
import {AppContainer} from 'react-hot-loader';

function mount() {
    render(<AppContainer><HelloWorld/></AppContainer>, document.getElementById('demo'));
}

if(module.hot) {
    module.hot.accept('./helloworld.js', (...args) => {
        mount();
    });
}

mount();
```

It's a slight bit more complicated to consume the Hot Module Replacement API, but that's generally all you need to do.  If you are mounting a Router, AppContainer should mount that.  HMR does not currently work with Webpack code-splitting, so we will implement that later as a production optimization.

At this point, you should be able to start the dev server and load up the site again, this time with working HMR!

```shell
$ NODE_ENV=devel node server.js
```

Now that we have a working configuration with Hot Module Replacement, we can start bolting on all the cool stuff we need and want!
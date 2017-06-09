---
title: Tool chain of React.js
date: 2017-04-26 19:15:11
tags:
- react
---
This is a runthrough of a typical workflow building a web app with React. It's served as a index map for people who are not yet into this area. The demonstrating repository is at https://github.com/robturtle/react-toolchain .

<!-- more -->

> A big picture of a web application:

![](https://www.dropbox.com/s/xn7rljdpm3hgz7u/web%20app%20hierarchy.svg?raw=1)

## NPM: the build system & package repository

> A newer and promising alternative is [yarn](https://yarnpkg.com/en/).

The package installation and building process is specified by file [package.json](https://docs.npmjs.com/files/package.json) at project root. Use `npm init` to build one.

Example output as below:

```json
{
  "name": "react-toolchain",
  "version": "0.1.0",
  "description": "A demonstration of React toolchain",
  "repository": "git@github.com:robturtle/react-toolchain.git",
  "author": "Yang Liu <jeremyrobturtle@gmail.com>",
  "license": "MIT"
}
```

### [Dependencies](https://docs.npmjs.com/files/package.json#dependencies) declaration

There're 2 major category of dependencies — `dependencies` and `devDependencies`, where latter are only needed when developing. We can adding one dependency along with actually installing it. The corresponding commands are:

- `npm install --save`  to install locally and adding dependency into `dependencies`.

  Short for `npm i -S`. (yarn: `yarn add`)

- `npm install --save-dev` to install locally and adding dependency into `devDependencies`.

  Short for `npm i -D`. (yarn: `yarn add -D`)

The meaning of "installing locally" is to install it under the folder "node_modules" under current location.

> Use `npm i -g` to install a pacakge globally, i.e. in a system-wise location.

With a valid `package.json`, another coder is able to reinstall all the dependencies on another machine. The only thing he need to do is typing `npm i` in the same directory.

### [Run scripts](https://docs.npmjs.com/misc/scripts)

NPM also served as a (kind of, yet constrained) build system where command lines in the `scripts` property can be executed using `npm run`. There're serveral preset task names which connect to certain hooks (like `prepublish`, `postinstall`, etc) or some common process (like `start`, `test`).

- The universal way of running a script is by `npm run <script-name>`.
- These scripts are executed directly by the command `npm <script-name>`:
  - `npm start` (default value: `"node server.js"`)
  - `npm stop`
  - `npm restart` (If not given, runs `stop` then `start`)
  - `npm test`

Keep in mind that the script "start" is nothing different between any other scripts (except for the hooks), meaning that you can assign any arbitrary command to it.

> NOTE: Besides the system path, the `binroot` paths of each local package are also included and at higher priority so that all their exported executables are accessible by the script runner.

You can compare the PATH between which internally in `npm run` and your regular shell settings by adding this command:

```json
{
  "scripts": {
    "testpath": "echo $PATH"
  }
}
```

The script executor is `sh`.

### [Versioning](https://docs.npmjs.com/cli/version)

By convention [semantic versioning](http://semver.org/) is used. The `npm version` allows us change the versioning information in `package.json` conveniently. And a pre-changing hook "version" can also be added into the "scripts" property.

## Babel: the JS to JS compiler

- The executables is in package `babel-cli`, including `babel` the compiler, and `babel-node` the ES2015 compliant node. (check this [example](https://github.com/babel/example-node-server) for the real world node servers)
- The compiling rules are orgainized as plugins.
- Some common used plugins are grouped into presets.
- Configuration can be set in `.babelrc` file or "babel" property in `package.json`.

Some common used presets:

- env: automatically choosing the best ES standard. (replacing the older "latest")
- react: compiling React.
- stage-x: refer to the [document](https://babeljs.io/docs/plugins/#presets-stage-x-experimental-presets-).

```json
{
  "babel": {
    "presets": [
      "env",
      "react",
      "stage-2"
    ]
  },
  "devDependencies": {
    "babel-cli": "^6.24.1",
    "babel-preset-env": "^1.4.0",
    "babel-preset-react": "^6.24.1",
    "babel-preset-stage-2": "^6.24.1"
  },
  "scripts": {
    "build": "babel src -d lib"
  },
  "dependencies": {
    "react": "^15.5.4",
    "react-dom": "^15.5.4"
  }
}
```

At [this commit](https://github.com/robturtle/react-toolchain/commit/0049382a8b8c1bba4bb3252c0d14e2899f734fd7), we can run `npm run build` to compile JS files from `src` folder into `lib` folder.

## Webpack: the module bundler

> The original docs for webpack is been criticized for a long time. Considering using this ["community docs"](https://webpack.js.org) instead.

The original purpose of webpack is to serve as a performance tool trying to minimize the data flow from  the server to the client. It interprets all the `import`/`require` statements in the source files and assemble all the needed files in "chunks". Chunks are just regular browser compliant JavaScript files, and our product HTML file is going to require those chunks instead.

It's important to know that those statements are read and consumed by webpack, so we don't have to worry about its againsting current language standard. And we also don't have to let babel translate such statements. The following configuration shuts down that.

```json
{
  "babel": {
    "presets": [
      [
        "env",
        {
          "modules": false
        }
      ],
      "react",
      "stage-2"
    ]
  }
}
```

This [example](https://github.com/robturtle/react-toolchain/commit/c64370aa976ed5238d49dee1c4a8384ad0fd46e8) shows the basic usage of webpack. Use command `npm run bundle` to create the bundled file under `dist` folder. You can execute the bundled code by `node dist/bundle.js`.

### Configuration file

[Configuration file](https://webpack.github.io/docs/configuration.html) can be stored in file `webpack.config.js` and after that the simple `webpack` command is enough to do the job.

This [example](https://github.com/robturtle/react-toolchain/commit/b7f906dac049549db5a5d80a55b0773e36b4b50c) demonstrate the use of the config file.

```javascript
module.exports = {
  context: __dirname + "/lib",
  entry: "./try-webpack.js",
  output: {
    path: __dirname + "/dist",
    filename: "bundle.js"
  }
};
```

#### Libraries splitting

By walking through this [guide](https://webpack.js.org/guides/code-splitting-libraries/), we will finally reach a common practice which split our code base into 3 parts, namely application code, vendor code, and webpack's manifest. This solution ensure a unchanged part can be reused and will not be bundled repeatedly.

```javascript
var path = require('path');
var webpack = require('webpack');

module.exports = function(env) {
  return {
    context: path.resolve(__dirname, 'lib'),
    entry: {
      main: './try-webpack.js'
    },
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: '[name].[chunkhash].js'
    },
    plugins: [
      new webpack.optimize.CommonsChunkPlugin({
        name: 'vendor',
        minChunks: function (module) {
          return module.context && module.context.indexOf('node_modules') !== -1;
        }
      }),
      new webpack.optimize.CommonsChunkPlugin({
        name: 'manifest'
      })
    ]
  };
};
```

[Example commit](https://github.com/robturtle/react-toolchain/commit/25b6a2425974ad8936033fb1597344d1bd2f370f).

You can check for yourself by repeatedly run `npm run bundle`, and convince yourself that webpack didn't generate new chunks with new hash in the `dist` folder.

> In the HTML example on the [deterministic hashes](https://webpack.js.org/guides/caching/#deterministic-hashes) section, the `//<![CDATA` syntax is called `cdata-js` and its usage is discussed at [StackOverflow](http://stackoverflow.com/questions/66837/when-is-a-cdata-section-necessary-within-a-script-tag).

### Loaders

What the webpack do more (maybe it should not) than a bundler is we can introduce and chain preprocessors to certain kinds of files, making webpack also a build system to automate the process.

One obvious example is that we can use `babel-loader` to run `babel` before we bundle the JavaScript files.

```javascript
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        include: path.resolve(__dirname, 'src'),
        loader: 'babel-loader'
      }
    ]
  }
```

With [this example](https://github.com/robturtle/react-toolchain/commit/b9d4b81f0eb5eb67e585b79b05baeee0df1cb649), we use babel as the preprocessor of JS files. So the intermediate `lib` folder is no longer needed since webpack can directly deal with the original source files.

With loaders, webpack is not constrained to deal with only JavaScript files but anything that can be translated into JS, meaning almost everything: JSON, CSS, JPEG, etc.

### Troubleshoots

#### Node environment

When bundling node files, use `node` options to make sure node environment works as expected. What againsting the intuition is that you [have to set the value as false](https://github.com/webpack/webpack/issues/1599#issuecomment-186841345) to enable that predefine variable.

```javascript
// serverConfig
{
  target: 'node',
  node: {
    console: false,
    global: false,
    process: false,
    Buffer: false,
    __filename: false,
    __dirname: false,
  },
}
```

#### Webpack can't bundle itself

No, [webpack can't bundle itself](https://github.com/webpack/webpack/issues/1434). But considering the reason you want to do it might be just adding a babel preprocessor for your node codes. And this will require a lot error-prone webpack configurations. So, why not just using babel instead? And if you insist using webpack, consider [setting all node_modules as externals](https://www.npmjs.com/package/webpack-node-externals).

#### MultiCompiler.outputPath

> I've modify the implementation of webpack-dev-middleware to avoid this problem. PR is expected after I update its tests.

Let's say you have configurations grouped as an array. When passed to `webpack()` it will generate a `MultiCompiler`. If the `outputPath` are different among configurations, the `outputPath` will be set as the lowest common root path: [code](https://github.com/webpack/webpack/blob/master/lib/MultiCompiler.js#L36)

```javascript
// https://github.com/webpack/webpack/blob/master/lib/MultiCompiler.js#L36
Object.defineProperty(this, "outputPath", {
		configurable: false,
		get: function() {
			var commonPath = compilers[0].outputPath;
			for(var i = 1; i < compilers.length; i++) {
				while(compilers[i].outputPath.indexOf(commonPath) !== 0 && /[\/\\]/.test(commonPath)) {
					commonPath = commonPath.replace(/[\/\\][^\/\\]*$/, "");
				}
			}
			if(!commonPath && compilers[0].outputPath[0] === "/") return "/";
			return commonPath;
		}
	});
```

This will cause trouble when combining with `webpack-dev-middleware`, for its middleware will concat the file path using this common path rather than iterating all the sub-compilers' `outputPath`.

[code](https://github.com/webpack/webpack-dev-middleware/blob/master/middleware.js#L39)

```javascript
// https://github.com/webpack/webpack-dev-middleware/blob/master/middleware.js#L39
function webpackDevMiddleware(req, res, next) {
  ...
  var filename = getFilenameFromUrl(context.options.publicPath, context.compiler.outputPath, req.url);
  ...
}
```

A good solution is for multiple configurations, we choose web root as is common path. For files out of web root such as `server.js`, we can use `filename: ../server.js` to specify the correct destination. Another benefit of it is that `server.js` will be keep away from public by nature.

## Express: one backend solution

> Note this is only one of all backend solutions since React is just a view level library and it can work with almost all backends.

### Node.js

Checkout [this article](http://voidcanvas.com/node-vs-browsers/) for the environment difference between node and browser. In few words, node is a JS environment with a module system, different set of predefined global variable, and of cause without DOM-related stuffs.

A bad thing for node is its standard libraries like `fs` are written in the callback style, which makes node a perfect place for callback hell. Hence for the modern programmers, [promise style apadters](https://github.com/normalize/mz) may be desirable.

> I used to think the async things are mainly for better IO scheduling, till JS told me you could just use it for a better looking LOL.

### [Express](https://expressjs.com/en/starter/hello-world.html)

Express is a very simple application framework with very few concepts. Basically it can be put in this way:

- Express use *handlers* to accept requests and send back responses;
- Handlers are normal functions with certain signatures;
- Handlers can be mounted at  some *mount point* (i.e. path prefix);
- Handlers can be chained by the definition order;
  - Handlers can use `next()` to pass to next handler for the same route. Such handlers are called *middlewares*;
  - Handlers can send back response which by doing so will terminate the whole processing chain.

That's pretty much about it. Check out `express-generator` for a common organization of a web app.

### [Pug](https://pugjs.org/api/getting-started.html): a template engine

A template engine is to extract the common part among different responses generated by the server. 

One important thing to deal with template engine is take care of the escaping. Normally all the string literals are escaped in case of XSS attacks. That's good but just make sure it generate the contents as you expected.

### [Optional] Isomorphic rendering

The main idea about isomorphic rendering is for the first time the page is rendered from on server side rather than on the browser so that users would not waiting too long for his first interaction. However, as we could see that most backends naturally don't have a browser environment so we have to kind of simulate it. The term "isomorphic" is to infer that both ends have the identical environment. However, we got to admit that currently most of the simulation have many flaws and when dealing with libraries without Server-side Rendering support it almost like a hack. It's still a viable optimization though, but reconsider the sitution before actually adopting it.

[See also (CN)](https://github.com/camsong/blog/issues/8)

## Development

> This chapter is inspired by the project [react-starter-kit](https://github.com/kriasoft/react-starter-kit). Which uses `webpack-hot-middleware` and `Browsersync` as the development server.

One golden goal for developing is the changes can be applied on the fly. This means the source code is being *watched* and a detection of modification will trigger a refresh of running application. Since some of the targets are generated in the compile time (jsx transformation, bundled file, etc), extra efforts are required to *hot replace* them. This is called *Hot Moduel Replacement* (HMR).

> Webpack provides both watch mode and HMR. Please don't get confused with them. They both taking effects when modification was detected on the source code. The difference is in the watch mode webpack compiles bundles to the file system, which make no effect on the running process; while the HMR makes webpack compile them in the memory and replace the old mapping.

### [Webpack HMR](https://github.com/glenjamin/webpack-hot-middleware)

The official solution is a express middleware `webpack-dev-middleware` which hot reloads only on `webpack-dev-server`. Another one is `webpack-hot-middleware` which adding HMR feature to any compatible existing  servers. Although `webpack-dev-middleware`'s HMR is constrained, its hosting function is still valuable. For development we could combine these two.

To enable the `webpack-hot-middleware`, the required modifications are:

- Adding `webpack-hot-middleware/client` into webpack entries;
- Applying `webpack.HotModuleReplacementPlugin` globally;
- Passing a webpack compiler to construct the middleware.

#### HMR for entry point

HMR works on module level and entry file is the last stop of the accepting chain which by default rejects updates. In order to enable HMR for entry point also, add following code in the entry file. See also the [official docs](http://webpack.github.io/docs/hot-module-replacement-with-webpack.html).

```javascript
// https://github.com/glenjamin/webpack-hot-middleware/issues/43#issuecomment-155697146
if (module.hot) {
  module.hot.accept();
}
```

### [React HMR](https://webpack.js.org/guides/hmr-react/)

The package `react-hot-loader` supports this feature. The required modifications are:

- Adding `react-hot-loader/patch` into webpack entries;
- Applying `react-hot-loader/babel` plugin to `babel-loader`;
- Applying `webpack.HotModuleReplacementPlugin` globally.

> We need version 3.+ here, so install it with `npm i -D react-hot-loader@next`

### [Browsersync](https://www.browsersync.io/docs): a watch server

Browsersync is a live-reload server which refreshes the hosting files and, with the middlewares above plugged in, replacing code when necessary. 

The whole picture of the HMR patching is as follows:

![](https://www.dropbox.com/s/4nth9pu0led4rbt/react%20dev%20server.svg?raw=1)

Since HMR is needed only when developing, we can leave the original webpack configuration unchanged and use a script to do patch & host.

At [this commit](https://github.com/robturtle/react-toolchain/commit/5ed097fff39969e92d0bddc8ad0ed8a8ae9223cf), the development server is setup.

#### Dealing with Content Security Policy

Since Browsersync's behaviors are actually injections which might be rejected by the browser because of the CSP. Refer to this [StackOverflow discussion](http://stackoverflow.com/questions/30233218/browser-sync-is-blocked-by-chrome-csp) for the solution. Considered we will never expose our development server, it's OK to totally disable the CSP for it.

## Redux — a state manager

## Communication with backend

### XMLHTTPRequest — the traditional way

### Web socket — the modern way

## Authentication

### Cookies

### JSON Web Token

### OAuth

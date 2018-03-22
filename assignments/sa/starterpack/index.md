---
layout: page
title: SA3 Webpack + Babel + eslint
published: true
---


## Overview

Today we'll be learning about a set of tools for making your code compact and pretty.  So far you've been using html and css in separate files and the sites have been fairly compact — but as your sites grow more complex and include more and more libraries there has to be a way to manage it all.  We'll introduce a set of tools to help you build complex sites with many module dependencies easily.

💻 : run in Terminal<br>
🚀 : a step to not forget


🚀 To start grab the github classroom link to start a new repository. Work in the `master` branch for now, you'll see why later.


## [yarn](https://yarnpkg.com/en/) and [NPM](https://www.npmjs.com/)

![](img/npm.jpg){: .fancy .tiny }

![](img/yarn.jpg){: .fancy .tiny }


[NPM](https://www.npmjs.com/) is the dependency package manager for Node.js and the javascript ecosystem in general.  While we won't exactly be using node just yet today, we will be using the package manager to install various javascript tools. Node.js is a javascript platform for running javascript outside of a browser for developing server side applications and we will be using it extensively later on. You can do things like `npm install somepackage` and it will download that package and add it to your project. Great. WTF is [yarn](https://yarnpkg.com/en/)?  Yarn is a faster/better npm.  It uses the same repository of packages.  You can use it practically interchangeably - just decide which you will use and try to stick to it. Yarn is newer and we're going to try it out for 18S and see how it goes.


### Installation

🚀 NPM comes with Nodejs so we'll first install that.

#### os x:

```bash
brew update
brew upgrade
brew install node
brew install yarn
```

Make sure you have the right versions of node/npm/yarn:

```bash
node --version
#v9.8.0
npm --version
#5.6.0
yarn --version
#1.5.1
```

Newer than these is fine, but if you have older ones you might need to force uninstall/reinstall:

If you run into problems here you can try forcing a reinstall:

```bash
sudo chown -R $USER /usr/local  #in case you had ever run brew with sudo
brew uninstall --force node
brew install node
```

#### windows, other:

install from [nodejs.org](https://nodejs.org/en/). *you may need to restart git shell afterwards*

### start a node project

🚀 We will start a node project even though we will only be using the node package manager.

```bash
#make sure you are in your cloned workspace

yarn init #accept all the default answers if you want
```

All this did was create a barebones `package.json`  file.  This file defines a nodejs based project and is required.  It will primarily list all of the dependencies of our project.  It will also define some convenient scripts for us.

### Index file

🚀 Lets create a simple javascript file to get us started. It is good practice to keep your app in a subdirectory of the project so lets create that and call it `src`.

```bash
mkdir src #create directory
touch src/index.js   #creates an empty file
```

🚀 Now in **Atom** lets give our app something to do.

```javascript
console.log('starting up!');
```

### yarn start

Now lets make it so we can run our app!

🚀 Edit your `package.json` file:

* change:
  `"main": "index.js"` to `"main": "src/index.js"`
* add/modify a `"scripts"` section:
  `"scripts": { "start": "node src/index.js" }`

Now in 💻 you can:

```bash
💻 yarn start

yarn run v1.5.1
$ node src/index.js
starting up!
✨  Done in 0.16s.
```

Great, you just ran a little bit of javascript on the computer without a browser.


## Webpack

### Install Webpack for the project

![](img/what-is-webpack.png){: .fancy .small}

[Webpack](https://webpack.js.org) is a build tool.  What it does is take all your dependencies (all the various files and libraries that your project is using) and bundle them up together — but in a smart way where it will only bundle up the things you need. It can do any preprocessing that you need, such as converting various code syntax and supports hot-reloading, so you never have to refresh the page manually again.  Most cleverly it actually analyzes your code and figures out what you are doing.  It does [tree shaking](https://webpack.js.org/guides/tree-shaking/) to only include code that actually used and discards everything else.

🚀 Lets install webpack:

```bash
yarn add webpack webpack-dev-server webpack-cli --dev
```

`yarn add` is how you add most any javascript library to your project.  It will install in a directory `node_modules` in your project.  Note: `yarn install` would install it but not add it to your `project.json`.  Generally you want to add it to your project so that later if others were to download the project they could install the same packages.

Note: when you pass `--dev` it will add those modules to your `package.json` in a `devDependencies` section.  In this case we are saying these are *development* only dependencies. So you will now notice new lines declaring those dependencies in `package.json` as `"devDependencies"`. Leave off the `--dev` flag if you are adding in a library you want to always include.

:warning: `node_modules` is **not** something you need to commit to git.  These are libraries that you can assume will be available and so as long as you have saved the dependencies to the `package.json` file anybody can later install all of the required packages by simply running `yarn install`.

🚀 Let's make sure we don't do that now.  Create a file called `.gitignore` and add the following:

```bash
public/build
node_modules
yarn-error.log
```

### Now what?  

Let's set up a simple webapp so we can see start using webpack.

🚀 Create an `index.html` in the project root folder (not in src/):

```html
<!DOCTYPE html>
<html>
  <head>
    <title>starter pack</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet">
  </head>
  <body>
    <div id="main">Howdy</div>
  </body>
  <script src="build/bundle.js"  charset="utf-8"></script>
</html>
```

Wait what is this `build/bundle.js`? Good q. That is the file we are going to have webpack build for us.

🚀 Lets add in [JQuery](http://jquery.com/) really quick. We won't really be using much jquery this term, but for a quick example it will suffice.

```bash
💻 yarn add jquery   #notice there is no --dev
```


🚀 Change your `index.js` to the following:

```javascript
const $ = require('jquery');

$('#main').html('Here we go!');

```

All this will do is find the element with the `id` of `main` and change the content to 'Here we go!'.


### Configure webpack

🚀 Lets give webpack a very basic configuration. Create a file called `webpack.config.js` with these contents:

```javascript
const path = require('path');
module.exports = {
  entry: [ './src' ], // this is where our app lives
  devtool: 'source-map', // this enables debugging with source in chrome devtools
  output: {
    path: path.join(__dirname, 'public/build'), ////output files go to public/build
    publicPath: '/build/',
    filename: 'bundle.js', //package it all up in one bundle
  },
  devServer: {
    contentBase: path.join(__dirname, 'public'), //any static files should go in public
    historyApiFallback: true, // useful later
  },
};
```

Note, we are now doing something a bit different with our directory structure. We've added `public`.

The overall project structure will now look like:

```bash
src/ #all the js and scss source files for your app
public/  #where your static assets will live like images and your index.html file
public/build #autogenerated directory where webpack will put your bundle files
```

🚀 Before we go on, let's move the `index.html` file so that it lives in the right place.

```bash
mkdir public
mv index.html public
```

🚀 now try running webpack directly from the commandline: `./node_modules/.bin/webpack --mode development`

```bash
💻 ./node_modules/.bin/webpack --mode development
Hash: 10aaf12a7b59b4cde0dc
Version: webpack 4.2.0
Time: 466ms
Built at: 3/21/2018 11:05:35 PM
        Asset     Size  Chunks             Chunk Names
    bundle.js  269 KiB    main  [emitted]  main
bundle.js.map  351 KiB    main  [emitted]  main
Entrypoint main = bundle.js bundle.js.map
[./src/index.js] 62 bytes {main} [built]
   [0] multi ./src 28 bytes {main} [built]
    + 1 hidden module
```
{: .example}

Great! It is building our index file in addition to one hidden module which isn't ours. In this case this is jquery! How'd it know?  Well it analyzed our `index.js` and saw our `require` statement, then it located the module in `node_modules` and included it.


### webpack-dev-server

Shall we look at what we have done so far?  We could start up a python webserver but webpack comes with a dev server built in, lets use that instead.

🚀 Edit your `package.json` again and change `"start": "webpack-dev-server --inline --mode=development"`

*Note: on windows the syntax is a tiny bit different you need `SET NODE_ENV=development & webpack-dev-server --inline`*

🚀 Now you can simply run `yarn start`:

```bash
💻 yarn start

yarn run v1.5.1
$ webpack-dev-server --inline --mode=development
ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /build/
ℹ ｢wds｣: Content not from webpack is served from /Users/tim/Sandbox/CS52/18s/assignments/starter/public
ℹ ｢wds｣: 404s will fallback to /index.html
ℹ ｢wdm｣: Hash: 7ac205b0de50609287b4
[...]
```

![](img/webpack-dev-server01.png){: .fancy .tiny}

Ok,  we're now loading our page through webpack-dev-server and also loading in our javascript and changing some HTML!


### Hot reloading

Wait, we were promised hot-reloading, whatever that means!

Ok, lets config that. This will make it so your page reloads every time you save any files that are part of the javascript app (`index.html` is not included however).

🚀 Edit your `package.json` and add `--hot` to the end of the the `start` command.

`ctrl + c` out of your webpack-dev-server and run `yarn start` again.

Try changing your `index.js` file to change the text of `#main` to something else.  You should see those changes happen immediately!

On the terminal you'll see anytime you change the js file that the webpack recompiles automatically.


## Babel

![](img/babel.png){: .fancy .small }

We've talked a little bit about various JS versions.  Babel is a transpiler that converts newer es6+ javascript syntax to more compatible es5 syntax that browsers actually understand (browsers are typically behind in their ECMAScript support). If you are curious you can try it out at [babeljs](https://babeljs.io/repl/)


### Install and configure Babel

🚀 Let's install babel and the webpack babel-loader.

```bash
yarn add babel-core babel-loader babel-preset-env babel-polyfill --dev
```

Babel needs to be configured for the particulars of what feature set of ECMAScript you want.

🚀 Create a file called `.babelrc` in your project:

```json
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions"]
      }
    }]
  ]
}
```


🚀 Finally add the following to your `webpack.config.js` file:

```javascript
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /node_modules/,
      use: [
        { loader: 'babel-loader'}
      ]
    }
  ],
},
```

🚀 and change your `entry` to be `entry: [ 'babel-polyfill', './src' ],`
this loads in some nice extra babel functions before we get into our app so we can use them.

🚀 `ctrl + c` out of your webpack-dev-server and run `yarn start` again to pick up the changes to the config files.

###  use some es6

🚀 Let's change our `index.js` to do something a tiny bit more interesting and we'll use ES6 syntax.

```javascript
// change require to es6 import style
import $ from 'jquery';
```

🚀 Now write some javascript that updates the `#main` element every second to:  `You've been on this page for ${num} seconds.`.  This will be a simple counter.  

* you could use `setInterval` to call a function
* if you have anonymous functions please use arrow notation `() => { }`
  * in fact you might run into scope problems if you don't use arrow notation as arrow notation does better things with scope than regular anonymous functions.
* avoid using `var` (remember `let` and `const`)

Excellent, now your page is keeping count of how long its been since you loaded / reloaded it. **SUPER USEFUL!**


## Linting

![](img/eslint.png){: .fancy .small}

Ok, how about we add in linting. Linters are code parsers that check your code for syntax errors, common style mistakes, and makes sure that your code is clean and follows some best practices. Linters help save time, detect bugs, and improve code quality. Linters exist for all types of languages and even markdown such as HTML, CSS, and JSON. Linters are especially helpful for detecting syntax errors while using dynamically typed languages in lightweight editors such as Atom and Sublime.

In *Atom* install the `linter-eslint` plugin.

![](img/atom-eslint.png){: .fancy .small}


The recommended linter plugin for javascript is [Eslint](http://eslint.org/).

🚀 Install `eslint`:

```bash
yarn add --dev eslint babel-eslint eslint-loader
yarn add --dev eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react
```

`eslint` comes with a series of plugins for various javascript packages.  In particular Airbnb's style guide is one that we will be **strongly** requiring:  [eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb). We'll be changing some of the rules and the rules are flexible (you may disable some of the more annoying ones).  

🚀 After installing `eslint`, create an eslint configuration file `.eslintrc` with something like the following. This file instructs eslint to use the airbnb rules and overrides some of the common rules that I found particularly obtrusive. You are allowed to turn off certain rules if you prefer but take a look at the documentation and read about why the rule was implemented first.

```json
{
    "extends": "airbnb",
    "parser": "babel-eslint",
    "env": {
        "browser": true,
        "es6": true
    },
    "rules": {
        "strict": 0,
        "quotes": [2, "single"],
        "no-else-return": 0,
        "new-cap": ["error", {"capIsNewExceptions": ["Router"]}],
        "no-console": 0,
        "import/no-unresolved": [2, { "caseSensitive": false } ],
        "no-unused-vars": ["error", { "vars": "all", "args": "none" }],
        "no-underscore-dangle": 0,
        "arrow-body-style": 0,
        "one-var": ["error", { "uninitialized": "always", "initialized": "never" }],
        "one-var-declaration-per-line": ["error", "initializations"],
        "max-len": ["error", 200],
        "no-extra-parens": 0,
        "no-restricted-syntax": [
          0,
          "DebuggerStatement"
        ],
        "no-debugger": "warn"
    }
}
```

🚀 Now go to Atom -> Preferences -> Packages -> linter-eslint -> Settings, and check "Fix Errors on Save". Super useful. This will fix indentation problems and some other things automatically whenever you save.

When you see errors such as below:

![linter error](img/linter-error.png){: .fancy .small}

You can click on the definition of the error to learn more.  Note: many of these show up as errors but are not compiler errors like you are used to in Eclipse.  The browser or node may still run the code fine — however it is recommended that you fix these errors or learn about what the errors mean and disable them only if you disagree with that particular rule stylistically. I left the above errors so you could fix them!

🚀  Now let webpack include eslint! In your `webpack.config.js` file, add `eslint-loader` to the loaders section. This will run your code through eslint before compiling it – thus making sure it is all good and giving you warnings and errors otherwise.

From here on your assignments will all use an `.eslintrc` file as well as a `.babelrc` file.  Adhering to a code style and ES6 will at first seem annoying but you'll find ES6 to be a much more beautiful version of the language and over time will grow to appreciate the linting rules as well. This is also pretty much industry standard.


## SASS Webpack loader

Lets make webpack handle CSS for us also, and we'll even upgrade that to [SASS](http://sass-lang.com/guide)!

![](img/sass.png){: .fancy .small}


🚀 Add loading the output `style.css` file into your `index.html`:

```html
<head>
  <link rel="stylesheet" type="text/css" href="build/style.css">
```

*Note: loading your own stylesheet should be the last in order of loading - so that your rules override previous ones.*

🚀 The only loader that comes built-in to webpack is javascript so we need some more packages:

```bash
yarn add css-loader sass-loader postcss-loader node-sass style-loader extract-text-webpack-plugin --save-dev
```

🚀 Now you need to modify the `webpack.config.js` file to include the css/sass loaders. This part gets a bit complicated.  We are including both css and sass loaders, and doing a little bit of trickery with the `extract-text-webpack-plugin`.  The reason for this is that by default webpack is really all about the javascript. That means if it was going to be in charge of CSS it would load it into the page using javascript. Which could cause annoying flashing as styles get applied.  We want the CSS to be output separately as text into `style.css` rather than into `bundle.js` so that we can load it in `<head>`.



```javascript
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const extractSass = new ExtractTextPlugin({
  filename: 'style.css',
  disable: process.env.NODE_ENV === 'development',
});

module.exports = {
  entry: ['./src'],
  devtool: 'inline-source-map',
  output: {
    path: path.join(__dirname, 'public/build'),
    publicPath: '/build/',
    filename: 'bundle.js',
  },
  devServer: {
    contentBase: path.join(__dirname, 'public'),
    historyApiFallback: true,
  },
  module: {
    rules: [
      {
        test: /(\.scss)$/,
        use: extractSass.extract({
          use: [
            {
              loader: 'css-loader',
              options: { sourceMap: true },
            },
            {
              loader: 'sass-loader',
              options: { sourceMap: true },
            },
          ],
          fallback: 'style-loader', // use style-loader in development mode
        }),
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: ['babel-loader', 'eslint-loader'],
      },
    ],
  },
  plugins: [
    extractSass,
  ],
};
```

🚀 Create a  `src/style.scss` file and add in some style!

```css
body {
  font-family: "Roboto", sans-serif;
  background: darken(white, 0.2);
}
```

🚀 Now, you need to have webpack include it, so add `import './style.scss'` to your `index.js` file! Remember webpack won't include anything that isn't used by your app directly.

🚀 Restart your webpack server and see if you style changes are picked up.  Note how you can change styles and the hot-reloading kicks in!

*Note: you will see that when you run locally as you are now that chrome devtools will tell you that the style.css file is a 404 error. You can ignore this error for now. The webpack-dev-server is injecting the styles via javascript when in development mode so that it can hot reload them, however in production mode that file will be automatically generated.*

## AutoPrefixing!

As you've been playing with CSS you've probably noticed the internet mentioning vendor prefixes such as:

```
-webkit-transition: all 4s ease;
-moz-transition: all 4s ease;
-ms-transition: all 4s ease;
-o-transition: all 4s ease;
transition: all 4s ease;
```

Due to varying browser support of features these are often associated with newer CSS. This should be something easy for a machine to solve rather than us humans manually remembering to make a mess of our CSS code, and indeed someone has.  Enter [autoprefixer](https://github.com/postcss/autoprefixer).

This is a module that we can incorporate and have webpack automatically prefix our canonical css to work in all browsers. It is actually a plugin for a [loader](https://webpack.js.org/concepts/loaders/), but regardless, you can see how it is configured here.

🚀 Edit your `webpack.config.js` file once again and add in:

```js
  {
    loader: 'postcss-loader',
    options: {
      plugins: () => [autoprefixer()],
    },
  },
```

in between your `css-loader` and your `sass-loader` objects.

🚀 and add in `const autoprefixer = require('autoprefixer');` to the top of the file.

🚀 Guess what you need to `npm install --save-dev` as well?  Yup, `autoprefixer`.



## Deployment

How would you deploy a webpack setup such as this?

Well we can continue using gh-pages!  But we can make it even easier. We're going to add in a little package to help.

```bash
💻 npm install gh-pages
```

🚀 Add the following to your `package.json` `scripts` section:

```javaScript
"build": "WEBPACK_ENV=production webpack --colors",
"clean": "rimraf public/build",
"deploy": "npm run build; gh-pages -d public; npm run clean"
```

Now you have a deploy npm script you can run!

```
💻 npm run deploy
```

This will build your app into the build directory and the push all the right stuff to github to the gh-pages branch and everything. However!  This will not push your code that is still in `master` you will still need to `add`, `commit`, and `push` all of your actual code from `local/master` to `origin/master`.  All the deploy script does is help keep your published website in `gh-pages` clean and separate from your source code.

🚀 git `add`, `commit`, `push` all the code that is currently in your `master` branch to your `origin master` branch. You should now have:  `master` branch with `/src` and all your webpack and package files, and you don't have to worry about a `gh-pages` branch because that is fully automatic with the deploy script.


## A word about debugging

As always you should use Chrome Dev Tools with the console as the first line of defense. Checking for errors there is the first thing you should look for.  You can also use breakpoints.  The recommended way to is locate the source in devtools and click the line number to add the breakpoint. Because of webpacks magic all sources end up in webpack:// *(Sources -> weback:// -> . -> src)*

![](img/chrome-devtools.png){: .fancy }

You can also use the statement `debugger;` in your code, but that is somewhat frowned upon as you might forget and leave one in a piece of code that doesn't get executed often causing mayhem.


## Frontend Stater Pack is ready to go!

You now have a nicely set up starter pack that you can do

### To Turn In

1. Checklist:
  * webpack-dev-server starts and serves pages
  * hot-reloading works as advertised
  * babel is configured
  * eslint is configured
  * webpage displays and counts seconds
  * js is es6 and linted without errors
1. A short answer response to:
  * describe the environment you set up.
  * any questions about what/why/how that you feel are unresolved?



## Resources:


* [getting started with webpack2](https://blog.madewithenvy.com/getting-started-with-webpack-2-ed2b86c68783)
* [webpack docs](https://webpack.js.org/)
* [babeljs](https://babeljs.io/)
* [State of the Art Javascript 2016](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b#.26xksjxvt)

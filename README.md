# ReactJS Webpack Tutorial

Documentation below has been pulled from https://webpack.js.org/guides/getting-started/

First initialize npm, install webpack locally, and install the webpack-cli (the tool used to run webpack on the command line):

```
npm init -y
npm install webpack --save-dev
npm install webpack-cli --save-dev
```

Now we'll create the following files and their contents:

**src/index.js**

```javascript
function component() {
  const element = document.createElement('div');

  // Lodash, currently included via a script, is required for this line to work
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

**index.html**

```html
<!doctype html>
<html>
  <head>
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

We also need to adjust our package.json file in order to make sure we mark our package as private, as well as removing the main entry. This is to prevent an accidental publish of your code.

* Remove _"main": "index.js",_ from the package.json
* Add _"private": true,_ into the package.json in the same spot

At this point you will want to commit your changes with the following command:

```
git commit -am "Initial version"

```

In this example, there are implicit dependencies between the <script> tags. Our index.js file depends on lodash being included in the page before it runs. This is because index.js never explicitly declared a need for lodash; it just assumes that the global variable _ exists.

There are problems with managing JavaScript projects this way:

* It is not immediately apparent that the script depends on an external library.
* If a dependency is missing, or included in the wrong order, the application will not function properly.
* If a dependency is included but not used, the browser will be forced to download unnecessary code.

Let's use webpack to manage these scripts instead.

## Creating a Bundle

First we'll tweak our directory structure slightly, separating the "source" code (/src) from our "distribution" code (/dist). The "source" code is the code that we'll write and edit. The "distribution" code is the minimized and optimized output of our build process that will eventually be loaded in the browser.

```
mkdir dist
mv index.html dist/index.html
```

To bundle the lodash dependency with index.js, we'll need to install the library locally:

```
npm install --save lodash
```

When installing a package that will be bundled into your production bundle, you should use _npm install --save_. If you're installing a package for development purposes (e.g. a linter, testing libraries, etc.) then you should use _npm install --save-dev_. More information can be found in the [npm documentation](https://docs.npmjs.com/cli/install).

Now, lets import lodash in our script.

**src/index.js**

```javascript
import _ from 'lodash';

function component() {
  const element = document.createElement('div');

  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

Now, since we'll be bundling our scripts, we have to update our _index.html_ file. Let's remove the lodash _<script>_, as we now _import_ it, and modify the other _<script>_ tag to load the bundle, instead of the raw _/src_ file:

**dist/index.html**

```
<!doctype html>
<html>
 <head>
   <title>Getting Started</title>
 </head>
 <body>
   <script src="main.js"></script>
 </body>
</html>
```

In this setup, index.js explicitly requires lodash to be present, and binds it as _ (no global scope pollution). By stating what dependencies a module needs, webpack can use this information to build a dependency graph. It then uses the graph to generate an optimized bundle where scripts will be executed in the correct order.

With that said, let's run npx webpack, which will take our script at src/index.js as the entry point, and will generate dist/main.js as the output. The npx command, which ships with Node 8.2/npm 5.2.0 or higher, runs the webpack binary (./node_modules/.bin/webpack) of the webpack package we installed in the beginning:

```
npx webpack
```

Open index.html in your browser and, if everything went right, you should see the following text: 'Hello webpack'.

You will want to commit your changes with the following command:

```
git commit -am "Creating a Bundle"

```

## Modules

The import and export statements have been standardized in ES2015. Although they are not supported in most browsers yet, webpack does support them out of the box.

Behind the scenes, webpack actually "transpiles" the code so that older browsers can also run it. If you inspect dist/main.js, you might be able to see how webpack does this, it's quite ingenious! Besides import and export, webpack supports various other module syntaxes as well, see Module API for more information.

Note that webpack will not alter any code other than import and export statements. If you are using other ES2015 features, make sure to use a transpiler such as Babel or Bublé via webpack's loader system.

## Using a Configuration

As of version 4, webpack doesn't require any configuration, but most projects will need a more complex setup, which is why webpack supports a configuration file. This is much more efficient than having to manually type in a lot of commands in the terminal, so let's create one:

**webpack.config.js**
```
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

Now, let's run the build again but instead using our new configuration file:

```
npx webpack --config webpack.config.js
```

If a webpack.config.js is present, the webpack command picks it up by default. We use the --config option here only to show that you can pass a config of any name. This will be useful for more complex configurations that need to be split into multiple files.

A configuration file allows far more flexibility than simple CLI usage. We can specify loader rules, plugins, resolve options and many other enhancements this way. See the configuration documentation to learn more.

## NPM Scripts

Given it's not particularly fun to run a local copy of webpack from the CLI, we can set up a little shortcut. Let's adjust our package.json by adding an npm script:

Now the npm run build command can be used in place of the npx command we used earlier. Note that within scripts we can reference locally installed npm packages by name the same way we did with npx. This convention is the standard in most npm-based projects because it allows all contributors to use the same set of common scripts (each with flags like --config if necessary).

Now run the following command and see if your script alias works:

```
npm run build
```

Custom parameters can be passed to webpack by adding two dashes between the npm run build command and your parameters, e.g. npm run build -- --colors.
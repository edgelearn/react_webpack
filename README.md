# ReactJS Webpack Tutorial

Documentation below has been pulled from https://webpack.js.org/guides/getting-started/

## Initial Setup

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
git commit -am "Initial Setup"

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

The _import_ and _export_ statements have been standardized in ES2015. Although they are not supported in most browsers yet, webpack does support them out of the box.

Behind the scenes, webpack actually "transpiles" the code so that older browsers can also run it. If you inspect _dist/main.js_, you might be able to see how webpack does this, it's quite ingenious! Besides _import_ and _export_, webpack supports various other module syntaxes as well, see [Module API](https://webpack.js.org/api/module-methods) for more information.

Note that webpack will not alter any code other than _import_ and _export_ statements. If you are using other ES2015 features, make sure to use a transpiler such as Babel or Bublé via webpack's loader system.

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

If a _webpack.config.js_ is present, the _webpack_ command picks it up by default. We use the _--config_ option here only to show that you can pass a config of any name. This will be useful for more complex configurations that need to be split into multiple files.

A configuration file allows far more flexibility than simple CLI usage. We can specify loader rules, plugins, resolve options and many other enhancements this way. See the [configuration documentation](https://webpack.js.org/configuration) to learn more.

You will want to commit your changes with the following command:

```
git commit -am "Using a Configuration"

```

## NPM Scripts

Given it's not particularly fun to run a local copy of webpack from the CLI, we can set up a little shortcut. Let's adjust our _package.json_ by adding an [npm script](https://docs.npmjs.com/misc/scripts):

```
    "scripts": {
-      "test": "echo \"Error: no test specified\" && exit 1"
+      "test": "echo \"Error: no test specified\" && exit 1",
+      "build": "webpack"
    },
```

Now the _npm run build_ command can be used in place of the npx command we used earlier. Note that within _scripts_ we can reference locally installed npm packages by name the same way we did with _npx_. This convention is the standard in most npm-based projects because it allows all contributors to use the same set of common scripts (each with flags like _--config_ if necessary).

Now run the following command and see if your script alias works:

```
npm run build
```

Custom parameters can be passed to webpack by adding two dashes between the _npm run build_ command and your parameters, e.g. _npm run build -- --colors_

You will want to commit your changes with the following command:

```
git commit -am "NPM Scripts"

```

## Loading CSS

In order to import a CSS file from within a JavaScript module, you need to install and add the style-loader and css-loader to your module configuration:

```
npm install --save-dev style-loader css-loader
```

**webpack.config.js**
```
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  }
};
```

webpack uses a regular expression to determine which files it should look for and serve to a specific loader. In this case any file that ends with .css will be served to the style-loader and the css-loader.

This enables you to import './style.css' into the file that depends on that styling. Now, when that module is run, a <style> tag with the stringified css will be inserted into the <head> of your html file.

Let's try it out by adding a new style.css file to our project and import it in our index.js:

**src/style.css**
```
.hello {
  color: red;
}
```

**src/index.js**
```
import _ from 'lodash';
import './style.css';

function component() {
  const element = document.createElement('div');

  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
  element.classList.add('hello');

  return element;
}

document.body.appendChild(component());
```

Now run your build command:

```
npm run build
```

Open up index.html in your browser again and you should see that Hello webpack is now styled in red. To see what webpack did, inspect the page (don't view the page source, as it won't show you the result, because the <style> tag is dynamically created by JavaScript) and look at the page's head tags. It should contain our style block that we imported in index.js.

Note that you can, and in most cases should, minimize css for better load times in production. On top of that, loaders exist for pretty much any flavor of CSS you can think of -- postcss, sass, and less to name a few.

You will want to commit your changes with the following command:

```
git commit -am "Loading CSS"

```

## Loading Images

So now we're pulling in our CSS, but what about our images like backgrounds and icons? Using the file-loader we can easily incorporate those in our system as well:

```
npm install --save-dev file-loader
```

**webpack.config.js**
```
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          'file-loader'
        ]
      }
    ]
  }
};
```

Now, when you import MyImage from './my-image.png', that image will be processed and added to your output directory and the MyImage variable will contain the final url of that image after processing. When using the css-loader, as shown above, a similar process will occur for url('./my-image.png') within your CSS. The loader will recognize this is a local file, and replace the './my-image.png' path with the final path to the image in your output directory. The html-loader handles <img src="./my-image.png" /> in the same manner.

Let's add an image to our project and see how this works, you can use any image you like into src/icon.png

**src/index.js**
```
import _ from 'lodash';
import './style.css';
import Icon from './icon.png';

function component() {
  const element = document.createElement('div');

  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
  element.classList.add('hello');

  // Add the image to our existing div.
  const myIcon = new Image();
  myIcon.src = Icon;

  element.appendChild(myIcon);

  return element;
}

document.body.appendChild(component());
```

**src/style.css**
```
.hello {
  color: red;
  background: url('./icon.png');
}
```

Let's create a new build and open up the index.html file again:

```
npm run build
```

If all went well, you should now see your icon as a repeating background, as well as an img element beside our Hello webpack text. If you inspect this element, you'll see that the actual filename has changed to something like 5c999da72346a995e7e2718865d019c8.png. This means webpack found our file in the src folder and processed it!


You will want to commit your changes with the following command:

```
git commit -am "Loading Images"

```


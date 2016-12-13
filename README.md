# es6-help
Helping a student with Webpack + babel + ES6

## Package.json
This is the heart and soul of your project. All npm packages will be added here.
The npm packages generate a directory called `node_modules` where they are stored.
This should not be pushed up to GitHub (GH) and can be "ignored" using a `.gitignore`. This can be generated for specific environments via GH - new repository or simply created in your text editor or via the Command Line Interface (CLI) with the following command: `touch .gitignore`.

For more in-depth reading:[NPM-Package.json](https://docs.npmjs.com/getting-started/using-a-package.json)

### Scripts:
NPM scripts run commands that execute code in your project. The two you will most likely see in projects and use yourself are `npm start` & `npm test`. These are specific to many third party technologies such as Heroku (by default looks for `npm start` to run your project) & Travis CI (by default looks for `npm test` to test your build).
You should avoid these two scripts being automated with packages like `nodemon` & `supervisor` and instead create separate scripts such as `npm run test:watch`. An important note is that `npm` does not require the `run` keyword for `npm start`, `npm test`, `npm stop`, & `npm restart`; however, it does require `run` for all other scripts, therefore `npm run test:watch`.

For more in-depth reading: [NPM-Scripts](https://docs.npmjs.com/misc/scripts)

Example:

```
"scripts": {
  "test": "mocha --compilers js:babel-register test/**/*.js*",
  "test:watch": "nodemon --exec '_mocha --reporter min --compilers js:babel-register test/**/*.js*'",
  "postinstall": "npm run build",
  "start": "babel-node server/index.js",
  "start:dev": "npm run build && npm run dev",
  "dev": "webpack-dev-server --inline --hot --content-base build --history-api-fallback",
  "local": "heroku local",
  "mkdir": "mkdir -p build",
  "build": "npm run clean && npm run mkdir && npm run build:html && npm run build:js",
  "clean": "rm -rf build",
  "build:html": "npm run clean:html && cp src/public/index.html build/",
  "clean:html": "rm -f build/index.html",
  "build:js": "npm run clean:js && webpack",
  "clean:js": "rm -f build/$npm_package_name.$npm_package_version.js build/$npm_package_name.$npm_package_version.js.map"
}
```

### Dependencies vs Dev-Dependencies
Basically what you can ask yourself is "Does my project need this to run when `npm start` is ran?" If the answer is yes then it should be placed in the DEPENDENCIES object. If not then it's a DEV-DEPENDENCY. I say this because when you go to host an application on sites like Heroku it will by default ONLY load your DEPENDENCIES so if you have a package that is critical to the application running in your DEV-DEPENDENCIES the project will break.

The following example is from a `package.json` that hosts a project built with webpack on Heroku. Notice nearly all babel packages and webpack are in the DEPENDENCIES of the project because they will be needed to compile ES6 syntax to ES5 through webpack.

```
"dependencies": {
  "babel-cli": "^6.18.0",
  "babel-core": "^6.18.2",
  "babel-loader": "^6.2.9",
  "babel-preset-es2015": "^6.18.0",
  "babel-preset-react": "^6.16.0",
  "babel-preset-stage-0": "^6.16.0",
  "babel-register": "^6.18.0",
  "bcrypt": "^1.0.0",
  "body-parser": "^1.15.2",
  "chai": "^3.5.0",
  "enzyme": "^2.6.0",
  "express": "^4.14.0",
  "mongoose": "^4.7.1",
  "morgan": "^1.7.0",
  "passport": "^0.3.2",
  "passport-local": "^1.0.0",
  "passport-spotify": "^0.3.0",
  "react": "^15.4.1",
  "react-dom": "^15.4.1",
  "validator": "^6.2.0",
  "webpack": "^1.14.0"
},
"devDependencies": {
  "babel-eslint": "^7.1.1",
  "chai": "^3.5.0",
  "eslint": "^3.11.1",
  "eslint-config-rallycoding": "^3.1.0",
  "mocha": "^3.2.0",
  "react-addons-test-utils": "^15.4.1",
  "webpack-dev-server": "^1.16.2"
}
```

## Webpack
For more in-depth reading:[Webpack](https://webpack.js.org)
NOTE: Webpack 2 is comming soon so this information will not be valid long.
For more reading on the `webpack` package: [Webpack Package](https://www.npmjs.com/package/webpack)

All though you can bundle your server you will likely only ever bundle your client side code.

I will base this break down of the `webpack.config.js` on the example from Thinkful:

The main purpose is to bundle JavaScript (JS) files for usage in a browser, yet it is also capable of transforming, bundling, or packaging just about any resource or asset.

Obviously the first thing you will need to do is create a file in the root directory of your project normally these files are named `webpack.config.js`. You will likely see projects on GH that have `production`, `development`, etc versions of this file I will let you take the plunge on creating multiple configs for different environments.

So first steps:
`touch webpack.config.js`
`npm install --save-dev webpack`
Next go into your package.json and create the following scripts (for simplicity this example is not taking into account minification or any css pre/post processors are present in the project):
```
"mkdir": "mkdir -p build",
"build": "npm run clean && npm run mkdir && npm run build:html && npm run build:js",
"clean": "rm -rf build",
"build:html": "npm run clean:html && cp src/public/index.html build/",
"clean:html": "rm -f build/index.html",
"build:js": "npm run clean:js && webpack",
"clean:js": "rm -f build/$npm_package_name.$npm_package_version.js build/$npm_package_name.$npm_package_version.js.map"
```


```
var path = require('path');

var webpack = require('webpack');

var packageData = require('./package.json');

var filename = [packageData.name, packageData.version, 'js'];

module.exports = {
    entry: path.resolve(__dirname, packageData.main),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: filename.join('.'),
    },
    devtool: 'source-map',
    module: {
      loaders: [
        {
          test: /\.js$/,
          exclude: /(node_modules)/,
          loader: 'babel-loader',
          query: {
            presets: ['es2015']
          }
        }
      ]
    }
}
```

With the setup from Thinkful webpack is going to look at your package.json to name the bundle. Whatever your `"name":` key has will be what the bundle is called plus the version number of your project in the package.json `"version":`. So in the case of this repository's bundle what will be yielded as `es6-help.1.0.0.js`.

Now for the important part! `"main":` is the entry point for your build to start. Whatever executes your client side code should be here. In most cases this is a file called `client.js` or perhaps `client/index.js`.

### Webpack + Babel
For more on Babel: For more in-depth reading:
[Babel](https://babeljs.io)
[Babel-GH](https://github.com/babel/babel)

So first off Babel is a transpiler for changing ES6 syntax to ES5. This is done because the browsers are not all running exclusively on ES6 yet so send it ES6 code and Chrome or any other browser will not render what you want at all because it has no idea what you sent it!

Therefore if you are using ES6 you need Babel.
But you are using webpack?
How do you bundle and transpile and other big fancy 50 cent words???

Enter the loaders!

Above in the example there is a module with a key called `loaders`. You can have multiple loaders but for sake of simplicity we will assume you only need one. The loaders are going to look at the code as it is being bundled transpile it to ES5 (I am probably very under discussing this point! Go read docs for more info!!!).
This loader is specifically only looking for files ending in `.js`. You can set this for css files, jsx, you name it! (again read docs probably not everything).
Our loader is `babel` so at this point the babel-loader package will fire up and do it's thing. The last part `query` is telling the loader what to look for as presets. There are many many presets. If you add any to your project they need to be added to this section as well as your `.bablerc` or `"babel":` object in the package.json.

In the example we are using `es2015` or our code is written in ES6 so query for ES6 syntax. Later in the Thinkful React Course you will use `react` & `stage-0`, but as I said there are many more.


## ES6
You want to know the sweetness of ES6?
Check out the following!
[ES6-Official](http://es6-features.org/#Constants)
[ES6-GH](http://es6-features.org/#Constants)
[thenewboston-ES6-Basics](https://www.youtube.com/watch?v=ZJZfIw3P8No&list=PL6gx4Cwl9DGBhgcpA8eTYYWg7im72LgLt)
[Stephen Grider ES6 Course on Udemy](https://www.udemy.com/javascript-es6-tutorial/)


## Conclusion

I just put this together in like 45 minutes so if there are spelling mistakes forgive me and I'm positive there is over simplification that is why links to these technologies are present. Don't take my word for anything and definitely do not believe this is the word of God on anything. Tech changes all the time and we must strive to keep up with it! That being said feel free to PR this repo and open issues!

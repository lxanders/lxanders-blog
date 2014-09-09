{
  title: 'Creating a base for developing a web application with node',
  date: '2014-09-06',
  author: 'Alexander Schmidt'
}

In the following steps I will guide you through the creation of a technical basis for a dynamic web application. The core technologies used are:

- node
- express for the server side
- handlebars as templating engine
- bootstrap for some eyecandy
- grunt as the build tool

This guide is deliberately verbose in some steps: There are some steps that render steps executed before less useful. The reason for this is to let the reader understand what is happening and why. If you just want to get the complete package just clone the example project from my [github repository](https://github.com/lxanders/node-express-base).

Create your local git repository
--------------------------------

As using a version control system is always a good idea you should use one. Git is a distributed version control system and has all the features you want. 

Hosting your repository in the internet today is easy. My best choice is [github](https://github.com/). Github offers a reliable and easy to use hosting and management for git repositories. It is free if you are willing to make your sources public. However if you don't want your code to be visible to the world you can either pay for private repositories or use something like [bitbucket](https://bitbucket.org/). Both offer support for pull requests and repository management.

- Just change to your project directory and enter `git init`
- Add a `.gitignore` file (otherwise files will be tracked that you don't want to commit). Just add a new line with a file name or a folder name here to ignore something else. 

Example content:

```
node_modules
```

Create your repository at the github or bitbucket page and follow the instructions how to initialize it. For github these are next steps (just modify the following to match your data and run it):

```
git remote add origin git@github.com:yourGithubUsername/yourInterestingRepository
git push -u origin master
```

Create your node package json file
----------------------------------

The following `package.json` example contains some sane defaults:

```
{
    "name": "yourInterestingProjectNameHere",
    "version": "0.0.1",
    "description": "An even more interesting short description for your project.",
    "main": "index.js",
    "scripts": {
        "test": "eslint . && mocha test/unit/ --recursive"
    },
    "repository": {
        "type": "git",
        "url": "https://github.com/yourUsername/yourRepositoryName"
    },
    "dependencies": {
        "express": "4.4.4"
    },
    "devDependencies": {
        "eslint": "0.6.2",
        "mocha": "1.20.1",
        "chai": "1.9.1"
    },
    "engines": {
        "node": ">=0.10"
    },
    "engineStrict": true
}

```

Running `npm install` will install your configured dependencies. In the configuration file above this would be 
- [express](https://github.com/visionmedia/express) as your server side web development framework
- [mocha](https://github.com/visionmedia/mocha) for running your tests
- [chai](https://github.com/chaijs/chai) for your test assertions
- [eslint](https://github.com/eslint/eslint) for linting your files

If you didn't know already: Dependencies listed in `devDependencies` are not part of the application you are developing but are helpers for the development process so they are not shipped with your application when you publish a package.

Improve your code qualitiy: use a linting tool
----------------------------------------------

The example `package.json` file from above already contains the development dependency for this.

- Add the `.eslintrc` files listed below. They are used for configuring your project rules and globals.  The examples already contains some defaults that will help a lot keeping your code clean
  - Activating the `node` environment disables warnings and errors regarding node specific globals
  - Setting the `mocha` environment will add globals for the keywords like `describe`, `it` etc.
  - You can have multiple `.eslintrc` files. These files will be used recursive for all the subdirectories. If you have special folders or rules that should only be applied on specific folders you can just create another one in this directory and it will overwrite the settings of the more general one. So we are adding one for our project base and another one for our tests folder as we have more specific settings for this one
  - For more documentation how the rules used work just visit the good [eslint documentation on rules](http://eslint.org/docs/rules/)


`.eslintrc`
```
{
    "env": {
        "node": true
    },
    "rules": {
        "quotes": [2, "single"],
        "no-warning-comments": [2, { "terms": ["todo", "fixme", "wtf"], "location": "anywhere" }],
        "strict": 2,
        "no-process-exit": 2,
        "eqeqeq": 2,
        "no-delete-var": 2,
        "one-var": 2,
        "max-params": [2, 4],
        "no-plusplus": 2,
        "semi": [2, "always"],
        "no-mixed-requires": 0,
        "valid-jsdoc": 0
    }
}
```

Just create the test folder (`mkdir test`) and a file with the following contents:

`test/.eslintrc`
```
{
    "env": {
        "node": true,
        "mocha": true
    },
    "rules": {
        "no-unused-expressions": 0
    }
}
```

- Add a `.eslintignore` file (otherwise files are being checked that aren't part of your sources - e.g. the node modules). The file follows very similar syntax rules as the `.gitignore` file - every line stands for a file or directory name that is to be ignored. Don't forget the only difference: The stars indicate that all subcontent is to be ignored too (this is the difference to `.gitignore` files).

Example:

```
node_modules/**
```

Overview: The project structure
-------------------------------

This is the project file structure I'm aiming for - if you followed the steps above, some of the files and directories are already present, while others are missing:

```
yourInterestingProjectNameHere
    .eslintignore
    .eslintrc
    .git
    .gitignore
    client
        css
    index.js
    node_modules
    package.json
    README.md
    server
        index.js
    templates
    test
        unit
```

- Create the `test/unit` folder - all your unit tests will reside here: `mkdir test/unit`
- Create the `client` folder - it will contain the code that is shipped as the frontend
- Create the `client/css` folder - it will contain the style sheets that are used in the frontend
- Create the `templates` folder - this is where your templates will reside
- Create the `server` folder - it holds the server side code
- Create your `index.js` file - it is the starting point for your express server
- Create the server's `index.js` file - it will be used for starting the actual express server

Note that git will not mark empty folders as a modification of your repository. So just add the folders when you add files to them.

The contents of the files are described in the next section.

Create your applications startup files
--------------------------------------

Feel free to use these basic versions of the index files.
 
The `./index.js` file will require the more concrete server file and start the server:

```
'use strict';

var app = require('./server/index');

app.listen(3000);
console.log('Server started on port 3000');
```

The `./server/index.js` file will initialize the express server and contain the endpoint definitions etc. later:

```
'use strict';

var path = require('path'),
    express = require('express'),
    app = express();

module.exports = app;
```

The separation of files might look like overkill for now but works really well for me and more complex scenarios.

Start your web application for the first time by running `node index.js`. This should result in something like this:

```
$ node index.js
Server started on port 3000
```

Templating
----------

To create a dynamic web application we need something to render templates. For me [handlebars](http://handlebarsjs.com/) worked very well. It is lightweight and does exactly what a templating engine should - it renders templates and gives us the ability to render dynamic content while not bringing too much other functionallity. A template shouldn't contain business logic and source code itself so we doesn't need the abilities for that in a templating engine.

To use handlebars quite comfortably we're using a wrapper that works nicely with express. For this I recommend [hbs](https://github.com/donpark/hbs).

- Enable the view engine in your express app and register an example endpoint. Add the following to your `server/index.js` file:

```
app.set('view engine', 'hbs');
app.set('views', path.join(process.cwd(), 'templates'));

app.get('/', function (req, res) {
    res.render('index');
});
```

Note about the `views` directive: I don't like the default folder `views` that is used by express for templates. So I changed it here.

- Create a layout file for your page: `templates/layout.hbs`

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>

<h1>This is dummy content in the Layout-File.</h1>

{% raw %}{{{body}}}{% endraw %}

</body>
</html>
```

You actually have to use `layout.hbs` as file name. Rendering any template will then have the view engine automatically use the layout file as basisi and replace the `{% raw %}{{{ body }}}{% endraw %}` part with the content of the template to render.

- Create a base template for your web application's root: `templates/index.hbs` and just put some text in it - the content here is put in the rendered result when `res.render('index')` is called by visiting your web app's root in a browser 

```
<p>And this is dummy content in the Index-File.</p>
```

Improve the process of starting your server
-------------------------------------------

To make it even easier to remember (especially when working with others on a project this comes in handy as the developers don't have to know which is the main file) you can add a start script in your `package.json`. Just add something like this to the `scripts` section (if you followed this guide, you already have a test script - this is exactly where you want to put your start script):

```
"scripts": {
    "test": "eslint . && mocha test/unit/ --recursive",
    "start": "node index.js"
},
```

After adding this, you can just run `npm start` and your server is started.

Automatic restarting of the server
----------------------------------

During the development it is a good idea to have your server automatically restart when your code changes.

For this I'm using [nodemon](https://github.com/remy/nodemon). I think installing packages globally can be a problem: Think of a complex project with dozens of used modules. A new developer will be very happy if he doesn't have to find out all the global dependencies by himself and can rely on a simple `npm install` instead that takes care of most of the work. So I recommend adding `nodemon` to your `devDependencies`. Just add the following: `"nodemon": "1.2.0"` and run `npm install`.

The approach with the start script from above helps us now as you can just change the start script - the command to start the server itself didn't change. Modify the start script to run the locally installed `nodemon`: 

```
"start": "./node_modules/.bin/nodemon index.js"
```

All files are watched for changes now and the server is restarted automatically.

Normally you don't need all files to be watched - this can even cause performance issues. Just add a `nodemon.json` in your project's root folder:

```
{
    "verbose": true,
    "ignore": [
        "test",
        "node_modules",
        "client"
    ]
}
```

Add some styling
----------------

To register static files in your `express` app add the following to your `server/index.js` file:

```
app.use('/client', express.static(path.join(__dirname, '../client')));
```

To get some basic styling just add twitter [bootstrap](http://getbootstrap.com/). Get the compiled and minified css file, extract it and put the `bootstrap.css` and `bootstrap-theme.css` files to your `client/css` folder.

After that add the following line to the head section of your `templates/layout.hbs` file:

```
<link href="client/css/bootstrap.css" rel="stylesheet">
```

Improve your build process
--------------------------

Especially on bigger projects you most likely will have at least some more complex build steps. This can be done quite easily using `grunt`. An command line interface is available which runs the `grunt` version used in your project. This is one of the tools I even recommend installing globally, so just run:

```
npm install -g grunt-cli
```

Now let’s add the dependencies to our local project to the `package.json` file:

```
"devDependencies": {
    [...]
    "grunt": "0.4.5",
    "grunt-contrib-clean": "0.5.0",
    "grunt-nodemon": "0.2.1"
}
```

And change the scripts to run the specified `grunt` task at startup (you still start your server by entering `npm start`):

```
"scripts": {      
    "test": "eslint . && mocha test/unit/ --recursive",
    "start": "grunt server"
},
```

The configuration for `grunt` is done in the `Gruntfile.js` file. Create it and use the following content:

```
'use strict';

module.exports = function (grunt) {

    grunt.initConfig({

        clean: {
            dependencies: ['node_modules']
        },

        nodemon: {
            development: {
                script: 'index.js',
                ignore: [
                    'test/**',
                    'node_modules/**',
                    'client/**'
                ]
            }
        }

    });

    grunt.loadNpmTasks('grunt-contrib-clean');
    grunt.loadNpmTasks('grunt-nodemon');

    grunt.registerTask('compile', ['clean:build']);
    grunt.registerTask('server', ['nodemon']);
    grunt.registerTask('default', ['server']);
};
```

There are two task definitions for now in this short example:

- `compile` task: Running `grunt compile` runs a cleanup that is configured in the `initConfig` part of the configuration. Normally you would add other steps for the compile too but this goes too far for this guide. The cleaning step will delete the configured dependencies of your project. So the `node_modules` is deleted whenever you run it 
- The `server` task runs `nodemon`. The target is configured to use the `index.js` file on the same level than the `Gruntdile.js`. Additionally we are configuring `nodemon` to ignore some files. Before using `grunt` we did this in the `nodemon.json` file. This file can now be safely deleted
- Note that you can run `grunt clean` from the console. This task is implicitly available through the `grunt-contrib-clean` package and uses the targets specified in the `initConfig` section - in this configuration only the `dependencies` target is found and they will be cleaned

Conclusion
----------

If you followed the steps you should have a nice basis for a web application. You can find the complete example code in my [github repository for this example project](https://github.com/lxanders/node-express-base). I hope this was enough help to get you started. I might add some further howtos later so check back from time to time, if you liked this guide.


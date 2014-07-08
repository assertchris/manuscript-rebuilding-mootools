# Grunt

Using CoffeeScript means we need some post-processing. We'll also want to compress the JavaScript files incase people prefer the smaller file-size. To do both of these, we'll use Grunt!

## Installing Grunt

Grunt is a JavaScript task runner. You can find it at [http://gruntjs.com](http://gruntjs.com).

The first step is installing the command-line interface (referred to as the CLI, hereafter). You should have NPM installed already!

A> Installing Node and/or NPM are a bit outside of the scope of this series. Find out how to do both at [http://nodejs.org](http://nodejs.org)

The CLI is something you'll want to install globally. It's not project-specific, so there's no harm in re-using it across multiple projects. Install it with:

```
$ npm install -g grunt
  
grunt@0.4.5 /usr/local/lib/node_modules/grunt
├── dateformat@1.0.2-1.2.3
├── which@1.0.5
├── eventemitter2@0.4.13
```

Next, we need to pull in a few post-processor dependencies:

```
{
  "name": "library",
  "version": "0.0.1",
  "devDependencies": {
    "buster": "0.*",
    "grunt": "0.*",
    "grunt-contrib-coffee": "0.*",
    "grunt-contrib-uglify": "0.*",
    "grunt-contrib-watch": "0.*"
  }
}
```

A> This file should be saved as `package.json`

`package.json` is a file which NPM uses to determine which dependencies should be downloaded. It benefits from the addition of a few fields ([https://www.npmjs.org/doc/json.html](https://www.npmjs.org/doc/json.html)), and we've also added `devDependencies`. They're all for the CoffeeScript-to-JavaScript conversion process we've been speaking about.

The following command will install these dependencies:

```
$ npm install
  
npm WARN package.json library@0.0.1 No description
npm WARN package.json library@0.0.1 No repository field.
npm WARN package.json library@0.0.1 No README data
```

This will download the dependencies to a directory called `node_modules`. You'll want to add this directory to `.gitignore` so the dependencies aren't tracked.

## Using Grunt

To use Grunt effectively, we need to create a Gruntfile. This is similar to a Makefile et. al:

```
module.exports = function (grunt) {
  
  grunt.initConfig({
    "pkg": grunt.file.readJSON("package.json"),
    "coffee": {
      "default": {
        "options": {
          "bare": true,
          "join": true
        },
        "files": [
          {
            "expand": true,
            "flatten": false,
            "cwd": "source",
            "src": ["**/*.coffee"],
            "dest": "build",
            "ext": ".js"
          }
        ]
      }
    },
    "watch": {
      "default": {
        "tasks": [
          "coffee",
          "uglify"
        ],
        "files": [
          "source/**/*"
        ]
      }
    },
    "uglify": {
      "default": {
        "options": {
          "compress": true,
          "report": false
        },
        "files": [
          {
            "expand": true,
            "flatten": false,
            "cwd": "build",
            "src": ["**/*.js", "!**/*.min.js"],
            "dest": "build",
            "ext": ".min.js"
          }
        ]
      }
    }
  });
  
  grunt.loadNpmTasks("grunt-contrib-coffee");
  grunt.loadNpmTasks("grunt-contrib-watch");
  grunt.loadNpmTasks("grunt-contrib-uglify");
  
  grunt.registerTask("default", [
    "coffee",
    "uglify"
  ]);
  
};
```

A> This file should be saved as `gruntfile.js`

We'll not go too deep into how Grunt works, nor the intricacies of the Grunt plugins we're using here. Those are a subject for another series. Instead (after reading the example) accept that it's doing the following:

1. A `coffee` command/plugin is configured to convert all `.coffee` files it finds in the `source` directory into JavaScript files (of the same name/path) in the `build` directory.

2. A `watch` command/plugin is configured to scan the `source` directory for any changes, and to run the `coffee` and `uglify` commands when they happen.

3. A `uglify` command/plugin is configured to convert all `.js` files it finds in the `build` directory into minified JavaScript files (ending in `.min.js`) in the same places it finds them.

A> The `uglify` command is configured to ignore files ending in `.min.js` when it minifies `.js` files, in the `build` directory. If not for this exclusion, the `.min.js` files would grow exponentially with each `grunt` cycle.

The three plugins are loaded and a `default` command is registered (to run the `coffee` and `uglify` commands in sequence).

This means you can run any of the following commands:

1. `grunt` - this will run the `default` command.
2. `grunt watch` - this will start the file-watcher.
3. `grunt coffee` - this will convert the `.coffee` files to `.js`.
4. `grunt uglify` - this will convert `.js` files to `.min.js`.

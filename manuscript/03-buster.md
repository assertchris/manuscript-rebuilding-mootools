# Buster

Part of writing good software is testing. Whether you write your tests before or after the code they cover, having tests is essential!

## Installing Buster

Buster is a testing toolkit which we'll use to write unit tests. Like Grunt, it's best installed globally, with:

```
$ npm install -g buster
  
> contextify@0.1.8 install /usr/local/lib/node_modules/buster/node_modules/buster-syntax/node_modules/jsdom/node_modules/contextify
> node-gyp rebuild
```

This gives us access to a command called `buster-test`. Before it can tell us if our library works; it needs some configuration:

```
module.exports = {
  "tests": {
    "environment": "node",
    "tests": [
      "tests/**/*"
    ]
  }
};
```

A> This file should be saved as `buster.js`

## Using Buster

This configuration tells Buster that we want to test all files in a `tests` directory, and to load them as node modules. A simple test looks like:

```
var buster = require("buster");
var assert = buster.referee.assert;
  
var subject = require("path/to/test/subject");
  
buster.testCase("subject", {
  "has a dummy test": function () {
    assert.isTrue(
      true
    );
  }
});
```

A> This file should be saved within the `tests` directory

In order to run this test, you need to use a command resembling the following:

```
$ buster-test --config=buster.js
  
1 tests, 1 assertion, 1 runtime ... OK
```


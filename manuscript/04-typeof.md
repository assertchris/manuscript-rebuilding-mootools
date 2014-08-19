# typeOf 

Dynamically-typed programming languages, of which JavaScript is one, present an interesting problem. The functions and methods designed to work with variables either need to accept many different types for a single parameter; operating accordingly to for each type, or they need a way of asserting that the parameter is of the correct type to begin with.

## Enter typeOf… 

MooTools has a few methods to do this kind of type checking. The one we’ll re-create, in this post, is the `typeOf()` function. JavaScript has a native `typeof` keyword which, when combined with a variable, identifies the basic type of the variable.

Stated differently, if you use the `typeof` keyword and then a variable, you get a string to compare it to:

{lang=js}
```
typeof "foo" === "string";          // true
typeof 1 === "number";              // true
typeof function(){} === "function"; // true
```

This is helpful, but only to a point. The number of types this construct can identify are six (more if you count ECMAScript 6 and Host objects). Some of the comparisons are also misleading. For instance: `null` is identified as object and regular expressions are identified as `object` or `function`, depending on implementation.

There is also no way to augment the `typeof` construct to allow the definition of new types. 

And so MooTools defined the `typeOf()` function to overcome these limitations, and provide a consistent means of identifying types:

{lang=js}
```
var typeOf = this.typeOf = function(item){
  if (item == null) return 'null';
  if (item.$family != null) return item.$family();
  
  if (item.nodeName){
    if (item.nodeType == 1)
      return 'element';
    if (item.nodeType == 3) return (/\S/).test(item.nodeValue)
      ? 'textnode' : 'whitespace';
  } else if (typeof item.length == 'number'){
    if ('callee' in item) return 'arguments';
    if ('item' in item) return 'collection';
  }
  
  return typeof item;
};
```

A> This is from `Source/Core/Core.js`

This function addresses a few of the shortcomings of the native construct, but still suffers from a few problems of its own. 

### Elements 

MooTools was born in the age of DHTML. Classes may have been the most compelling tool provided, but element manipulation was also a significant contributor to the appeal of the library. Even at this fundamental level; elements are part of the make-up. 

MooTools wasn’t built to address the issues of DOM manipulation, yet they are a big part of the codebase. The goal (at least as I have come to understand it) was to create a rich toolset for serious application development, through re-usability and clarity. 

A modern approach would be to separate the DOM logic out of such a fundamental aspect of the library, and add it in the form of an extension.

### Dates and Regular Expressions 

These types are fundamental to JavaScript development, yet the `typeOf()` function still returns them as `object` or `function`, depending on implementation. These should be as easy to identify as strings and numbers.

## A Different Take 

I thought about this, for a long time, and came up with an alternative which I think addresses all of the problems highlighted:

```
"use strict"
 
do ->
  typeOf = (item) ->
    if item == undefined
      return "undefined"
  
    if item == null
      return "null"
  
    if typeof item.getType == "function"
      return item.getType()
  
    types = {
      "[object Boolean]"   : "boolean",
      "[object Number]"    : "number",
      "[object String]"    : "string",
      "[object Function]"  : "function",
      "[object Array]"     : "array",
      "[object Date]"      : "date",
      "[object RegExp]"    : "regexp",
      "[object Object]"    : "object",
      "[object Arguments]" : "arguments"
    }
  
    return types[Object.prototype.toString.call(item)]
  
  module.exports = typeOf
```

A> This is from `source/core/typeOf.coffee`

In order, this function will take the following steps to determine the applicable type of a variable:

1. If the variable is either `undefined` (in the case of `typeOf()`) or `null`, then those will be returned as strings.
2. If the variable has a method, called `getType` then the result of it will be returned.
3. If the string representation of a variable matches any of the pre-defined options, the applicable type string will be returned.

This means variables (of any of the eleven defined types) will be identified, and that a custom identification function can also be added.

A> The DOM aspects of this function have been removed, and I’ll only talk about them much later on.

## Tests

Fortunately, this function is fairly easy to test. We just need to make sure that new instances of each type are compared against the expected type:

{lang=js}
```
var buster = require("buster");
var assert = buster.referee.assert;
 
var typeOf = require("../../build/core/typeOf.js");
 
buster.testCase("build/core/typeOf", {
  "identifies types with getType() function" : function () {
    var dummy = {
      "getType" : function () {
        return "dummy type";
      }
    };
  
    assert.equals(
      "dummy type",
      typeOf(dummy)
    );
  },
  
  "identifies 'basic' types" : function () {
    var equalities = [
      [
        "undefined",
        typeOf()
      ],
      [
        "null",
        typeOf(null)
      ],
      [
        "boolean",
        typeOf(true)
      ],
      [
        "number",
        typeOf(5)
      ],
      [
        "string",
        typeOf("foo")
      ],
      [
        "function",
        typeOf(new Function())
      ],
      [
        "array",
        typeOf([1, 2, 3])
      ],
      [
        "date",
        typeOf(new Date())
      ],
      [
        "regexp",
        typeOf(/[a-z]/)
      ],
      [
        "object",
        typeOf({"hello" : "world"})
      ],
      [
        "arguments",
        (function () {
          return typeOf(arguments);
        }())
      ]
    ];
  
    for (var i = 0; i < equalities.length; i++) {
      assert.equals(
        equalities[i][0],
        equalities[i][1]
      );
    }
  }
});
```

A> This is from `tests/core/typeOf.js`
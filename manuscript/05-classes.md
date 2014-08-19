# Classes 

One of the most famous aspects of MooTools was the idea of classes. JavaScript doesn't (yet) have support for creating classes, so the syntactic sugar MooTools added made it possible to simulate this language construct in a way traditional OOP developers were familiar with.

## CoffeeScript, Duh!

CoffeeScript does have "native" support for classes, in a similar way that MooTools had. That is; you can use a class keyword, and CoffeeScript will generate the approximation of a class in native JavaScript (while not actually using native JavaScript classes).

There are still some deficiencies with the CoffeeScript implementation, as far as the MooTools feature-set is concerned. For a start, class extension is supported but mixing implementation is not. Furthermore, the `Class.Extras` (a set of utility mixins extending the functionality of the base class system, in MooTools) still need to be implemented to sufficiently replicate the MooTools feature-set.

## Implements

The **[CoffeeScript Cookbook](http://coffeescriptcookbook.com)** demonstrates a **[possible implementation of mixing support](http://coffeescriptcookbook.com/chapters/classes_and_objects/mixins)**, but it's not at all similar to how MooTools allowed mixins to be defined:

```
var Animal = new Class({
  initialize: function(age){
    this.age = age;
  }
});
  
var Cat = new Class({
  Implements: Animal,
  setName: function(name){
    this.name = name
  }
});
```

A> This is from **[http://mootools.net/docs/core/Class/Class](http://mootools.net/docs/core/Class/Class)**

In order for a CoffeeScript class to maintain the ability to call methods on `super`, the extend keyword needs to stay. We can add an `implements`-like method though. After a bit of research, I stumbled across a nice guide to implementing this sort of thing: **[http://elving.me/post/47457631121/mixins-in-coffeescript](http://elving.me/post/47457631121/mixins-in-coffeescript)**.

The general idea is that we can great a static method which copies methods and properties over to the class we're defining:

```
"use strict"
  
do ->
  
  class Implements
    @implement = (mixins...) ->
      for mixin in mixins
        for own key, value of mixin
          @::[key] = value 
```

A> This is from `source/core/class.coffee`

To have this functionality, base a class must extend `Implements` at some point. It's like a base class, and if you prefer you can even name it that! It can be used similarly to:

```
class Animal
  eat: ->
    console.log("om nom nom")
      
class Cat extends Implements
  @implement Animal
```

We can remove this restriction by altering the implementation a bit:

```
implement = (base, mixins...) ->
  for mixin in mixins
    for key, value of mixin::
      base::[key] = value
  
class Implements
  @implements = (mixins...) ->
    implement.apply(null, [this].concat(mixins))
```

A> This is from `source/core/class.coffee`

This new structure allows a cleaner method for applying mixins:

```
class Animal
  eat: ->
    console.log("om nom nom")
  
class Cat # extends Implements
  # @implements Animal
  
  constructor: ->
    console.log("new cat")
  
implement(Cat, Animal)
  
cat = new Cat();
cat.eat();
```

You can use either format, as they'll do the same thing. For the new format though, you don't need to extend `Implements`, so there are fewer places for errors to creep in.

## Testing

The previous tests we wrote didn't deal with how CoffeeScript generates code, or how structures converted from CoffeeScript syntax would work in a Javascript environment. Testing this mixin functionality requires a bit of modification to our Buster setup.

A> I was overjoyed to find a buster-coffee plugin which would allow me to use CoffeeScript in test files. I spent hours trying to get it to work. In the end I tried a completely different approach.

Tests can be written in CoffeeScript and executed by calling `coffee test.coffee`. We'll still use Buster to run the actual tests, but we'll no longer need the global Buster CLI command. We can install Buster locally:

```
❯ npm install --save-dev buster
  
npm WARN package.json library@0.0.1 No description
npm WARN package.json library@0.0.1 No repository field.
npm WARN package.json library@0.0.1 No README data
...
```

Then we can remove the old `buster.js` file and add an index file:

```
require("./core/class.coffee")
require("./core/typeOf.coffee")
```

A> This is from `tests/index.coffee`

Then we can convert the typeOf tests to CoffeeScript, and add new class tests:

```
buster = require("buster")
assert = buster.referee.assert
  
typeOf = require("../../build/core/typeOf.js").typeOf
  
buster.testCase("build/core/typeOf", {
  "identifies types with getType() function": ->
    dummy = {
      "getType": ->
        return "dummy type"
    };
  
    assert.equals(
      "dummy type",
      typeOf(dummy)
    )
  
  "identifies 'basic' types" : ->
    equalities = [
      ["undefined", typeOf()],
      ["null", typeOf(null)],
      ["boolean", typeOf(true)],
      ["number", typeOf(5)],
      ["string", typeOf("foo")],
      ["function", typeOf(->)],
      ["array", typeOf([1, 2, 3])],
      ["date", typeOf(new Date())],
      ["regexp", typeOf(/[a-z]/)],
      ["object", typeOf({"hello" : "world"})],
      ["arguments", do -> return typeOf(arguments)]
    ];
  
    for equality in equalities
      assert.equals(
        equality[0],
        equality[1]
      )
})
```

A> This is from `tests/core/typeOf.coffee`

```
buster = require("buster")
assert = buster.referee.assert
  
implement  = require("../../build/core/class.js").implement
Implements = require("../../build/core/class.js").Implements
  
buster.testCase("build/core/class", {
  "normal extends works (baseline)": ->
    class Foo
      hello: "hello"
      sayHello: ->
        return this.hello
  
    class Bar extends Foo
      sayWorld: ->
        return "world"
  
    bar = new Bar()
  
    assert.equals(
      "hello",
      bar.sayHello()
    )
  
    assert.equals(
      "world",
      bar.sayWorld()
    )
  
  "Implements works": ->
    class Foo
      hello: "hello"
      sayHello: ->
        return this.hello
  
    class Bar extends Implements
      @implements Foo
  
      sayWorld: ->
        return "world"
  
    bar = new Bar()
  
    assert.equals(
      "hello",
      bar.sayHello()
    )
  
    assert.equals(
      "world",
      bar.sayWorld()
    )
  
  "implement works": ->
    class Foo
      hello: "hello"
      sayHello: ->
        return this.hello
  
    class Bar
      sayWorld: ->
        return "world"
  
    implement(Bar, Foo)
  
    bar = new Bar()
  
    assert.equals(
      "hello",
      bar.sayHello()
    )
  
    assert.equals(
      "world",
      bar.sayWorld()
    )
})
```

A> This is from `tests/class.coffee`

We can now run these test with:

```
❯ coffee tests
  
5 tests, 18 assertions, 1 runtime ... OK
```

A> You'll notice I modified `source/core/typeOf.coffee` to return an object instead of the typeOf function. This will make it more consistent with the modules we make in the future.
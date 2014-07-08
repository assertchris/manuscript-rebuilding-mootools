# CoffeeScript

So I'm starting a series on rebuilding one of the oldest JavaScript frameworks still in use by proposing it be rewritten in CoffeeScript? You bet!

CoffeeScript is a significant-whitespace, compiles-to-JavaScript language. I've used it for a long time, and it appeals to the Python developer in me.

I've selected CoffeeScript for a few other reasons:

## Neat Compilation

CoffeeScript compiles to rather nice JavaScript. Let's look at an example:

```
class Logger
  
  constructor: (method) ->
    this.method = method
  
  log: (messages...) ->
    this.method.apply(null, messages)
```

This is a simple class, which initialises with a logging method, and invokes it with a list of messages. You could instantiate it as simply as:

```
logger = new Logger(console.log)
```

Both of these examples are written in CoffeeScript, yet they are easy for a JavaScript developer to understand. If you've seen the `Function.prototype.apply` method, you'll know it accepts an array as the second parameter.

This leads you to understand that `messages...` is an array; and it can therefore be assumed that variables ending in `...` are variadic. The entire language can be discovered in this way.

But how does this translate into JavaScript?

```
var Logger,
  __slice = [].slice;
  
Logger = (function() {
  function Logger(method) {
    this.method = method;
  }
  
  Logger.prototype.log = function() {
    var messages;
    messages = 1 <= arguments.length ? __slice.call(arguments, 0) : [];
    return this.method.apply(null, messages);
  };
  
  return Logger;
  
})();
```

As far as cross-compilers go; it's not quite ugly. You can understand exactly what's going on. Nothing is obfuscated or magical.

## Classes

One of MooTools' hooks is the ability to create classes. Not actual classes, in the sense that they don't change the fundamental, prototypical nature of JavaScript. But close enough that developers can enjoy some of the benefits of Object Oriented code.

To rebuild a class-based framework, we need to be able to build classes. We could do that using loads of code or we could just let CoffeeScript do that for us. When mainstream JavaScript support catches up (with class support) CoffeeScript will adjust without us having to.

## Documentation

The CoffeeScript documentation is great. The language isn't too different from JavaScript that it needs a monolithic documentation website, so [http://coffeescript.org](http://coffeescript.org) does well to explain it. 

Then there's the CoffeeScript Cookbook: [http://coffeescriptcookbook.com](http://coffeescriptcookbook.com). It's full of examples of how to extend and use the language in new and interesting ways.

## Alternatives

There are alternative libraries and frameworks offering the same OOP/neat compilation benefits as CoffeeScript. I've never used any of them in production, so I'm sticking to a bit of what I know for the sake of learning much new. 
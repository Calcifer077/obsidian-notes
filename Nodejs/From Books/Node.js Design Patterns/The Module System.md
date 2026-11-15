---
title: The Module System
created: 2026-11-04
tags:
  - nodejs
---
In the [previous chapter](The%20Node.js%20Platform.md), we got to know about modules.

In generic terms, modules are the bricks for structuring non-trivial applications. Modules allow you to divide the codebase into small units that can be developed and tested independently. Modules are also the main mechanism to enforce information hiding by keeping private all the functions and variables that are not explicitly marked to be exported.

# The need for modules 

A good module system helps in following:

- Having a way to split the codebase into multiple files.
- Allowing code reuse across different projects.
- Encapsulation (or information) hiding.
- Managing dependencies

A module is the actual unit of software, while a module system is the syntax and tooling that allows us to define modules and to use them within our projects.

# The module system and its patterns 


## The revealing module pattern 

One of the bigger problems with JavaScript in browser is the lack of namespacing. What it means is that every variable defined is available in global scope. Internal application codes or third-party dependencies can pollute the scope while exposing their own pieces of functionalities.

A popular technique to solve this class of problems is called the _revealing module pattern_, and it looks like this:

```js
const myModule = (() => {
	const privateFoo = () => {}
	const privateBar = []
	
	const exported = {
		publicFoo: () => {},
		publicBar: () => {}
	}
	
	return exported 
})() // once the paranthesis are parsed, the function will be invoked.

console.log(myModule) // will print both functions, 'publicFoo' and 'publicBar'
console.log(myModule.privateFoo, myModule.privateBar) // will print undefined two times.
```

This pattern leverages a self-invoking function which are also called as **Immediately Invoked Function Expression (IIFE)** and it is used to create a private scope, exporting only the parts that are meant to be public. 

In JS, variables created inside a function are not accessible from the outer scope (outside the function). Functions can use the return statement to selectively propagate information to the outer scope.

In the preceding code, the `myModule` variable contains only the exported API, while the rest of the module content is inaccessible from outside.

# CommonJS modules 

Two main concepts of CommonJS specification:

- `require` is a function that allows you to import a module from the local filesystem.
- `exports` and `module.exports` are special variables that can be used to export public functionality from the current module.

## A homemade module loader 

Let's build a similar (commonJS) system from scratch. The following code mimics a subset of the functionality of the original `require()` function of Node.js.

Below code loads the content of a module, wraps it into a private scope, and evaluates it:

```js
function loadModule (filename, module, require) {
	const wrappedSrc = 
		`
			(function (module, exports, require) {
	${fs.readFileSync(filename, 'utf8')}
})(module, module.exports, require)
		`
    	
	eval(wrappedSrc)
}
```

The source code of a module is essentially wrapped into a function, as it was for the _revealing module pattern_. The difference here is that we pass a list of variables to the module (`module`, `exports` and `require`). Notice that we used `module.exports` which is discussed below.

Another detail to note is that we are using `readFileSync` to read module's content. This make sure that if we are importing multiple modules, they (and their dependencies) are loaded in the right order. 

Let's now implement the `require()` function:

```js
function require(moduleName) {
  console.log(`Require invoked for module: ${moduleName}`);

  const id = require.resolve(moduleName); // (1)

  if (require.cache[id]) { // (2)
    return require.cache;
  }
  // module metadata
  const module = { // (3)
    exports: {},
    id,
  };

  // update the cache
  require.cache[id] = module; // (4)

  // load the module
  loadModule(id, module, require); // (5)

  // return exported variables
  return module.exports; // (6)
}

require.cache = {};
require.resolve = (moduleName) => {
  /* resolve a full module id from the modulename */
};
```

What the above code does:

1. A module name is accepted as input, and its full path is resolved which we call `id`. This task is delegated to `require.resolve()` which will be discussed later on.
2. If the module has already been loaded in the past, it should be available in the cache. If this is the cache, we just return it immediately.
3. If the module has never been loaded before, we set up the environment for the first load. In particular, we create a `module` object that contains an exports property initialized with an empty object literal. This object will be populated by the code of the module to export its public API.
4. After the first load, the `module` object is cached.
5. The module source code is read from its file and the code is evaluated, as we saw before. We provide the module with the `module` object that we just created, and a reference to the `require()` function. The module exports its public API by manipulating or replacing the `module.exports` object.
6. Finally, the content of `module.exports`, which represents the public API of the module, is returned to the caller.

## Defining a module 

Look at the following code:

```js
// load another dependency 
const dependency = require('./anotherModule')

// a private function 
function log() {
	console.log(`Well done ${dependency.username}`)
}

// the API to be exported for public use
module.exports.run = () => {
	log()
}
```

The essential concept to remember is that everything inside a module is private unless it's assigned to the `module.exports` variable. The content of this variable is then cached and returned when the module is loaded using `require()`.

## `module.exports` versus `exports` 

Both are the same thing. The `exports` variable is just a reference to the initial value of `module.exports`.

Node sets things up roughly like this before running your file:

```js
module.exports = {};
let exports = module.exports; // same object, two names 
```

Adding properties works with either name, because both refer to the same object:

```js
exports.foo = 1;          // module.exports.foo is now 1
module.exports.bar = 2;   // exports.bar is also 2
// require() returns { foo: 1, bar: 2 }
```

Reassigning `exports` breaks the link, because you're only rebinding the local variable:

```js
exports = { foo: 1 };
// exports now points at a new object
// module.exports is still the original empty {}
// require() returns {}
```

Reassigning `module.exports` works, because that's the thing `require()` reads:

```js
module.exports = function greet() {};
// require() returns the function
```

If you mix them the last reassignment of `module.exports` wins, and anything you attached to the old object through `exports` is lost:

```js
exports.a = 1;
module.exports = { b: 2};
// require() returns { b: 2}, and a is gone
```

## The require function is synchronous 

Our `require` function and the `require` function by node.js are both synchronous. As a consequence, any assignment to `module.exports` must be synchronous as well. For example, the following code is incorrect and will cause trouble:

```js
setTimeout(() => {
	module.exports = function() {...}
}, 100)
```

You can still export `async` functions. `require` being synchronous means that it won't wait for anything. So the `async` function will just run somewhere else.

## The resolving algorithm 

The term _dependency hell_ describes a situation whereby two or more dependencies of a program in turn depend on a shared dependency, but require different incompatible versions. Node.js solves this problem by loading a different version of a module depending on where the module is loaded from. This is done by package managers and the resolving algorithm using in the `require()` function. 

The resolving algorithm takes a module name and returns its full path. This path is then used to load its code and also to identify the module uniquely. The resolving algorithm can be divided into the following three major branches:

- **File modules**: If `moduleName` starts with `/`, it is considered an absolute path (starting from the root) and it's returned as it is. If it starts with `./`, then it is considered as relative path (starting from the current file), which is than further calculated.
- **Core modules**: If `moduleName` is not prefixed with `/` or `./`, the algorithm will first try to search within the core Node.js modules.
- **Package modules**: If no core module is found, then the search continues by looking for a matching module in the first `node_modules` directory that is found navigating up in the directory structure starting from the requiring module. The algorithm continues to search for a match by looking into the next `node_modules` directory up in the directory tree, until it reaches the root of the filesystem.

## The module cache 

Each module is only loaded and evaluated the first time it is required, since any subsequent call of `require()` will simply return the cached version. Caching has some important functional implications:

- It makes it possible to have cycles within module dependencies.
- It guarantees, to some extent, that the same instance is always returned when requiring the same module from within a given package.

The module cache is exposed via the `require.cache` variable. You can invalidate any cached module by deleting the relative key in the `require.cache` variable. 

## Circular dependencies 

Let's walk through an example to see how CommonJS behaves when dealing with circular dependencies. 

![](../../../assets/Pasted%20image%2020261115095256.png)

A module called `main.js` requires `a.js` and `b.js`. In turn, `a.js` requires `b.js`. But `b.js` requires `a.js`. We have a circular dependency here as module `a.js` requires module `b.js` and module `b.js` requires module `a.js`.  Let's have a look at the code of these two modules:

- Module `a.js`:

```js
exports.loaded = false 
const b = require('./b')
module.exports = {
	b, 
	loaded: true // overrides the previous export
}
```

- Module `b.js`:

```js
exports.loaded = false 
const a = require('./a')
module.exports = {
	a,
	loaded: true
}
```

`main.js`:

```js
const a = require('./a')
const b = require('./b')
console.log('a ->', JSON.stringify(a, null, 2))
console.log('b ->', JSON.stringify(b, null, 2))
```

If we run `main.js`, we will see the following output:

```
a -> {
  "b": {
    "a": {
      "loaded": false
    },
    "loaded": true
  },
  "loaded": true
}
b -> {
  "a": {
    "loaded": false
  },
  "loaded": true
}
```

In order to understand in more detail what happens above:

![](../../../assets/Pasted%20image%2020261115102146.png)

The steps are as follows:

1. The processing starts in `main.js`, which immediately requires `a.js`.
2. The first thing that module `a.js` does is set an exported value called `loaded` to `false`.
3. At this point, module `a.js` requires module `b.js`.
4. Like `a.js`, the first thing that module `b.js` does is set an exported value called `loaded` to `false`.
5. Now, `b.js` requires `a.js` (cycle)
6. Since `a.js` has already been traversed, its currently exported value is immediately copied into the scope of module `b.js`.
7. Module `b.js` finally changes the `loaded` value to `true`.
8. Now that `b.js` has been fully executed, the control returns to `a.js`, which now holds a copy of the current state of module `b.js` in its own scope. 
9. The last step of module `a.js` is to set its `loaded` value to `true`.
10. Module `a.js` is now completely executed and the control returns to `main.js`, which now has a copy of the current state of module `a.js` in its internal scope.
11. `main.js` requires `b.js`, which is immediately loaded from cache.
12. The current state of module `b.js` is copied into the scope of module `main.js` where we can finally see the complete picture of what the state of every module is.

One thing to notice is that module `b.js` has a partial view of module `a.js`, and this partial view gets propagated over when `b.js` is required in `main.js`. The picture can easily be reversed when module `b.js` is required before `a.js`.

# Module definition patterns

The module system, besides being a mechanism for loading dependencies, is also a tool for defining APIs. The aim is to maximize information hiding and API usability, while balancing these with other software qualities, such as _extensibility_ and _code reuse_.

## Named exports 

The most basic method for exposing a public API is using **named exports**, which involves assigning the values we want to make public to properties of the object referenced by `exports` (or `module.exports`). In this way, the resulting exported object becomes a container or namespace for a set of related functionalities. 

The following code shows a module implementing this pattern:


```js
// logger.js
exports.info = (message) => {
	console.log(`info: ${message}`)
}

exports.verbose = (message) => {
	console.log(`verbose: ${message}`)
}
```

The exported functions are then available as properties of the loaded module, as shown in the following code:

```js
// main.js
const logger = require('./logger')
logger.info('This is an informational message')
logger.verbose('This is a verbose message')
```

## Exporting a function 

You can also reassign the whole `module.exports` variable to a function. The main strength of this pattern is the fact that it allows you to expose only a single functionality. This way of defining modules is also known as **substack pattern**. 

```js
// logger.js 
module.exports = (message) => {
	console.log(`info: ${message}`)
}
```

A possible extension to this pattern is using _named_ exports as well. This is a very powerful combination because it still gives the module the clarity of a single entry point (the main exported function) and at the same time it allows us to expose other functionalities that have secondary or more advanced use cases.

```js 
module.exports.verbose = (message) => {
	console.log(`verbose: ${message}`)
}
```

Usage: 

```js
// main.js
const logger = require('./logger')
logger('This is an informational message')
logger.verbose('This is a verbose message')
```

The above pattern encourages the adoption of the **single-responsibility principle (SRP)**.

## Exporting a class 

A module that exports a class is a specialization of a module that exports a function. The difference is that with this new pattern we allow the user to create new instances using the constructor, but we also give them the ability to extend its protoype and forge new classes.

```js
class Logger {
  constructor(name) {
    this.name = name;
  }

  log(message) {
    console.log(`[${this.name}] ${message}`);
  }

  info(message) {
    this.log(`INFO: ${message}`);
  }

  vrebose(message) {
    this.log(`VERBOSE: ${message}`);
  }
}

module.exports = Logger;
```

And, we can use the preceding module as follows:

```js
// main.js 
const Logger = require('./logger')
const dbLogger = new Logger('DB')
dbLogger.info('This is an informational message') 
const accessLogger = new Logger('ACCESS') 
accessLogger.verbose('This is a verbose message')
```

## Exporting an instance 

We can leverage the caching mechanism of `require()` to easily define stateful instances created from a constructor or a factory, which can be shared across different modules. 

```JS
// logger.js
class Logger {
  constructor(name){
    this.count = 0;
    this.name = name
  }

  log (message){
    this.count++;
    console.log(`[${this.name}] ${message}`);
  }
}

module.exports = new Logger('default');
```

This module can be used as follows:

```js
// main.js 
const logger = require('./logger')
logger.log('This is an informational message')
```

Because the module is cached, every module that requires the logger module will actually retrieve the same instance of the object, thus sharing its state. 

## Modifying other modules or the global scope 

A module can even export _nothing_. 

We said that a module can modify other modules or object in the global scope; this is called **monkey patching**. It generally refers to the practice of modifying the existing objects at runtime to change or extend their behavior or to apply temporary fixes.

Following example shows us how we can add a new function to another module:

```js
// patcher.js

// ./logger is another module 
require('./logger').customMessage = function() {
	console.log('This is a new functionality')
}
```

Using our new patcher module is as easy as writing the following code:

```js
// main.js

require('./patcher')
const logger = require('./logger')
logger.customMessage()
```

# ESM: ECMAScript modules 

The most important differentiator between ESM and CommonJS is that ES modules are _static_, which means that imports are described at the top level of every module and outside any control flow statement. Also, the name of the imported modules cannot be dynamically generated at runtime using expressions, only constant strings are allowed. 

For instance, the following code wouldn't be valid when using ES modules:

```js
if(condition){
	import module1 from 'module1'
}else {
	import module2 from 'module2'
}
```

While in CommonJS, it is perfectly fine to write something like this:

```js
let module = null 
if(condition){
	module from 'module1'
}else {
	module2 from 'module2'
}
```

Static imports allow the static analysis of the dependency tree, which allows optimizations such as dead code elimination (tree shaking) and more. 

## Using ESM in Node.js 

Node.js will consider every `.js` file to be written the CommonJS syntax by default; therefore, if we use the ESM syntax inside a `.js` file, the interpreter will simply throw an error. 

There are several ways to tell the Node.js interpreter to consider a given module as an ES module rather than a CommonJS module:

- Give the module file the extension `.mjs`
- Add to the nearest parent `package.json` a field called `"type"` with a value of `"module"`

## Named exports and imports 

ESM allows us to export functionality from a module through the ==🔴`export`== keyword.

In an ES module, everything is private by default and only exported entities are publicly accessible from other modules. 

The export keyword can be used in front of the entities that we want to make available to the module users. 

```js
// logger.js
// exports a function as `log`
export function log(message) {
  console.log(message);
}
// exports a constant as `DEFAULT_LEVEL`
export const DEFAULT_LEVEL = "info";
// exports an object as `LEVELS`
export const LEVELS = {
  error: 0,
  debug: 1,
  warn: 2,
  data: 3,
  info: 4,
  verbose: 5,
};
// exports a class as `Logger`
export class Logger {
  constructor(name) {
    this.name = name;
  }
  log(message) {
    console.log(`[${this.name}] ${message}`);
  }
}
```

If we want to import entities from a module we can use the `import` keyword. The syntax is quite flexible, and it allows us to import one or more entities and even to rename imports. Let's see some examples:

```js
import * as loggerModule from './logger.js'
console.log(loggerModule)
```

In this example, we are using the `*` syntax (also called **namespace import**) to import all the members of the module and assign them to the local `loggerModule` variable. We would get a output of something like this:

```
[Module] {
  DEFAULT_LEVEL: 'info',
  LEVELS: { error: 0, debug: 1, warn: 2, data: 3, info: 4,
    verbose: 5 },
  Logger: [Function: Logger],
  log: [Function: log]
}
```

We can use all exported variables, functions by just using `loggerModule.log` or anything else.

If we don't want to import everything:

```js
import { log } from './logger.js'

// or 

import { log, Logger } from './logger.js'
```

When we use this type of `import` statement, the entities are imported into the current scope, so there is a risk of a name clash. The following code, would not work:

```js
import { log } from './logger.js'
const log = console.log
```

If we try to execute the preceding snippet, the interpreter fails with the following error:

```
SyntaxError: Identifier 'log' has already been declared
```

In situations like this one, we can resolve the clash by renaming the imported entity with the `as` keyword:

```js
import { log as log2 } from './logger.js'
const log = console.log 

log('message from log')
log2('message from log2')
```

## Default exports and imports 

We read about single responsibility principle and how CommonJS follows it above. You do it with with something like this:

```js
module.exports = new Logger('default')
```

In ESM, we can do something similar through what's called a **default export**. A default export makes use of the `export default` keywords and it looks like this:

```js
// logger.js
export default class Logger {
	constructor (name) {
		this.name = name
	}
	
	log (message) {
		console.log(`[${this.name}] ${message}`)
	}
}
```

In this case, the name `Logger` is ignored, and is named while importing itself.

```js
// main.js 
import MyLogger from './logger.js'
const logger = new MyLogger('info')
logger.log('Hello World')
```

We can name the imported variable anything we want at the time of importing itself without using `as` keyword. 

One thing that we cannot do, though, is import the default entity explicitly. In fact, something like the following will fail:

```js
import { default } from './logger.js'
```

The execution will fail with a `SyntaxError`. This happens because the `default` keyword cannot be used as a variable name. It is valid as an object attribute and the following would work:

```js
import * as loggerModule from './logger.js'
loggerModule.default
```

## Module identifiers 

**Module identifiers** (also called _module specifiers_) are the different types of values that we can use in our `import` statements to specify the location of the module we want to load. 

- _Relative specifiers_ like `./logger.js` or `../logger.js`. They are used to refer to a path relative to the location of the importing file. 
- _Absolute specifiers_ like `file:///opt/nodejs/config.js`. They refer directly and explicitly to a full path. Note that this is the only way with ESM to refer to an absolute path for a module, using a `/` or a `//` prefix won't work. 
- _Bare specifiers_ are identifiers like `fastify` or `http`, and they represent modules available in the `node_modules` folder and generally installed through a package manager (such as npm) or available as core Nodej.s modules.
- _Deep import specifiers_ like `fastify/lib/logger.js`, which refer to a path within a package in `node_modules`.

In browser environments, it is possible to import modules directly by specifying the module URL, for instance, `https://unpkg.com/lodash`. This feature is not supported by Node.js.

## Async imports 

As we learned earlier, the `import` statement is static and therefore subject to two important limitations:

- A module identifier cannot be constructed at runtime.
- Module imports are declared at the top level of every file and they cannot be nested within control flow statements. 

In ESM, we can overcome this by using the special `import()` operator. This process is called as `async imports` or `dynamic imports`.

The `import()` operator is syntactically equivalent to a function that takes a module identifier as an argument and it returns a promise that resolves to a module object.

Let's say we want to build a command line application that can print "Hello World" in different languages. Let's create some example modules for some of the languages we want to support:

```js
// strings-el.js
export const HELLO = 'Γεια σου κόσμε'
// strings-en.js
export const HELLO = 'Hello World'
// strings-es.js
export const HELLO = 'Hola mundo'
// strings-it.js
export const HELLO = 'Ciao mondo'
// strings-pl.js
export const HELLO = 'Witaj świecie'
```

Now let's create the main script that takes a language code from the command line and prints "Hello World" in the selected language:

```js
// main.js 
const SUPPORTED_LANGUAGES = ['el', 'en', 'es', 'it', 'pl']
const selectedLanguage = process.argv[2]

if(!SUPPORTED_LANGUAGES.include(selectedLanguage)) {
	console.error('The specified language is not supported')
	process.exit(1)
}

const translationModule = `./strings-${selectedLanguage}.js` // (1)
import(translationModule)                                    // (2)
	.then((strings) => {                                       // (3)
		console.log(strings.HELLO)
	})
```

The first three statements are quite simple.

The second part of the code is where we actually use dynamic imports:

1. First of all, we build the name of the module that we want to import. Note that the module name needs to be a relative path to the module file, that's why we are prepending `./` to the filename.
2. We use the `import()` operator to trigger the dynamic import of the module.
3. The dynamic import happens asynchronously, so we can use the `.then()` hook on the returned promise to get notified when the module is ready to be used. The function passed to `then()` will be executed when the module is fully loaded and `strings` will be the module namespace imported dynamically. After that, we can access `strings.HELLO` and print its value to the console.


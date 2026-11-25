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

## Module loading in depth 

Let's learn how JS code is parsed and evaluated when using ES modules. 

### Loading phases 

The goal of the interpreter is to build a graph of all the necessary modules (a **dependency graph**). A dependency graph can be defined as a **directed graph** representing the dependencies of a group of objects. 

The dependency graph is needed by the interpreter to figure out how modules depend on each other and in what order the code needs to be executed. When the `node` interpreter is launched, it gets passed some code to execute, generally in the form of a JS file. This file is the starting point for the dependency resolution, and it is called the **entry point**. From the entry point, the interpreter will find and follow all the `import` statements recursively in a depth-first fashion, until all the necessary code is explored and then evaluated. 

This process happens in three separate phases:

- **Phase 1 - Construction (or parsing)**: Find all the imports and recursively load the content of every module from the respective file.
- **Phase 2 - Instantiation**: For every exported entity, keep a named reference in memory, but don't assign any value yet. Also, references are created for all the `import` and `export` statements tracking the dependency relationship between them (**linking**). No JS code has been executed at this stage. 
- **Phase 3 - Evaluation**: Node.js finally executes the code so that all the previously instantiated entities can get an actual value. Now running the code form the entry point is possible because all the blanks have been filled. 

In simple terms, we could say that Phase 1 is about finding all the dots, Phase 2 connects those creating paths, and, finally, Phase 3 walks through the paths in the right order.

Due to its dynamic nature, CommonJS will execute all the files while the dependency graph is explored. We have seen that every time a new `require` statement is found, all the previous code has already been executed. This is why you can use `require` even within `if` statements or loops, and construct module identifiers from variables.

In ESM, these three phases are totally separate from each other, no code can be executed until the dependency graph has been fully built, and therefore module imports and exports have to be static. 

### Read-only live bindings 

Another fundamental characteristics of ES modules, which helps with cyclic dependencies, is the idea that imported modules are effectively _read-only live bindings_ to their exported value. TF does that mean?

Let's look at a example:

```js
// counter.js
export let count = 0
export function increment() {
	count++
}
```

If you use this:

```js
// main.js
import { count, increment } from './counter.js'
console.log(count) // prints 0
increment()
console.log(count) // print 1
count++ // TypeError: Assignment to constant variable
```

What we can see in this code is that we can read the value of count at any time and change it using the increment() function, but as soon as we try to mutate the count variable directly, we get an error as if we were trying to mutate a const binding.

This approach is different from CommonJS. In CommonJS, the entire `exports` object is copied (shallow copy) when required from a module. This means that, if the value of a primitive variables like numbers of strings is changed at a later time, the requiring module won't be able to see those changes.

### Circular dependency resolution 

Let's reimplement the circular dependency example we saw in the _CommonJS module_ section using the ESM syntax:

![](../../../assets/Pasted%20image%2020261115095256.png)

Let's have a look at the module `a.js` and `b.js` first:

```js
// a.js
import * as bModule from './b.js'
export let loaded = false 
export const b = bModule
loaded = true

// b.js
import * as aModule from './a.js'
export let loaded = false 
export const a = aModule 
loaded = true
```

Import these in our entry file:

```js
// main.js 
import * as a from './a.js'
import * as b from './b.js'
console.log('a ->', a)
console.log('b ->', b)
```

When we run `main.js`, we will see the following output:

```
a -> <ref *1> [Module] {
	b: [Module] { a: [Circular *1], loaded: true},
	loaded: true
}
b -> <ref *1> [Module] {
	a: [Module] { b: [Cicular *1], loaded: true},
	loaded: true
}
```

The main thing to notice here is that the modules `a.js` and `b.js` have a complete picture of each other, unlike what would happen with commonJS, where they would only hold partial information of each other. We can see that because all the `loaded` values are set to `true`. If we swap the order of imports for the module `a.js` and `b.js`, the final outcome does not change.

Let's learn more about what happens in the three phases of the module resolution:

#### Phase 1: Parsing 

During the parsing phase, the code is explored starting from the entry point (`main.js`). The interpreter looks only for `import` statements to find all the necessary modules and to load the source code from the module files. The dependency graph is explored in a depth-first fashion, and every module is visited only once. This create view of dependencies that look like a tree structure.

![](../../../assets/Pasted%20image%2020261121142050.png)

How it works:
1. We start from `main.js`, the first import leads us to `a.js`.
2. In `a.js` we find an import pointing to `b.js`.
3. In `b.js`, we have an import back to `a.js`, but `a.js` is already visited, so this path is not explored again.
4. At this point, the exploration starts to wind back: `b.js` doesn't have other imports, so we go back to `a.js`. `a.js` doesn't have other `import` statements so we go back to `main.js`. Here we find another import pointing to `b.js`, but we have already explored this module, so this is ignored. 

At this point, our depth-first visit of the dependency graph has been completed and we have a linear view of the modules, as shown below:

![](../../../assets/Pasted%20image%2020261121142352.png)

#### Phase 2: Instantiation 

In the instantiation phase, the interpreter walks the tree view obtained in the previous (parsing) step from bottom to top. For every module, the interpreter will look out for all the exported properties first and build out a map of the exported names in memory:

![](../../../assets/Pasted%20image%2020261121142549.png)

1. The interpreter starts from `b.js` and discovers that the module exports `loaded` and `a`.
2. Then, the interpreter moves to `a.js`, which exports `loaded` and `b`.
3. Finally, it moves to `main.js`, which does not export any functionality.
4. Note that, in this phase, the exports map keeps track of the exported names only; their associated values are considered uninitialized for now.

After this sequence of steps, the interpreter will do another pass to link the exported names to the modules importing them:

![](../../../assets/Pasted%20image%2020261121150239.png)

1. Module `b.js` will link the exports from `a.js`, referring to them as `aModule`.
2. In turn, `a.js` will link to all the exports from `b.js`, referring to them as `bModule`.
3. Finally, `main.js` will import all the exports in `b.js`, referring to them as `b`; similarly, it will import everything from `a.js`, referring to them as `a`.
4. Again, it's important to note that all the values are still uninitialized. In this phase, we are only linking references to values that will be available at the end of the next phase.

#### Phase 3: Evaluation 

The last step is the evaluation phase. In this phase, all the code in every file is finally executed. The execution order is again bottom-up. With this approach, `main.js` is the last file to be executed. This way, we can be sure that all the exported values have been initialized before we start executing our main business logic:

![](../../../assets/Pasted%20image%2020261121180516.png)

In the above diagram, below steps happen:

1. The execution starts from `b.js` and the first line to be evaluated initializes the `loaded` export to `false` for the module.
2. Similarly, here the exported property `a` gets evaluated. This time, it will be evaluated to a reference to the module object representing module `a.js`.
3. The value of the `loaded` property gets changed to `true`. At this point, we have fully evaluated the state of the exports for module `b.js`.
4. Now the execution moves to `a.js`. Again, we start by setting `loaded` to `false`.
5. At this point, the `b` export is evaluated to a reference to module `b.js`.
6. Finally, the `loaded` property is changed to `true`. Now we have finally evaluated all the exports for `a.js` as well.

After all these steps, the code in `main.js` can be executed and all the exported properties are fully evaluated. 

## Modifying other modules 

We saw that entities imported through ES modules are _read-only live bindings_, and therefore we cannot reassign them from an external module.

It is true that we can't change the bindings of the default export or named exports of an existing module from another module, but, if one of these bindings is an object, we can still mutate the object itself by reassigning some of the object properties.

For example, let's alter the behavior of the core `fs` module so that it prevents the module from accessing the filesystem and returns mocked data instead.

```js
// mock-read-file.js
import fs from "fs"; // (1)

const originalReadFile = fs.readFile; // (2)
let mockedResponse = null;
function mockedReadFile(path, cb) {  // (3)
  setImmediate(() => {
    cb(null, mockedResponse);
  });
}
export function mockEnable(respondWith) {  // (4)
  mockedResponse = respondWith;
  fs.readFile = mockedReadFile;
}

export function mockDisable() {  // (5)
  fs.readFile = originalReadFile;
}
```

Explanation of above steps:

1. The first thing to do is import the default export of the `fs` module.
2. We want to replace the `readFile()` function with a mock implementation. Before doing that, we save a reference to the original implementation. We also declare a `mockedResponse` value that we will be using later.
3. The function `mockedReadFile()` is the actual mocked implementation that we want to use to replace the original implementation. This function invokes the callback with the current value of `mockedResponse`. 
4. The exported `mockEnable()` function can be used to activate the mocked functionality. 
5. Finally, the exported `mockDisable()` function can be used to restore the original implementation of the `fs.readFile()` function.

Now let's see a simple example that uses this module:

```js
// main.js 
import fs from 'fs' // (1)
import { mockEnable, mockDisable } from './mock-read-file.js'

mockEnable(Buffer.from('Hello World')) // (2)

fs.readFile('fake-path', (err, data) => { // (3)
	if(err) {
		console.error(err)
		process.exit(1)
	}
	console.log(data.toString()) // 'Hello World'
})

mockDisable()
```

Let's discuss step by step what happens in this example:

1. The first thing that we do is import the default export of the `fs` module. 
2. Here we enable the mock functionality. We want, for every file read, to simulate that the file contains the string "Hello World".
3. We read a file using a fake path. This code will print "Hello World" as it will be using the mocked version of the `readFile()` function. 

The above approach is very fragile. In the `mock-read-file.js`, we could have tried the two following imports for the `fs` module:

```js
import * as fs from 'fs' // then use fs.readFile
```

or

```js
import { readFile } from 'fs'
```

There are certain issues with the preceding two `import` statements:

- We would get a read-only live binding into the `readFile()` function, and therefore, we would be unable to mutate it from an external module. If we try these approaches, we will get an error when trying to reassign `readFile()`.
- Another issue is on the consumer side within our `main.js`, where we could use these two alternatives import styles as well. In this case, we won't end up using the mocked functionality, and therefore the code will trigger an error while trying to read a nonexistent file.

This shows us how monkey patching could be much more complicated and unreliable in the context of ESM. 

We could also use the `syncBuiltinESMExports()` function from the `module` package. When this function is invoked, the value of the properties in the default exports object gets mapped again into the equivalent named exports, effectively allowing us to propagate any external change applied to the module functionality even to named exports:

```js
import fs, { readFileSync } from 'fs'
import { syncBuiltinESMExports } from 'module'

fs.readFileSync = () => Buffer.from('Hello, ESM')
syncBuiltinESMExports()

console.log(fs.readFileSync === readFileSync) // true
```

> Note that `syncBultinESMExports` only work for built-in Node.js modules.


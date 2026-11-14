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


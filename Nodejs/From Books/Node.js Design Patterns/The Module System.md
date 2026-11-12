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


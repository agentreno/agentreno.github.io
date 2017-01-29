---
layout: post
title: "Javascript Modules"
date: 2017-01-29
---

This is an attempt to describe the various methods of creating modules in
Javascript. I have attempted to sum up the advantages and disadvantages of each
method.

### Method #1: No Module, Script Tag

*index.html*

```javascript
...
<div id="target"></div>
<script>
var message = 'hello';
var showMessage = function () {
   document.getElementById('target').innerHTML = message;
};
</script>
...
```

| Pros | Cons |
| ---- | ---- |
| Quick and easy | Can't be reused without copy-pasting... |
| | Which leads to multiple copies of the same code, each having to be maintained separately, with inevitable versioning problems |
| | Each copy tends to become coupled to the page it's on over time e.g. `document.getElementById('target')`
| | All variables are global (usually on `window`) and may conflict with other code

### Method #2: No Module, Separate File

*index.html*

```
...
<div id="target"></div>
<script src="main.js"></script>
<script>
   showMessage(document.getElementById('target'), 'hello');
</script>
...
```

*main.js*

```
var showMessage = function (target, message) {
   target.innerHTML = message;
};
```

| Pros | Cons |
| ---- | ---- |
| Quick and easy | Still sets global variables with potential for conflict |
| More re-usable,  `main.js` can be used elsewhere and `showMessage()` is more flexible now. | What if `main.js` has dependencies of it's own e.g. jQuery? jQuery must be loaded before `main.js`, this problem of load ordering grows in any reasonably sized project. |
| `main.js` can be versioned and maintained in one place | |

### Method #3: Module Pattern, Global Import

*index.html*

```
...
<div id="target"></div>
<script src="messagesLib.js"></script>
<script>
   messagesLib.showMessage(document.getElementById('target'), 'hello');
</script>
...
```

*messagesLib.js*

```
var messagesLib = (function (root) {
   var private_var = 'private';
   root.showMessage = function (target, message) {
      target.innerHTML = message;
   };
})(messagesLib || {});
```

| Pros | Cons |
| ---- | ---- |
| Variables declared inside the IIFE aren't defined as global variables. | Augmentation of `messagesLib` can happen in any order, but load ordering still matters for dependencies. |
| Globals are set explicitly and namespaced within `messagesLib`. | Code that uses `messagesLib` doesn't get to choose the name of the global. Conflicts, while less likely, are still possible. |
| `messagesLib` can be augmented with multiple JS files, each adding more functionality under the namespace. |
| Load order doesn't matter, existing `messagesLib` will be used or created if it doesn't exist yet. |

### Method #4: CommonJS Module

CommonJS modules are a format used extensively in Node and work through `require` and `module.exports`.  Most applications use AMD (see next method). To use it in the browser requires a browser implementation of `require` such as [require1k](http://stuk.github.io/require1k/) or [browserify](http://browserify.org/) (which is also a bundler).

*index.html*

```
<div id="target"></div>
<script src="require1k.js" data-main="./main"></script>
```

*messagesLib.js*

```
module.exports = {
   showMessage: function (target, message) {
      target.innerHTML = message;
   }
}
```

*main.js*

```
var messagesLib = require('./messagesLib');
messagesLib.showMessage(document.getElementById('target'), 'hello');
```

[See a Plunker of this pattern in action](https://plnkr.co/edit/GVog8WrV1FGXnx3bBd0g)

| Pros | Cons |
| ---- | ---- |
| Dependencies are explicit, not hidden as use of a global namespace within the code. | Loads synchronously, code is blocked while the library is fetched from the server. |
| Allows `main.js` to choose the name of the global. | Works this way because it was mainly aimed at server-side module loading from disk.
| `module.exports` makes it clear which variables are public. |
| Load ordering with dependencies is no longer an issue because they are fetched when required.
| Familiar to Node programmers. | 


### Method #5: AMD (asynchronous module definition)

RequireJS is probably the most popular implementation of AMD.

*index.html*

```
<div id="target></div>
<script src="require.js" data-main="main.js"></script>
```

*main.js*

```
define(['messageLib'], function(messageLib){
  messageLib.showMessage(document.getElementById('target'));
});
```

*messageLib.js*

```
define(['hello'], function(hello) {
  return {
    showMessage: function (target) {
      target.innerHTML = hello;
    }
  }
});
```

*hello.js*

```
define([], function () {
  return 'hello';
});
```

[See a Plunker of this pattern in action](https://plnkr.co/edit/vBbZiGOSEksPG0EbkxdP)

| Pros | Cons |
| ---- | ---- |
| Modules are loaded asynchronously, other code on the page isn't blocked. Callback occurs when dependencies are loaded. | Designed for the browser, isn't used on the server-side. |
| Dependencies are even more explicit. | Syntax is a bit more verbose. |
| | Though it doesn't block, dynamic loading of each dependency when required doesn't take advantage of bundling and can be slow. |

### Method #6: UMD (universal module definition)

This is an attempt to write modules in a way that will work whether it is loaded via AMD, CommonJS or plain script tags and window globals. This also helped modules work in the browser and on the server.

*index.html*

```
<!-- Load module using AMD, CommonJS or plain script tags, up to you! -->
...
```

*messageLib.js*

```
(function (root, factory) {
   if (typeof define === 'function' && define.amd) {
      // AMD
      define(['hello'], factory);
   } else if (typeof exports === 'object') {
      // CommonJS
      module.exports = factory(require('hello'));
   } else {
      // Browser globals
      root.messageLib = factory(root.hello);
   }
}(this, function (hello) {
   var private_var = 'private';
   return {
      showMessage: function(target) {
         target.innerHTML = hello;
      }
   };
});
```

| Pros | Cons |
| ---- | ---- |
| Module now works client-side and server-side, using multiple loading methods. | Even if we end up using AMD in the browser and load modules async, we're still making one request per module and potentially loading all of the code upfront. This is now a module loading problem rather than anything to do with the way the module is defined. |
| | All of the methods so far have been compensating for a lack of language support for modules and module loading. See the next method...

### Method #7: ES6 import/export

*index.html*

```
<div id="target"></div>
<script src="main.js"></script>
```

*main.js*

```
import * as messagesLib from 'messagesLib';
messagesLib.showMessage(document.getElementById('target'));
```

*messagesLib.js*

```
export function showMessage(target) {
   target.innerHTML = 'hello';
}
```

| Pros | Cons |
| ---- | ---- |
| Native language support for modules. | This feature seems to be so cutting edge that it's not supported in any modern browser yet. You will need to use a transpiler like Babel. |
| You can selectively import parts of the module and alias them. | The same module loading problems as before apply. This isn't a module definition problem though.|

- A popular combination of module loading / definition methods is now ES6 import/export syntax, Babel for transpilation of this syntax, and Webpack for running Babel and bundling Javascript files together. Angular 2 uses this build process in it's guides and tutorials, while also using Babel to transpile Typescript down to ES5 Javascript.

- Webpack enables other interesting possibilities, like treating CSS as an application dependency, and creating separate bundles for vendor and application code for different caching policies.

- Webpack could also enable multiple bundles for different parts of the application, loading only part of the application initially. When code is likely to be required (e.g. a route is visited), another bundle can be loaded. This isn't quite the same as loading code when it's immediately required (could be a performance issue) since the bundle for a route could contain all of the code likely to be used in that part of the application.

### Notes

- The separate problems of module definition, dependency management and module loading became a little blurred together in this article. 

### References

- I was inspired to write this as an expanded version of [this excellent article on Javascript modules](https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc#.6d69hogly)

- There are many variations on the Javascript module patern, [this article explains some of them well](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)

- [Require1k](http://stuk.github.io/require1k/) seems to have been built as a fun hack but it's one of the only pure CommonJS in the browser implementations I could find that doesn't pre-process the require statements and bundle from them like Browserify.

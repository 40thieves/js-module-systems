# JS Module Systems

- Module system?
  - This is so normal in other languages that you might not have considered it a "feature"
  - It boils down to a system where you can load code from another "module" into the current "module"
    - Often, this "module" is refers to a file, but not necessarily always
    - Importantly, the scope from context A does not affect the scope from context B
    - The context could be third-party code, but also is first party code
    - Improves readability by allowing you to treat other parts of the codebase as a black box
  - Some terms
    - Export: publicly exposing a value within the current module to other modules
    - Import: bringing a value from another module into the current module
  - Rails has autoloading magic, but Ruby has `require 'foo'`
  - Node has `require('foo')`, but we'll look at this in more detail later
- In the beginning
  - JS was intended as a little scripting language to add interactivity to web pages
  - It was not intended as a language with complex dependencies
  - It didn't have any module system for a long time
- But as more complex code was being written in JS, a module system was increasingly required
  - People started inventing their own
  - The simplest thing to do is to wrap everything in an "immediately invoked function expression" (IIFE) then concatenate all the files together

    ```js
    (function() {
      // ... some code here
    })()
    ```

  - IIFEs are very useful since JS has "function-scope", so each IIFE's scope is isolated from each other
  - This is effectively what v1 did, and jQuery also has a version of this too
  - It works ok, but it's still difficult to reference code from another module/file
  - Also everyone's custom "module systems" were sutbly different so it was difficult to interoperate with each other
- There were several competing implementations of a generic module system for JS. We'll look at a couple of the most widely adopted ones
- Asynchronous Module Definition (AMD)
  - [Spec](https://github.com/amdjs/amdjs-api)
  - Example of an AMD-format module:

    ```js
    define(['path/to/moduleA', 'path/to/moduleB'], function(importedValueFromModuleA, importValueFromModuleB) {
      // do something with importedValueFromModuleA/importedValueFromModuleB
      return exportedValue
    })
    ```

  - You might recognise this from the frontend!
    - We used a tool called RequireJS (somewhat confusingly named) to build the frontend that popularised the AMD format
    - We have now migrated to webpack, which can parse the AMD format code (as well as the other formats)
  - All of the module's code must be wrapped in a call to the `define` function
  - The first argument is an array of dependencies: paths to other modules that will be imported into the current module
    - Note that the spec does not define the format of the paths
  - The second argument is a "factory" function that contains the code of the module itself
  - Anything `return`ed from the factory function will be exported from the module
- CommonJS (CJS)
  - [Spec](http://wiki.commonjs.org/wiki/Modules/1.1)
  - Example of a CJS-format module:

    ```js
    var importedValueFromModuleA = require('path/to/moduleA')
    var importedValueFromModuleB = require('path/to/moduleB')
    
    // do something with exportedValueFromModuleA/exportedValueFromModuleB
    
    module.exports = exportedValue
    ```

  - You might recognise this from Node!
    - The CommonJS format was authored by early Node committers and so it was built into Node itself ([Docs](https://nodejs.org/docs/latest/api/modules.html))
  - Every file is effectively wrapped in an IIFE "behind-the-scenes", thereby isolating the scope
  - Dependencies are imported with the `require()` function, which takes the path to the file as an argument and returns the imported value
    - Note that `require()` calls are valid **anywhere** within the module

  - Any value assigned to `module.exports` is exported out of the module
    - Note that because it is just relying on assignment, you can do some slightly weird things like exporting a randomised value. This means that imports/exports can only be determined at runtime (or with some heuristics)

- ES Modules (ESM)
  - [Spec](https://tc39.es/ecma262/#sec-modules)
  - With ES2015 (ES6), a module system was finally added to the language itself
  - Example of an ESM-format module:

    ```js
    import importedValueFromModuleA from 'path/to/moduleA'
    import importedValueFromModuleB from 'path/to/moduleB'
    
    // do something with exportedValueFromModuleA/exportedValueFromModuleB
    
    export default exportedValue
    ```

  - Now that it is part of the language, we don't have to rely on existing structures like functions
  - Dependencies are imported with the `import` keyword
    - Note that while it is valid to put the `import` keyword anywhere within the module, it is **always** hoisted to the top of the module (and thus most linting will prevent it being elsewhere)
  - Values are exported with the `export` keyword
  - ESM is fundamentally different to AMD/CJS: imports/exports are statically determined
    - This is useful since it improves guarantees around "tree-shaking"/dead code removal
  - ESM also introduced the concept of "named exports"
    - In AMD/CJS there is only 1 possible export from a module, although you can "fake" multiple exports using an object. In ESM this is referred to as a "default export" (see above for an example)
    - It is also possible in ESM to define a "named" export:

      ```js
      export foo
      export bar
      ```

      Which are then imported like this:

      ```js
      import { foo, bar } from './foo'
      ```

- Using ESM today
  - Thanks to the magic of webpack, we can start using ESM in the frontend today
  - Well sort of... webpack is actually compiling ESM syntax away to a more CJS style
  - In Node the answer is much more complicated. For complex technical reasons, Node's algorithm can't just switch on support for ESM by default
    - There are subtle differences in how JS parses ESM vs CJS. So it either has to parse each file twice or there has to be some mechanism for declaring a file as using ESM or CJS
    - Most of this is now resolved in latest versions of Node, but there are some [gotchas](https://2ality.com/2019/04/nodejs-esm-impl.html)

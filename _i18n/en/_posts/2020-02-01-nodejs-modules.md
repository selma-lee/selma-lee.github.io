---
layout: post
title: The Module Mechanism in nodejs - A summary of the learning process
date: 2020-02-01 14:09:10
tags: [NodeJs]
categories:
 - backend
---

# 1 Module realization

Steps:

<!-- more -->

1. Path analysis
    Priority: cache checking for core modules > cache checking for file modules > core modules > file modules file modules with relative paths > absolute paths > file modules in non-path form (third-party file modules, i.e., modules in node_modules)
2. file location
    File extension analysis: if there is no file extension, according to the order of .js, .json, .node to fill > Optimization point: if it is a .node and .json file, when passed to the `require()` with the extension, it will speed up a little bit
3. compile execution
    Create a new module object, load and compile according to the path - .js file: read the file synchronously through the fs module and then compile and execute it, during the compilation process, Node wraps the header and tail of the contents of the js file:

        ``` js
        (function (exports, reuire, module, __filename, __dirname) {
          // The contents of the file
          // Introducing dependencies
          var math = require('math')
          // Exporting
          exports.area = function() {
            // ...
          }
        })
        ```
    - .node files: these are extension module files written in C/C++ that don't need to be compiled, the `process.dlopen()` method loads the final compiled file and executes it. The `dlopen()` method has different implementations in windows and _nix platforms, it is encapsulated by libuv compatibility layer, in fact, it is compiled into a .dll file in windows, and into a .so file in _nix, but in order to look more natural, so the extension is unified as .node.
    - .json file: read the file synchronously through the fs module, and parse the result with `JSON.parse()`.
    - The rest of the extensions are treated as .js files.

# 2 Classification of Modules

The modules are divided into two categories:

* One category is Node-supplied modules, i.e., **core modules**.
* The other is user-written modules, namely, **file modules**.

## 2.1 Core modules

Core modules are compiled into binary executable files during the compilation of the Node source code. When the Node process starts, some of the core modules are loaded directly into memory, which is much faster than the ordinary file modules from the disk to find one by one. So when the core module is introduced, the two steps of file location and compilation can be omitted, and the path analysis is prioritized, so the loading speed is the fastest!

The core module is divided into two parts:

**1. Built-in modules (written in C/C++)**

In the [src](https://github.com/nodejs/node/tree/master/src) directory of the Node project, hereafter referred to as built-in modules. Performance is superior to scripting languages.

* Compilation: pre-compiled into a binary file. The `node_extensions.h` file unifies these hashed built-in modules into an array called `node_module_list`.
* Load: Use the `get_builtin_module()` method to retrieve them from the `node_module_list`, execute the

**2. js core module (written in js)**

It's in the [lib](https://github.com/nodejs/node/tree/master/lib) directory of the Node project. One class is used as a wrapping and bridging layer for C/C++ built-in modules; the other class is purely functional modules, which don't have to deal with the underlying layers, but are very important.

* Compilation: Node uses js2c.py, which comes with V8, to convert all js core modules into a C++ array of strings (the strings are the contents of the js core module files) to generate the `node_natives.h` header file. As mentioned above the process of compiling .js files undergoes header and tail wrapping, and differs from the file module as stated in 2.2.2 in the following ways: the way the source code is fetched (the core modules are loaded from memory) and the location where the execution results are cached.
* Load: take out the above array of strings via `process.binding('natives')` and place it in `nativeModule._source`. Load it directly from memory and execute it. _[P25]_
    ``` js
    nativeModule._source = process.binding('natives')
    ```

## 2.2 Core module introduction process

In the core modules, some modules are all written in C/C++; some modules have the core part done by C/C++, and the other parts are wrapped or exported outward by the js implementation to meet the performance requirements (scripting languages such as js are developed faster than static languages such as C/C++, but their performance is weaker than that of the static languages), such as `buffer`, `crypto`, `fs`, `os` etc.

That is, there is a dependency layer relationship where a file module may depend on a core module (Js) and a core module may depend on a built-in module (C/C++). Here is the flow for introducing `os` native modules.

![P25](https://user-gold-cdn.xitu.io/2020/2/1/16fff891e1381541?w=864&amp;h=862&amp;f=png&amp;s=99119)

## 2.3 Documentation module

Dynamically loaded at runtime, including the complete path analysis, file location, compilation and execution of these processes described above, is slower than the core module.

* Compile: the process of compiling the .js file as described above undergoes header and tail wrapping
* Load: dynamically loaded at runtime

Written by the developer , including ordinary JS module and C/C++ extension module . The main calling direction is that ordinary JS modules call C/C++ extension modules.

**C/C++ extension modules**

C/C++ extension module belongs to the category of file module. Mainly to improve performance, such as js only double type data type, and bit operation need to convert double type to int type, so js level to do bit operation efficiency is not high. This time you need to C / C + + extension module.

* Compile: compiles to a .node file
* Load: Load the files generated by the final compilation via the `process.dlopen()` method and then execute them.

## 2.4 Module call stack

![P33](https://user-gold-cdn.xitu.io/2020/2/1/16fff89c26dfe60d?w=1058&amp;h=562&amp;f=png&amp;s=59670)

Documentation modules include: JS modules and C/C++ extension modules; core modules include: js core modules and C/C++ built-in modules.

JS modules in file modules may call C/C++ extension modules, file modules may also call js core modules, js core modules may depend on C/C++ built-in modules. File modules may also call C/C++ built-in modules directly.

## 2.5 Third-party file modules (npm dependency packages)

It is basically the same as the normal file module, except for the difference in path analysis.

# 3 Shared modules for front and back end

* CommonJS specification: `Node.JS` follows the `CommonJS` specification. `module.exports = xxx` exports, `require()` introduces. Synchronized loading of modules.
* AMD: Requirejs' canonicalized output for module definitions, asynchronous module definitions, module loading does not affect the operation of statements following it, dependency fronting. `define` definition, `require` introduction.
    ``` js
    require(['clock'],function(clock){
      clock.start();
    });
    ```
* CMD: Seajs' normalized output for module definitions, synchronized module definitions, dependencies in close proximity. `define` definitions, `require` introductions.
    ``` js
    define(function(require, exports, module) {
       var clock = require('clock');
       clock.start();
    });
    ```
* es6 specification: `export` exports, `import` imports.

## 3.1 Compatible with multiple module specifications

In the current project, the general practice is:

babel converts es6 code to es5 (CommonJS specification). webpack does browser compatibility with CommonJS.

Referring to Nodejs, webpack mainly implements `exports` and `require` functions (`__webpack_require__`) and passes in `module`, `exports` and `__webpack_require__` parameters. See [webpack modularity principles-commonjs](https://segmentfault.com/a/1190000010349749) for details.

# Reference

* "In-depth nodejs.
* [webpack modularity principles-commonjs](https://segmentfault.com/a/1190000010349749)

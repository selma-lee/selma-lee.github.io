---
layout: post
title: A Simple Implementation of the Virtual DOM
date: 2019-09-09 18:29:10
tags: [React, Virtual DOM]
categories:
 - web-framework
---

# 1. what is the virtual DOM

Virtual DOM in short, using JS to follow the DOM structure to realize the tree structure of the object, you can also call the DOM object

<!-- more -->

# 2. Ideas for realization

1. simulate the DOM with JS objects (virtual DOM)
2. turn this virtual DOM into a real DOM and insert it into the page (render)
3. if an event occurs that modifies the virtual DOM, compare the differences between the two virtual DOM trees and get the difference object (patch array) (diff)
4. apply the diff object (patch array) to the real DOM tree (patch)

# 3. Code repository

Refer to the [official React website](https://zh-hans.reactjs.org/docs/introducing-jsx.html) to implement methods such as `createElement`. In order to save configuration such as hot update, `yarn create react-app my-app --typescript` is used to build the project.
See the code and comments directly for more details
[github](https://github.com/seminelee/virtual-dom)
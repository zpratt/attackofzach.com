---
title: Testing Javascript Singletons
tags:
  - javascript
  - tdd
id: 250
categories:
  - Guide
  - Javascript
  - Uncategorized
date: 2015-03-19 12:15:35
---

Testing javascript singleton modules can be a challenge, especially ones that maintain some sort of internal state. Examples of singletons that manage internal state are usually either a factory or cache manager. In order to effectively test these sorts of modules, you often either resort to exposing a method to reset the state, or carefully order your tests... until now!

To eliminate this problem, I propose the following solution (assuming you're using a spec-style test):

1.  Wait to require the module you're testing until all stubs, spies, and mocks are defined in a `beforeEach`
2.  `require` the module you're testing after stubbing/spying/mocking in the `beforeEach`
3.  in the `afterEach`, remove the module you're testing from the require.cache

        *   this effectively destroys the singleton and when it is `require`'d again the the `beforeEach` it will be rebuilt.

Here's how you remove a module from the `require.cache`:

<script src="https://gist.github.com/zpratt/c3aa4c65518b9a911804.js"></script> 
I'm assuming that you have your tests in a sibling directory to a lib directory, where your production modules live. If not, you can adjust the path spec accordingly.

Other things to be mindful of are:

*   Event listeners (both DOM and custom events)*   The state of the DOM itself*   Callbacks you've stored off in the body of stubs and/or spies within your test suite

These are all things that need to be cleared out in the `afterEach` as well.

But wait, I'm a front-end developer and I use karma or testem to execute my tests in a real browser. Well, I personally believe runners are a source of pain and frustration and should be avoided. However, it is theoretically possible with karma to achieve this by removing the script element that corresponds to the module you're testing in the `afterEach` and then append it back immediately after removing it. I am not familiar enough with testem to give equivalent advice.
---
title: BDD with MEAN - The Server Part 1
tags:
  - bdd
  - guide
  - javascript
  - mocha
  - yadda
id: 178
categories:
  - Guide
  - Javascript
  - Testing
date: 2014-02-20 01:25:00
---

As with any new endeavor, it pays to spend some time trying various solutions out and sometimes failing miserably. This is especially true for us progressive nerds who like to live on the bleeding edge without things like Stack Overflow to constantly save our ass. What I'd like to do is to help you avoid going through the pain of figuring out what works and what doesn't.

As I mentioned in my previous post, I already have a project that serves as a working example of if you wish to jump straight into the code: [https://github.com/zpratt/thoughtsom](https://github.com/zpratt/thoughtsom) . All of the gists used in this post were pulled from that project.

The first step on our journey to effective BDD testing with the MEAN stack will be to start wiring up the various tools we'll need to use to get a working environment. Afterwards, we'll build out a helper to manage our test environment and fixtures.

Let's start by reviewing our toolbox:

## Our Tool Box

*   [Grunt](http://gruntjs.com/)

        *   Grunt is used for general task running, . It can be used in a similar manner to how you might use rake when working on a ruby-based project. It is particularly useful on the server side when you combine it with watch and nodemon, which are more runtime oriented.
*   [Yadda](https://github.com/acuminous/yadda)

        *   As mentioned earlier, Yadda will be serving as our BDD layer/framework. Yadda itself needs to integrate with another test framework like Mocha or Jasmine to provide a complete stack. It includes a Gherkin parser, multilingual support, and parameterized step definitions.
*   [Mocha](http://visionmedia.github.io/mocha/)

        *   Mocha is a general purpose javascript test framework. Mocha will be used more explicitly on the TDD side, but Yadda's integration with it allows you to make use of the Before, BeforeEach, After, and AfterEach hooks to setup and tear down your test fixtures.
*   [Sinon](http://sinonjs.org/)

        *   Is a standalone test library that has an great support for stubing, spying, mocking, and additional assertions that compliment your regular xUnit style assertions when TDD'ing. Sinon also provides a fakeServer API that can be useful for prototyping endpoints or as a test double for external dependencies in BDD tests.
*   [Chai](http://chaijs.com)

        *   Provides both bdd style assertions (expect and should) as well as xUnit style assertions.
*   [grunt-mocha-cov](https://github.com/mmoulton/grunt-mocha-cov)

        *   A grunt plugin for running mocha tests. Since we've chosen to use mocha as the underlying test framework for Yadda, this will help automate the execution of our BDD tests.
*   [Supertest](https://github.com/visionmedia/supertest)

        *   Used to test our express routes. Supertest has the ability to consume an express app and it's associated routes. One of the things I especially like about Supertest is that it does not require you to have previously spun up an instance of your application in order to execute the tests.
*   [Proxyquire](https://github.com/thlorenz/proxyquire)

        *   Allows us to inject stubs or fake objects into the require namespace.
*   [Casual](https://github.com/boo1ean/casual)

        *   A robust library for generating test data.
*   [node-config](https://github.com/lorenwest/node-config)

        *   require paths and database connection information quickly gets messy. I like to extact some of that mess out into config files.
*   [Yaml](https://github.com/nodeca/js-yaml)

        *   I prefer yaml config files because they are short and sweet. This module works nicely with the node-config module.

## Folder structure

The next most logical step is to establish our folder structure. I have a specific folder structure that I like to use and I am not currently aware of a yeoman generator that will scaffold anything based on my requirements, so for now I recommend creating the following stucture:

<pre>|-- config
 |-- Gruntfile.js
 |-- LICENSE
 |-- package.json
 |-- README.md
 |-- src
 |   |-- server
 |   |   |-- app
 |   |   |   `-- controllers
 |   |   |-- models
 |   |   |-- repositories
 |   |   `-- schemas
 `-- test
     |-- acceptance
     |   -- server
     |       |-- features
     |       |   |-- step_definitions
     |       |   `-- support
     |-- helpers
     |   -- stubs
     `-- unit
         |-- server
         |   |-- app
         |   |   -- controllers
         |   `-- repository
</pre>

## The configuration file - config/default.yml

<script src="https://gist.github.com/zpratt/9107308.js"></script>

The configuration file is pretty straightforward, for the most part. I like to establish some path prefixes to avoid the 3rd level of require hell, which is crazy relative path references. The one cryptic part might be the knownObjectId key, which I will explain later in the section on world.js.

## The Gruntfile

<script src="https://gist.github.com/zpratt/9107164.js"></script>

The most important part of this is the `mochacov` configuration block. I am well aware of the simple mocha plugin, however, I have found that mochacov appears to be more active and it also gives much better output when things go wrong (which happened to me quite often while I was experimenting with how to get this all configured). Aside from my usage of mochacov, I have chosen to run jshint against both the production code and the tests. I feel that this is a good practice, which helps ensure consistency between code styles of production code and test code, as well as a safety net to help avoid tricky syntactic boogers.. I mean issues that are not always obvious.

## The World

One of the useful things I learned from my cucumber-js experiment is the value of having a helper that can aid in setting up and tearing down the test environment and related fixtures. I have chosen to borrow this concept for usage with Yadda/Mocha as well. The world.js file should go in the following directory: `test/acceptance/server/features/support/world.js`

<script src="https://gist.github.com/zpratt/9107407.js"></script>

There are 3 main highlights in the world.js helper:

1.  It handles connecting and disconnecting from our local mongodb instance with `connectToDB(done)` and `disconnectDB(done)`.

        *   Note that both of these take `done` as a parameter, which is the callback that mocha will use to determine when an asynchronous function is truly finished.
2.  It handles inserting test data for us, with at least one known ObjectId with `createThought(done)`

        *   This method currently only inserts one row of data into the datastore, however, the important part is that it inserts a record with a known primary key that we can then use in our route test to make sure that we can actually return a result when we hit the /thought/:id route. The value is pulled from our `knownObjectId` key in our `default.yml` configuration file.
3.  It clears the database (with `clearDB(done)`), which means that each of our test runs are idempotent.

## Summary

As I have started to dig into this, I have realized that I could essentially write a book on this topic. If you're anxious to take off with this and looking for more examples, I encourage you to take a look at my [thoughtsom repo on github](https://github.com/zpratt/thoughtsom), which I continue to build out and try to commit chunks to each week. Getting the test environment and project structure right at the beginning will help us stay organized and have a solid configuration base to build on top of.

I plan on breaking this up into separate posts in hopes to make this a complete, but consumable reference.

Here's a rough sketch of what to expect:

1.  Part 1 (this post): cover the basic libraries we'll use, and establish the prerequisites for automating our test runs.
2.  Part 2: Write our first feature and failing step definition using Supertest to verify an express route.
3.  Part 3: A little application architecture and more advanced usages, such as multiple step definitions and features.
4.  Part 4: Unit test overview: stubs and mocks with sinon

After these 4 parts, I plan to begin covering the UI side of things and demonstrate how most of the tools will translate to being used on the both the server and the browser sides of testing. I welcome any feedback on the quality of this article and the direction you'd like to see this take. Feel free to flame/praise me on [twitter](https://twitter.com/zpratt).
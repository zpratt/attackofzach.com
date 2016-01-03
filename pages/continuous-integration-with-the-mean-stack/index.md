---
title: Continuous Integration with the MEAN Stack
tags:
  - continuous integration
  - guide
  - javascript
  - MEAN
id: 202
categories:
  - Guide
  - Javascript
  - Testing
date: 2014-02-24 08:00:29
---

[Travis-CI](https://travis-ci.org/) has great support for continuous integration with the MEAN stack. It took some research in order to achieve a complete solution, which then caused some revamping of my initial config. I want to share my notes with my agile hacker friends so you can take advantage of this with your projects, both large and small. This article is meant to be a continuation of my previous articles on setting up a proper test environment for the MEAN stack. Some things in this article are contextual and may seem out of place unless you've read the previous articles:

1.  [TDD and BDD With The MEAN Stack: Introduction](http://attackofzach.com/tdd-bdd-mean-part1/)
2.  [BDD with MEAN - The Server Part 1](http://attackofzach.com/bdd-with-mean-the-server-part-1/)

Using continuous integration for large scale projects is, in my opinion, a requirement. Some might debate its value for small/personal projects, however, I'd like to put that to rest as well.

### For the non-believers

Some may think that CI for small scale, and/or personal projects, is overkill. However, I believe CI for small/personal projects provides the following benefits:

1.  Verification that your application can run in an isolated environment away your personal machine on standard infrastructure.
2.  Marketing and a certain level of assurance to potential consumers that you treat your tests as a first class concern.

        *   Users can see first-hand that your tests run effectively, instead of just telling them to run `npm test` themselves after downloading your code.
    *   It shows attention to the engineering process as well as an additional level of transparency.
3.  It is an enabler for Continuous Deployment. In fact, Travis-CI supports [continuous deployment](http://docs.travis-ci.com/user/deployment/) to several different environments out of the box.

I believe strongly in maintaining the application in an always shippable state. Having a proper CI strategy is one step on that path.

### Key Elements

*   Externalized application configuration
*   Travis-CI config for public github projects

### Application Configuration

I chose to use the [node-config](https://github.com/lorenwest/node-config) module with javascript format. There are two keys strengths to this module:

1.  It supports using javascript for the configuration format, which makes it easy to build reusable chunks of configuration by just exposing some simple functions to return parts of the configuration object literal.
2.  It maps the `NODE_ENV` environment variable to configuration files stored in the config directory in the root of your project by convention. This allows you to create specialized configurations for development, CI, production, etc.

I chose `travisci` as the name of my CI environment, which, as we'll see later, is set as the exported value of the `NODE_ENV` environment variable in the `.travis.yml` config file.

#### The reusable parent configuration

<script src="https://gist.github.com/zpratt/9176723.js"></script>

#### travisci Environment Configuration

<script src="https://gist.github.com/zpratt/9176902.js"></script>

### Travis-CI Configuration

<script src="https://gist.github.com/zpratt/9176787.js"></script>

The key ingredients to tying this configuration back to our node-config configuration file are the `NODE_ENV` and `BUILD_DIR` environment variables. With Travis-CI, you can export environment variables by declaring them under the following yaml block:

`env:
  global:`

Since I've chosen to use Grunt to automate task execution for the project, and bower to manage front-end dependencies, these global modules must be installed during the `before_script:` hook. The other blocks (`language`, `node_js`, and `services` tell Travis-CI what sort of environment is needed. What's really cool about this, is that we can go ahead and run our BDD tests in this environment as well since we have a mongodb instance available. This is one of my favorite features of Travis-CI.

#### Secret Sauce

I did some digging and discovered that Travis-CI clones your repo into `$HOME/&lt;username&gt;`, so the effective working directory will be `$HOME/&lt;username&gt;/&lt;cloned repo name&gt;`. However, in their [environment variable documentation](http://docs.travis-ci.com/user/ci-environment/#Environment-variables), it is recommended to not rely upon the value of the `HOME` environment variable. To work around this, I chose to hook into this in the configuration by using the `pwd` sub-shell command in the .travis.yml file as part of the value for `BUILD_DIR`. In the travisci.js configuration file, I then utilize the BUILD_DIR as the initial working directory. This is used to avoid having a bunch of relative path references in the `require` calls of the application.

### Parting Thoughts

I'm quite pleased with this setup for the Travis-CI environment. I went ahead and included their build badges in my [thoughtsom github repo](https://github.com/zpratt/thoughtsom). I intended to also include code coverage integration with [coveralls](https://coveralls.io/), however, I ran into some issues with the blanket code coverage tool. I plan to give istanbul a spin, but I have not had time to properly set that up yet. The long-term goal is to get continuous deployment to [OpenShift](https://www.openshift.com/) working.
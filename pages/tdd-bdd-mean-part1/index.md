---
title: 'TDD and BDD With The MEAN Stack: Introduction'
tags:
  - bdd
  - guide
  - javascript
  - mocha
  - tdd
id: 157
categories:
  - Guide
  - Javascript
  - Testing
date: 2014-02-08 03:20:13
---

As the [MEAN stack](http://thecodebarbarian.wordpress.com/2013/04/29/easy-web-prototyping-with-mongodb-and-nodejs/) is growing in adoption, a variety of testing strategies have sprouted up on the interwebs. As I have started to dig deeper into automated testing strategies for both sides of the MEAN stack, I have found it difficult to find advice or material on how to setup an environment to support the [mockist style](http://martinfowler.com/articles/mocksArentStubs.html#ClassicalAndMockistTesting) of test-driving the implementation. Many articles offer some light recommendations, but are typically more classicist-oriented. I'd like to help fill the gap by outlining a tooling strategy which I believe enables the following:

*   Automation
*   Mockist unit testing
*   BDD/ATDD testing
*   End-To-End system testing

In case you want to dive straight into the code, I've set up a [project on github](https://github.com/zpratt/thoughtsom) to prove out the technology.

I'm going to assume you've already got a working MongoDB and Node.js environment. If not, take a look at a [vagrant solution](https://github.com/cmoudy/mean-vagrant) or [mean.io](http://mean.io).

One of my goals has been to try to find tools that can work on both the server and the browser. I also put a lot of stock into automation, so I have aimed to avoid tools that require manual loading of browser pages to launch the tests. This has proven to be quite a challenge, but the xUnit-side of testing is much closer to that goal than the BDD-side. However, all hope is not lost.

Let's get down to business and outline the tools that are working well on both sides:

*   [Grunt](http://gruntjs.com/) - The task runner
*   [Mocha](http://visionmedia.github.io/mocha/) - The test framework
*   [Chai](http://chaijs.com/) - The assertion library
*   [Sinon](http://sinonjs.org/) - The stub, spy, and mock swissarmy knife

Some tools are sort of in the middle, where they can be made to work in both the browser and the server, but they aren't rock solid in one or the other:

*   [Yadda](https://github.com/acuminous/yadda) - BDD layer that works nicely with Mocha+Chai (needs a little help on the browser side, but [I'm working on that](https://github.com/zpratt/yadda-karma-example))
*   [Cucumber-js](https://github.com/cucumber/cucumber-js) - (BDD) works great on the server-side, but fails in the browser side (particularly because of a lack of support for karma integration)

And finally, some tools are targeted at one side or the other:

*   [Karma](http://karma-runner.github.io/) - THE test runner for the browser
*   [karma-browserifast](https://github.com/cjohansen/karma-browserifast) - A browserify plugin for Karma, which is needed to automate running Yadda BDD tests in the browser
*   [Supertest](https://github.com/visionmedia/supertest) - Awesome library for testing Express-based RESTful endpoints
*   [CasperJS](http://casperjs.org/) - Drive user interactions through the UI (and works with Yadda)

There is more to come. I plan to write two more articles to dig further into to the details of what it means to test drive on the server side and in the browser. For now, take a look at my [Yadda/Karma example on Github](https://github.com/zpratt/yadda-karma-example) for BDD in the browser and [thoughtsom](https://github.com/zpratt/thoughtsom) for BDD and unit testing on the server. I plan to build out of a few more code examples on the browser side of testing before the next set of articles. In the mean time, I welcome your feedback.

P.S. I am aware of [Protractor](https://github.com/angular/protractor), but I have not had much of a chance to experiment with it yet.
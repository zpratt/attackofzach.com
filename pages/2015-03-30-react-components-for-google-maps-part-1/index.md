---
title: React Components for Google Maps - Part 1
tags:
  - google maps
  - guide
  - javascript
  - react
id: 255
categories:
  - Guide
  - Javascript
date: 2015-03-30 08:16:13
---

Building [React](http://facebook.github.io/react/) components for use with [google maps](https://developers.google.com/maps/documentation/javascript/reference) can present some challenging problems for those who aren't aware of the basics/quirks of developing with google maps. I aim to shed some light on the fundamentals of how to get the two to play nicely together, and to encourage developing the components in a way that will promote testability. As always, feel free to [jump right into the code](https://github.com/zpratt/coffee-hunt) if you don't need the TL;DR description of how it all works.

As you'll notice in the example, I have created two npm modules to support this:

*   [async-google-maps](https://github.com/zpratt/async-google-maps) - provides a facade for asynchronously creating google map instances as well as a base overlay views that are positioned by a lat/lng
*   [idle-maps](https://github.com/zpratt/idle-maps) - builds on top of the async-google-maps module, by providing a set of React components for creating google map instances and overlay content

## Why?

The standard google maps modules are fine for simple use cases, such as displaying a marker for a given Lat/Lng point or InfoWindows for descriptive content about a given point. However, things get interesting when you need to be able to have full control over the style of a component, or to easily update content based on changes from the server. There are third-party libraries out there that will wrap the google maps primitives, but those have their own problems and often contain a bunch of bloat trying to handle all possible user interactions and browser quirks.

## Prerequisites

1.  A basic understanding of spatial data primitives: coordinate systems, projections, basic geometric data types
2.  A basic understanding of React
3.  A basic understanding of google maps
4.  A basic understanding of browserify
5.  A google maps API key

## Where we're going

As an introductory step, we will start by building a custom [OverlayView](https://developers.google.com/maps/documentation/javascript/reference#OverlayView) that is positioned by a point (as opposed to being positioned by a bounds, which is also possible). We will keep this simple by focusing on the basics using React with google maps and avoid fetching data from the server or binding UI interactions to a client-side router. We'll cover those in later articles.

## Separating concerns

My preferred method for separating concerns when building custom OverlayViews is to leverage the following pattern:

*   Build a constructor function which inherits from the `google.maps.OverlayView` constructor.
*   Implement a facade, (in the spirit of the [Command Pattern](http://en.wikipedia.org/wiki/Command_pattern)) which abstracts away the complexity of the coordination that needs to occur between the OverlayView and the React component
*   Implement the React component in such a way that it knows nothing about any of the google maps APIs

## What's next?

In the next article, we will take a deep dive into the implementation and highlight a few of the pitfalls I discovered while working on this myself.
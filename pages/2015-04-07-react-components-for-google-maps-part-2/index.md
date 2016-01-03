---
title: React Components for Google Maps - Part 2
tags:
  - google maps
  - guide
  - javascript
  - react
id: 267
categories:
  - General
  - Javascript
date: 2015-04-07 08:31:34
---

Making React and Google Maps play nice with each other is not a complex task, provided you have some insight in how to open seams in the google maps layer to allow it to interoperate with other libraries. I have learned this from the school of hard knocks, so I hope to shed some light on the tricky parts to save you some time. If you haven't already, I recommend you read [part 1 of this series](http://attackofzach.com/react-components-for-google-maps-part-1/) for some important background information.

Let's dig into the implementation. I have broken out the examples into a set of gists. If you don't care to follow through each step, you can [jump straight to the full example](https://gist.github.com/zpratt/7ed18be2573e1bb194cc) of the OverlayView.

### Implementing the OverlayView

There are 3 high level requirements for creating a custom OverlayView. The module must:

1.  Prototypically inherit from the `google.maps.OverlayView` class
2.  Implement a `onAdd` method, which will be called once the map has been set for the overlay
3.  Implement a `draw` method, which will be called each time the map is zoomed

#### Secret sauce

The secret sauce to getting the overlay and React to play nice is to take the DOM element representing the content of the overlay as a parameter through the constructor. As we will see later on, this will allow us to handle rendering the content of the overlay in the React component, but position it on the map through the overlay.

#### Extend the `google.maps.OverlayView` class

<script src="https://gist.github.com/zpratt/bc28f1d36da2eea584e3.js"></script>

#### Adding the overlay to the DOM

<script src="https://gist.github.com/zpratt/d2310a9cd8f9618a3b56.js"></script>

The `onAdd` method is the point at which the overlay can insert itself into the DOM. This is achieved by using the `getPanes` method, which is inherited from the parent OverlayView class. There are multiple panes, but for our particular case, we'll use the `overlayLayer` pane. This will allow us to control any click-like behavior completely within the DOM structure of the overlay.

It's important to understand the purpose of this method when integrating an overlay with a particular framework or library, because typically inserting a particular view into the DOM is often handled by the framework. This method is our seam that allows us to inject content into the map without having that content coupled to the map itself.

#### Positioning the overlay on the map

<script src="https://gist.github.com/zpratt/87db98b7ebc36b31a2c2.js"></script>

The `draw` method is invoked when the overlay needs to calculate its position on the map. In the simplest case, an overlay is positioned by a single lat/lng point, however, it is also possible to position the overlay by a bounds as well. We'll focus on the simplest case in this article. In order to translate a lat/lng to a pixel, we need to get a reference to the projection of the overlay. Projections are google maps' way of turning lat/lng points into pixels relative to something within the map structure.

To safely get a reference to a projection, we have to wait for the map instance to be idle. I have found it is best to handle this dependency outside the overlay, particularly if you plan on having to render hundreds or thousands of overlays on the map. I will discuss how to implement handling this externally as part of implementing the facade. Regardless of where it is handled, the way to accomplish knowing when the map is idle is by adding a listener to the map instance for the `idle` event.

### Performance

From my experience, when rendering a high volume of overlays on the map (300+), the `draw` method has the highest likelihood to impact rendering performance. There are two reasons for this. First of all, relatively speaking, it is expensive to determine the dimensions of an element in the DOM. We need the dimensions of the overlay in order to properly position it on the map. Secondly, the `draw` method will be invoked each time we zoom, therefore, we will need the dimensions of our overlay again in order to correctly position it.

When rendering high volumes of overlays, we can dramatically improve performance by caching the dimensions. To take it a step further, if your overlays all have a consistent height and width, you can calculate the dimensions once and use the cached copy for all other instances. Exactly how to achieve that is outside the scope of this article, but I wanted to highlight it, because it has caused me a great deal of heartburn in the past.

### Implementing the facade

Our general requirements for the facade are:

1.  Handle the creation of the google map instance (a factory)
2.  Provide a facility for consumers to know when the map instance is idle without having to add listeners themselves
3.  Provide a factory for creating our custom overlays

#### Map facade

<script src="https://gist.github.com/zpratt/856f523fd72e6dfc76f2.js"></script>

The best way I have found to handle both items 1 and 2 is to define a method that will handle creating the map instance, but returns a `Promise`. The `Promise` is resolved once the map is idle, and should resolve with the newly created map instance. This allows consumers to bind to the `Promise` and to perform any actions that are dependent upon the map being idle. As consumers, we don't care if we bind to the map early or late in its lifecycle, once it's idle, we know our callback will be invoked and we'll have access to an instance of the map.

#### Overlay factory

<script src="https://gist.github.com/zpratt/8c274163e6db89ec1b3f.js"></script>

To avoid coupling ourselves to our custom overlay, it is best to implement a factory to handle creating instances of the overlay for us. This allows us to handle the setup of the overlay in a single place, and will improve the testability of modules that need to create an overlay instance.

### Implementing the React component

<script src="https://gist.github.com/zpratt/8833c6f4b4b83437b411.js"></script>

By abstracting away the details of defining the overlay and its interactions with the map in our facade, there is very little overlay-specific logic that needs to be included in the component. All we need to do is call our factory. The key is to do this as part of the `componentDidMount` method of the component, since this is the point in time where we will have access to the DOM node of the component. You are free to handle the rendering of the content for the overlay in the `render` method of the component.

#### Secret sauce

The key to achieving this is to render the overlay component with a detached div element and not an element that is already in the DOM. This allows us to render our content in the detached element and to pass it to the overlay. The component can then safely share it's DOM node with the overlay. [Here's an example](https://github.com/zpratt/idle-maps/blob/master/examples/map-component.jsx) of how to achieve this with my [idle-maps](https://github.com/zpratt/idle-maps) module.

## Parting Thoughts

Where I work, we have a very map-centric application where, at times, we render 2,000+ overlays. We also have a variety of types of overlays that we leverage. Following the patterns I have laid out has proven to be very effective for us. While building out this capability within our application, it would have been quite valuable to have known some of these patterns early on. I hope by documenting them here, I can save another developer some time and pain going down this path.

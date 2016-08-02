---
layout: post
title: How Fast Can We Make Webpack?
summary: My quest for the optimal webpack developer experience.
categories: javascript
thumbnail: bolt
tags:
 - javascript
 - webpack
 - optimization
---

![Grail](https://i.imgur.com/2bYvGQh.jpg)

I've been working with React for a little more than a year now.  As our applications have grown in scope and complexity, we have started working with live reloading, moved to using css-modules, and have cast a wide 3rd party net.  Build times have been growing and growing and growing.  I battled with build times, trying to keep them reasonable, but still they grew and grew.  I knew that somewhere out there, my holy grail existed: a webpack configuration that didn't sacrifice development experience, was maintainable, and most of all, **fast**.


I'm going to be working on a series of blog posts that walk through building a comprehensive and optimal webpack configuration for React.  At the end, we will have a webpack configuration that supports hot modules, source maps, css-modules (with SASS), bundling of assets, and optimization of a production bundle.

These tutorials assume you have at least some prior knowledge with configuring webpack.  It would simply be too unwieldy to try and cover everything from the ground up in one place.

1. Part 1: The Basics - React with Hot Module Replacement
2. Part 2: The Fun Stuff - css-modules, asset bundling, and JSON loading
3. Part 3: Making Development Fast - DLL modules, babel gotchas
4. Part 4: The Production Bundle - size and asset optimization, code splitting

All of the source code here is available in the [git repository](http://site.com).
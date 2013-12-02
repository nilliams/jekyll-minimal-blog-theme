---
layout: post
title: Raspberry Pi Particles
subtitle: Fun with OpenVG on the Raspberry Pi
date: 2013-11-28 10:59:45
tags:
 - C
 - Node
 - JavaScript
 - Embedded
---

One of the first things I did with my Pi was to try and figure out what's possible in terms of graphics. This included a fun exploration of OpenVG (and very simple particle effects) on the Pi. I recommend checking out these projects, both of which I'll definitely be playing with more:

 * [node-openvg-canvas](https://github.com/luismreis/node-openvg-canvas) - If you're a JavaScript/Node fan, it can be pretty magic having your web demos work outside the browser.
 * [OpenVG Testbed](https://github.com/ajstarks/openvg) - If you don't mind getting your hands dirty with C. Examples and useful helper functions.

### OpenVG & Node

I'd heard about OpenVG before my Raspberry Pi arrived. At work we were using a similar bit of hardware, and achieving any sort of hardware-accelerated graphics was a massive bugbear. The RPi community is pretty amazing, and I enviously eyed the various graphics libraries that were springing up in my favourite languages. The [node-openvg-canvas](https://github.com/luismreis/node-openvg-canvas) project appealed especially, as it implements the HTML5 Canvas API in OpenVG. This means:

 * It's easy to grab demos from the web and try them out on the Pi with little-to-no conversion.
 * You can also use other libraries that *build on* the Canvas API, such as a charting library like [Flot](http://www.flotcharts.org/), theoretically even awesome libraries like [three.js](https://github.com/mrdoob/three.js/), which can render to Canvas (just one of its choices of renderers).

Step one for me was to grab the [first HTML5 canvas particle demo I could find online](http://thecodeplayer.com/walkthrough/make-a-particle-system-in-html5-canvas) to see how it ran with node-openvg-canvas.

It worked with minimal changes (my changes were backwards-compatible with the browser), but I couldn't add many particles to the scene before the framerate started to suck. My limit was a disappointing 20 particles. You can grab the code (and usage instructions) from [this gist](https://gist.github.com/nilliams/7213740#file-node-openvg-particles-js). Here's a video:

<iframe width="560" height="315" src="//www.youtube.com/embed/N8_oFIu5KwY" frameborder="0" allowfullscreen></iframe>


### OpenVG Testbed

The next thing to try was to go low-level (C) to get a feel for the performance I could expect, by using the raw OpenVG API. I came across the [OpenVG Testbed](https://github.com/ajstarks/openvg) project by [@ajstarks](https://twitter.com/ajstarks), he's blogged about it [here](http://mindchunk.blogspot.co.uk/2012/09/openvg-on-raspberry-pi.html). It's a set of examples and handy helper functions for interacting with OpenVG in C.

I wanted something a bit more 'particley' too, so I watched [this talk](http://www.youtube.com/watch?v=UCuVjPpExoQ&t=00h05m24s) (from 5m 24s) for a quick crash course in faking physics from Seb Lee Delisle, king of particle demos.

It was a pretty straightforward task, with the testbed providing some useful help. Performance wasn't massively improved over my Node efforts, I could get about 50 particles animated smoothly if I didn't care about monopolising ~45% of the CPU in the process.

I'm not comparing like for like demos, but my feeling here is that these sorts of naive implementations are going to be similarly performant using either approach (Node bindings or raw C API). Both are calling the same OpenVG functions under the hood, and Node isn't bringing much overhead when calling them in a single loop.

I suspect I'm missing a trick in taking advantage of some transform-related trickery, perhaps not. Material on the raw OpenVG APIs is frustratingly scarce.

So here's the result currently. Nothing mindblowing, but good fun and the fairly pleasing screensaver feel has given me ideas for further tinkering. (Code and usage instructions [here](https://gist.github.com/nilliams/7705819).

<iframe width="560" height="315" src="//www.youtube.com/embed/w6mHtbFF9hs" frameborder="0" allowfullscreen></iframe>

I did find applications where working with the C APIs was advantageous. One of my first experiments with the OpenVG Testbed was to create a scrolling 'ticker' line of text. The testbed allowed me to take advantage of lower-level OpenVG functions like `vgTranslate()`. The result is a nice, smooth scrolling line (at 1080p). There's a font conversion tool included in the testbed too, `font2openvg`, and I managed to convert a decent font over without much trouble. ([Video](http://www.youtube.com/watch?v=LJSKtFRr8i8)).

That's all for now. If you've managed to get more out of your Pi, give me a shout on [twitter](http://twitter.com/nickwuh).
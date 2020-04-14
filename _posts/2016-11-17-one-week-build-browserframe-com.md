---
layout: post
title: "One Week Build: BrowserFrame.com"
tags: [BrowserFrame, "JavaScript", "Vuejs", "Nodejs", "Web Development", "Startup"]
---

A one week build. A free tool that helps you wrap screenshots in different browser frames. Supports Chrome, Firefox, Safari, and more. Built with [node.js](https://nodejs.org/en/) and [vue.js](https://vuejs.org/), deployed on [zeit now](https://zeit.co/now).

![](/assets/img/one-week-build-browserframe-com/1_8R_LmrmgELAcfBUua4p8JQ.png)

---

## Problem

I use a lot of great tools for generating smartphone mockups and device art, like Android Device Art Generator, MockuPhone, Placeit, etc. However, I haven’t found a good tool for framing web apps / screenshots.

There are a couple of reasons why I want a tool for this (rather than just taking a screenshot of the app running on my computer). Here’s why:

**#1 — I want to remove clutter.** I don’t want to include my browser extensions, tabs, tab text, my profile name, bookmarks, etc.

![](/assets/img/one-week-build-browserframe-com/1BNrXJ8Ym4alOZNIFQBfddA.png)

**#2 — Generate cross-platform screenshots easily.** I constantly switch between Windows, Mac, and Linux depending on what I’m working on (right tool for the right job!).

**#3 — Sometimes I just want to showcase a portion of a page.** A good example of this is when you want to highlight a specific product feature, rather than taking a screenshot of the full page, so that you can zoom in on that particular feature.

**#4 —** **Photoshop templates aren’t good enough.** There are tons of them, and some look really good. But most aren’t easily resizeable and it’s just a hassle to have to do it that way.

---

## Solution

Simple problems should have simple solutions:

1. Take a screenshot

1. Easily upload the image

1. Select the desired browser frame

1. Download the final image

---

## Build

This was a simple build. But, to be honest, from start to finish it took more than one week. The “technical build” itself was definitely within one week — but I spent quite a lot of time diagnosing some interesting problems I ran into (more about that later).

### Backend

The stack and backend is very straight forward. I tried to put a lot of the logic in the frontend and keep the backend as simple as possible.

* **node.js & express** — quick to set up and get started

* **no database** — uploaded images are temporarily stored on disk

### Frontend

For this build I decided to use a frontend framework called [Vue.js](https://vuejs.org/). I’ve worked with it in the past and was pretty impressed. They recently released a new version of it so I decided to give that a try for this build.

One of the reasons why I really like Vue is that it’s so easy to get started with. Other frameworks like React and Angular are really powerful and popular, but also quite complex.

Vue is extremely quick to start using — especially for small simple components and pages. (Right tool for the right job people).

---

## Going Deep

There were a couple of things that made this build trickier than expected.

1. I ran into some very interesting problems with HTTP/2

2. I had to broaden my Photoshop skills

### 1. Troubleshooting HTTP/2

**zeit now**, which I use to deploy the app, uses the new HTTP/2 protocol by default. Essentially it’s the next generation of HTTP that aims to reduce latency.

I finished my built, published it, and all seemed well. Then I asked a friend to test it. He uploaded an image he had sitting on his desktop, and… “Internal Server Error”. It didn’t work.

After hours of debugging the problem, I realized that something was causing the image upload to become corrupt. Strangely, it wasn’t reproducible using all images, only some. Furthermore I couldn’t find any repeatable patterns across OS or browser.

In the end I believe the issue is related to how nginx handles multi-part form uploads in HTTP/2. Something is causing the data stream to become mangled. For those of you that are curious, you can read more about it here:

* [https://github.com/zeit/now/issues/110](https://github.com/zeit/now/issues/110)

* [https://github.com/pqvst/form-test](https://github.com/pqvst/form-test)

I worked around the problem by using the client-side FileReader API and then uploading images using ajax (thus avoiding a form submission). To be fair that was a better solution in any case, to keep the page responsive.

### 2. Mastering Slicing

One of the main tasks for this build was to create the artwork needed to actually produce the framed images. Obviously the frames can’t be static screenshots since the frame has to resize to fit the uploaded image.

I solved this by taking screenshots of the browsers and broke them down into different pieces (slices) consisting of non-stretchable pieces (corners) and stretchable pieces (edges).

![Slices in Photoshop](/assets/img/one-week-build-browserframe-com/1PcNbX2Lwqj4lPe85Xm6C-Q.png)*Slices in Photoshop*

For Windows it was pretty straight forward since there aren’t any rounded corners. So it was just a matter of cutting them into the different pieces.

![Chrome pieces for Windows](/assets/img/one-week-build-browserframe-com/10ubgZWVPPM37FwbypCoccw.png)*Chrome pieces for Windows*

However, on Mac, some browsers have rounded corners. Therefore I couldn’t just use the pieces directly since the uploaded screenshots didn’t have rounded corners. I solved this by creating corner masks that I applied to the uploaded screenshot so that they would match the browser.

![Corners masks for Chrome for Mac](/assets/img/one-week-build-browserframe-com/1kmMxK7MkvN55WEjtVagdLA.png)*Corners masks for Chrome for Mac*

A few hours later and I had 8 different browsers :)

```
Chrome (Windows & Mac)
Firefox (Windows & Mac)
Safari (Mac)
Edge (Windows)
IE (Windows)
Opera (Windows)
```

---

## Future Ideas

* Option to add padding to the image.
* Option to add a shadow (gives the image some nice depth)
* Customize dimensions

---

## Give it a try :)

[https://browserframe.com](https://browserframe.com)  
[https://www.producthunt.com/posts/browser-frame](https://www.producthunt.com/posts/browser-frame)

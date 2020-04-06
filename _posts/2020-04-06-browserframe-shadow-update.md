---
layout: post
title: "BrowserFrame: Shadow Update"
tags: [BrowserFrame, "JavaScript", "Web Development", "Web Design"]
---

Today [BrowserFrame.com](https://browserframe.com) gets a small update — It is now possible to add a shadow around the browser frame. Simply use the new `Shadown Amount` and `Shadow Color` options to adjust the shadow. To remove the shadow, simply clear the shadow options.

![](/assets/images/browserframe-shadow-update/demo.png)

The shadow amount and color can be customized using the new options.

![](/assets/images/browserframe-shadow-update/custom.png)

If the default shadow space is too large for your liking, you can apply a negative padding to reduce the extra space (but be careful not to clip the shadow as this would probably look ugly).

![](/assets/images/browserframe-shadow-update/padding.png)

---

## Technical Details

In my [previous blog post](https://pqvst.com/2019/06/14/one-week-build-browserframe-2-0/) I described the techniques I used to render browser frames using client-side using JavaScript and the canvas API. The reason I didn't add support for shadows in the first place was that I couldn't figure out a way to implement it using this method.

The main problem is that the shadow has to sit between the background color (if specified) and the browser frame). However, there is no `globalCompositeOperation` that would let me achieve this.

The workaround I use now is that the background (background color and shadow) and foreground (browser frame) are drawn in separate temporary canvases (off-screen).

```js
const backgroundCanvas = document.createElement('canvas');
const backgroundCtx = background.getContext('2d');
// draw background and shadow...

const foregroundCanvas = document.createElement('canvas');
const foregroundCtx = background.getContext('2d');
// draw browser frame...
```

These are then merged into the resulting canvas shown on the page. Thankfully this is very easy since the `drawImage` function accepts (among other things) canvas objects.

```js
ctx.drawImage(backgroundCanvas, 0, 0);
ctx.drawImage(foregroundCanvas, 0, 0);
```

---

## Give it a try!
[https://browserframe.com](https://browserframe.com)

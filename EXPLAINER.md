# Screen Enumeration

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays with differing properties.

This proposal gives developers access to a list of the available displays and the display properties of each display. It is foundational for some parts of the [Window Placement API proposal](https://github.com/spark008/window-placement) and can be used to enhance other parts.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
  * Move the speaker notes to a specific screen, not in fullscreen mode.
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
* **Video player that optimizes video specs/quality for each screen**
  * Video is optimized for the screen on which it is rendered.

## Goals / Non-goals

## Proposals

### **async** vs sync
One advantage of async APIs is that they are non-blocking.

### **`"display"`** vs `"screen"`
In OS APIs, a "display" represents a unit of rendering space (e.g. an external monitor) and the "screen" represents the singleton universe containing all the displays. Porting this terminology to Web APIs could make the Web APIs clearer to those who are already familiar with the OS APIs. The naming issue is complicated, however, by the various interpretations of the phrase "Web-exposed screen area" and the usage of the word "screen" in the existing `Screen` and `Window` interfaces.

The `Screen` interface uses the word "screen" to represent a unit of rendering space. According to the [spec](https://drafts.csswg.org/cssom-view/#web-exposed-screen-information), all screen properties return values that are relative to the area of the "output device", which in practice refers to the window's current monitor. For example, `window.screen.width` and `window.screen.height` will return different values if the window is moved between monitors with different dimensions. In addition, moving a window from the top-left corner of one monitor to the top-left corner of another monitor will not change the value of `window.screen.top` and `window.screen.left`, which are unstandardized but widely implemented. So introducing the OS terminology would require renaming "screen" to "display", and changing "screen" to refer to the sum of "displays".

The `Window` interface exposes a few screen-related properties of its own, `window.screenX` and `window.screenY`. These align more closely with the OS terminology, in that they return coordinates for a window relative to the entire screen space rather than relative to the display to which the window belongs. For example, given a window positioned to the right of the primary monitor, `window.screenX` returns a number larger than the width of the primary monitor. So using the OS API terminology would require no changes to the `Window` interface implementation.

### **`navigator`** vs `Window`/`WorkerGlobalScope`
somethin or other

```js
async () => {
  // Option 1 (Ideal): Add async `requestDisplays()` to an interface implemented
  //   by `Navigator`/`WorkerNavigator`
  const displaysV1 = await navigator.screen.requestDisplays();

  // Option 2: Add async `requestDisplays()` to `Navigator`/`WorkerNavigator`
  const displaysV2 = await navigator.requestDisplays();

  // Option 3: Add async `requestDisplays()` to `Window`/`WorkerGlobalScope`
  const displaysV3 = await requestDisplays();

  // Option 4: Add async `requestScreens()` to `Window`/`WorkerGlobalScope`
  const displaysV4 = await requestScreens();
}

// Option 5: Add `screens` property to `Window`/`WorkerGlobalScope`
const displaysV5 = screens;
```

## Privacy & Security

### Screen enumeration

Exposing the details of a user's multi-screen setup presents a fingerprinting concern.
In order to mitigate the amount of personally identifying information exposed, while maintaining the usefulness of the API, we can implement the following best practices:
* return the screens by increasing `width`
* limit the screen properties exposed to:
  * width
  * height
* limit the API to secure contexts

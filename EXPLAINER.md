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
At the OS level, a "display" represents a unit of rendering space (e.g. an external monitor) and the "screen" represents the singleton universe containing all the displays.
Porting this terminology to the Web could make the API clearer. There are discrepancies between the current implementations of the `Screen` and `Window` interfaces that complicate the naming issue. On one hand, the `Screen` interface already uses the word "screen" to represent a unit of rendering space, i.e. `window.screen` returns a different object if the window is moved to a different monitor. On the other hand, the `Window` interface returns coordinates for a window relative to the entire screen space rather than relative to the display to which the window belongs, e.g. for a window positioned on the rightmost monitor of a 2-monitor setup, `window.screenX` returns a number larger than the width of the leftmost monitor.

### **`navigator`** vs `Window`/`WorkerGlobalScope`
somethin or other

```js
async () => {
  // Option 1 (Ideal): Add async `requestDisplays()` to an interface implemented
  //   by `Navigator`/`WorkerNavigator`
  const displays1 = await navigator.screen.requestDisplays();

  // Option 2: Add async `requestDisplays()` to `Navigator`/`WorkerNavigator`
  const displays2 = await navigator.requestDisplays();

  // Option 3: Add async `requestDisplays()` to `Window`/`WorkerGlobalScope`
  const displays3 = await requestDisplays();

  // Option 4: Add async `requestScreens()` to `Window`/`WorkerGlobalScope`
  const displays4 = await requestScreens();
}

// Option 5: Add `screens` property to `Window`/`WorkerGlobalScope`
const displays5 = screens;
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

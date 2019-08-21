# Screen Enumeration

## Abstract

As it becomes more common to use more than one monitor, it becomes more
important to give Web developers the tools to make their applications perform
well across multiple displays with differing properties.

This proposal gives developers access to a list of the available displays and
the display properties of each display. It is foundational for some parts of
the [Window Placement API
proposal](https://github.com/spark008/window-placement) and can be used to
enhance other parts.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on the most
    appropriate screen for each window.
  * Move the speaker notes to a specific screen.
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
* **Video player that optimizes video specs/quality for each screen**
  * Video format is optimized for the screen on which it is rendered.

## Goals

### Current goals

* Enable access to available displays on both `Window` and service worker
  execution contexts
* Expose a subset of display properties needed to route content to the most
  suitable display

### Future goals

* Enable access to Chromecast displays

### Non-goals

* Expose an exhaustive set of display properties known to the OS
* Support screen change events (e.g. when a display is added/removed)

## Proposal

The leading option proposed here is to introduce a `ScreenManager` interface
that has an asynchronous `requestDisplays()` method gated behind a permission
prompt. If the user grants access to the displays connected to the device,
the method resolves to an array of `Display` objects, and rejects otherwise.
The `Display` object should contain a subset of the properties already
exposed in the `Screen` interface, as well as other properties needed to
support window placement features outlined in the [Window Placement API
explainer](https://github.com/spark008/window-placement/blob/master/EXPLAINER.md).
The interface should be implemented by both the `Navigator` and
`WorkerNavigator` interfaces, so that the API is exposed on both `Window` and
`ServiceWorker` execution contexts.

```js
async () => {
  // Get the displays connected to the device.
  //
  // Returns an Array of Display objects.
  //
  // Examples of other APIs with a similar shape:
  //   * `Navigator.bluetooth.requestDevice()`
  const displays = await navigator.screen.requestDisplays();

  for (const display in displays) {
    // Properties currently exposed in the Screen interface.
    console.log(display.colorDepth);     // 24
    console.log(display.width);          // 1680
    console.log(display.height);         // 1050
    console.log(display.availWidth);     // 1680
    console.log(display.availHeight);    // 1027

    // Properties in the Screen interface, but unstandardized.
    console.log(display.top);            // 0
    console.log(display.left);           // -1680
    console.log(display.availTop);       // 23
    console.log(display.availLeft);      // 0

    // Properties currently not Web-exposed.
    console.log(display.name);           // "DELL P2715Q"
    console.log(display.scalingFactor);  // 2
    console.log(display.isPrimary);      // true
    console.log(display.isInternal);     // true
  }
}
```

### **Synchronicity**: async vs sync

One advantage of asynchronous APIs is that they are non-blocking. Given the
privacy concerns with screen enumeration, it is possible that the API can
only expose screens for which the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing
any logic that does not depend on the result of the permission check, while
the user interacts with the display chooser UI.

### **Container class**: `Screen` vs `Display`

Some screen properties can already be found in the `Screen` interface, which is
synchronously accessible via `window.screen`. Extending this interface to
include the newly proposed properties reduces the complexity of screen-related
APIs as a whole, but poses a potential privacy concern by exposing all the
properties of the current screen without getting the user's permission.

### **Naming**: `"display"` vs `"screen"`

TODO: Revise this section.

In OS APIs, a "display" represents a unit of rendering space (e.g. an
external monitor) and the "screen" represents the singleton universe
containing all the displays. Porting this terminology to Web APIs could make
the Web APIs clearer to those who are already familiar with the OS APIs. The
naming issue is complicated, however, by the various interpretations of the
phrase "Web-exposed screen area" and the usage of the word "screen" in the
existing `Screen` and `Window` interfaces.

The Android API also uses the word "display" to reference a unit of rendering
space.

The `Screen` interface uses the word "screen" to represent a unit of
rendering space. The `width` and `height` properties return values that are
relative to the area of the "output device", which in practice refers to the
window's current monitor. The unstandardized `top` and `left` properties,
however, return values that are relative to the entire screen space.

The `Window` interface exposes a few screen-related properties of its own,
`window.screenX` and `window.screenY`. These align more closely with the OS
terminology, in that they return coordinates for a window relative to the
entire screen space rather than relative to the display to which the window
belongs. For example, given a window positioned to the right of the primary
monitor, `window.screenX` returns a number larger than the width of the
primary monitor. So using the OS API terminology would require no changes to
the `Window` interface implementation.

### **Scope**: `WindowOrWorkerGlobalScope` vs `Navigator`/`WorkerNavigator`

If we take inspiration from existing APIs with a similar shape, there are a
couple of places where the API could live.

1. The global scope, `Window`, is appealing as those familiar with the
`window.screen` API might anticipate finding multi-screen functionality in a
corresponding `window.screens` API. The `window` object currently contains a
sprawling mishmash of unrelated APIs, however, so tacking on additional
weight may contribute to the disorganization. In order to support the API in
service workers, `WorkerGlobalScope` would additionally need to implement the
API, so we'd likely add the API to the `WindowOrWorkerGlobalScope` mixin, which
is exposed on both `Window` and `WorkerGlobalScope` objects.

1. The `navigator` object nested beneath the global scope is an alternative that
would tuck the API into a less chaotic wing of the global scope. In order to
support the API in service workers, `WorkerNavigator` would additionally need to
implement the API.

## Alternative proposals

```js
async () => {
  // Option 1: Add async `requestDisplays()` to an interface implemented by
  //   `WindowOrWorkerGlobalScope`
  //
  // Examples of other APIs with a similar shape:
  //   * `WindowOrWorkerGlobalScope.caches.keys()`
  //   * `WindowOrWorkerGlobalScope.indexedDB.databases()`
  //
  // Note: The interface cannot be bound to a property named `screen` as the
  //   `window.screen` property already exists.
  const displaysV1 = await self.screens.requestDisplays();

  // Option 2: Add async `requestDisplays()` to `Navigator`/`WorkerNavigator`
  //
  // Examples of other APIs with a similar shape:
  //   * `Navigator.getVRDisplays()`
  const displaysV2 = await navigator.requestDisplays();

  // Option 3: Add async `requestDisplays()` to `WindowOrWorkerGlobalScope`
  const displaysV3 = await self.requestDisplays();

  // Option 4: Add async `requestScreens()` to `WindowOrWorkerGlobalScope`
  const displaysV4 = await self.requestScreens();

  // Option 5: Add `screens` property to `WindowOrWorkerGlobalScope`
  //
  // Examples of other APIs with a similar shape:
  //   * `Window.screen`
  const displaysV5 = self.screens;
}
```

## Privacy & Security

Exposing the details of a user's multi-screen setup presents a fingerprinting
concern. In order to mitigate the amount of personally identifying
information exposed, while maintaining the usefulness of the API, we can return
the screens ordered by a non-OS-differentiating property, like increasing
`width`.

To reduce the chance that the user's screen data gets compromised, we can also
limit the API to secure contexts.

To minimize the fingerprintable space, we can limit the set of screen
properties we expose to the bare minimum needed to support our use cases.

* New properties needed in a UI for the user to select on which display content
  should appear:
  * display name
* New properties needed to determine which display is the most appropriate for
  a given type of content:
  * resolution or scale factor
  * primary vs secondary display
  * built-in vs external display

To ensure that the user is aware of the data they are sharing and has control
over which displays the Web application can access, we propose implementing a
permission prompt. Calling `requestDisplays()` would prompt the user to select
which displays to share with the application.

A consequence of reusing the `Screen` interface to expose the properties listed
above is that the properties of the window's current screen are exposed without
getting the user's permission.
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

* Enable access to available displays on both `window` and service worker scopes
* Expose a subset of display properties needed to route content to the most
  suitable display

### Future goals

* Enable access to Chromecast displays

### Non-goals

* Expose an exhaustive set of display properties known to the OS

## Proposal

The leading option proposed here is to have `Navigator` and `WorkerNavigator`
implement a `NavigatorScreen` interface that has an asynchronous method for
getting the list of `Screen` objects to which the user has granted access.
Additionally, we propose adding a number of properties to the `Screen` interface
that will be essential to empowering the APIs that will consume this API.

```js
async () => {
  // Async method on interface implemented by `Navigator`/`WorkerNavigator`
  //
  // Examples of other APIs with a similar shape:
  //   * `Navigator.bluetooth.requestDevice()`
  const displays = await navigator.screen.requestDisplays();
}
```

### **Synchronicity**: async vs sync

One advantage of asynchronous APIs is that they are non-blocking. Given the
privacy concerns with screen enumeration, it is possible that the API can
only expose screens for which the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing
any logic that does not depend on the result of the permission check, while
the user interacts with the display chooser UI.

### **Naming**: `"display"` vs `"screen"`

In OS APIs, a "display" represents a unit of rendering space (e.g. an
external monitor) and the "screen" represents the singleton universe
containing all the displays. Porting this terminology to Web APIs could make
the Web APIs clearer to those who are already familiar with the OS APIs. The
naming issue is complicated, however, by the various interpretations of the
phrase "Web-exposed screen area" and the usage of the word "screen" in the
existing `Screen` and `Window` interfaces.

The `Screen` interface uses the word "screen" to represent a unit of
rendering space. According to the
[spec](https://drafts.csswg.org/cssom-view/#web-exposed-screen-information),
all screen properties return values that are relative to the area of the
"output device", which in practice refers to the window's current monitor.
For example, `window.screen.width` and `window.screen.height` will return
different values if the window is moved between monitors with different
dimensions. In addition, moving a window from the top-left corner of one
monitor to the top-left corner of another monitor will not change the value
of `window.screen.top` and `window.screen.left`, which are unstandardized but
widely implemented. So introducing the OS terminology would require renaming
"screen" to "display", and changing "screen" to refer to the sum of
"displays".

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

In addition, we can limit the set of screen properties we expose to the bare
minimum needed to support our use cases.

* New properties needed to determine which display is the most appropriate for
  a given type of content:
  * absolute coordinates in entire screen space (already accessible via
    `screenX`/`screenY` on `window`)
  * resolution
  * primary vs secondary display
* New properties needed in a UI for the user to select on which display content
  should appear:
  * display name

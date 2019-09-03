# Screen Enumeration

## Abstract

As it becomes more common to use more than one monitor, it becomes more
important to give Web developers the tools to make their applications perform
well across multiple displays with differing properties.

This proposal gives developers access to a list of the available displays and
the display properties of each display. It is foundational for some parts of the
[Window Placement API proposal](https://github.com/spark008/window-placement)
and can be used to enhance other parts.

## Use cases

* **Slide show presentation using multiple monitors**
  * Open the presentation, speaker notes, and presenter controls on the most
    appropriate monitor for each window.
  * Move the speaker notes to a specific monitor.
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple monitors.
* **Video player that optimizes video specs/quality for each monitor**
  * Video format is optimized for the monitor on which it is rendered.

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
prompt. If the user grants access to the displays connected to the device, the
method resolves to an array of `Display` objects, and rejects otherwise. The
`Display` object should contain a subset of the properties already exposed in
the `Screen` interface, as well as other properties needed to support window
placement features outlined in the [Window Placement API
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

  for (const display of displays) {
    // Properties currently exposed in the Screen interface.
    console.log(display.colorDepth);        // 24
    console.log(display.width);             // 1680
    console.log(display.height);            // 1050
    console.log(display.availWidth);        // 1680
    console.log(display.availHeight);       // 1027
    console.log(display.orientation.type);  // "landscape-primary"

    // Unstandardized properties in the Screen interface.
    console.log(display.left);              // -1680
    console.log(display.top);               // 0
    console.log(display.availLeft);         // 0
    console.log(display.availTop);          // 23

    // Properties currently exposed in the Window interface.
    // Exposed as Window.devicePixelRatio.
    console.log(display.scaleFactor);       // 2

    // New properties currently not Web-exposed.
    console.log(display.name);              // "DELL P2715Q"
    console.log(display.isPrimary);         // true
    console.log(display.isInternal);        // true
  }
}
```

### **Synchronicity**

One advantage of asynchronous APIs is that they are non-blocking. Given the
privacy concerns with screen enumeration, it is possible that the API can only
expose screen information if the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing any
logic that does not depend on the result of the permission check, while the user
interacts with the display chooser UI.

### **Container class**: `Display` (new) vs `Screen` (existing)

Some of the desired display properties already exist in the `Screen` interface.
Although extending this interface to include the remaining properties reduces
the surface area of screen-related APIs, it poses a potential privacy concern
since the properties of the window's current display would be exposed without
the user's permission via the existing synchronous `window.screen` API. Thus,
the preferred option is to create a new `Display` object, which duplicates some
properties but ensures that privacy-sensitive properties will always be exposed
asynchronously after checking for the user's permission.

### **Nomenclature and the Coordinate System**

This proposal uses the following multi-monitor terminology found in other APIs
(e.g. in desktop and mobile operating systems):

**Display**: A unit of rendering space, e.g. an external monitor.

**Screen**: The aggregate 2D space occupied by all the connected displays.

The top-left corner of the primary display defines the origin of the coordinate
system used to position each display. The existing screen-related Web APIs
already implement this layout, but do not use language that clearly defines
multi-monitor expectations.

### **Scope**: `WindowOrWorkerGlobalScope` vs `Navigator`/`WorkerNavigator`

If we take inspiration from existing similarly shaped Web APIs, there are a
couple of places where the API could live.

1. The global scope, `Window`, is appealing as those familiar with the
`window.screen` API might anticipate finding multi-display functionality in a
corresponding `window.screens` API. The `window` object currently contains a
sprawling mishmash of unrelated APIs, however, so tacking on additional weight
may contribute to the disorganization. In order to support the API in service
workers, we'd define the API in the `WindowOrWorkerGlobalScope` mixin, which is
implemented by both the `Window` and `WorkerGlobalScope` interfaces.

1. The `navigator` object nested beneath the global scope is an alternative that
would tuck the API into a less chaotic wing of the global scope. In order to
support the API in service workers, `WorkerNavigator` would additionally need to
implement the API.

## New Display Properties

Coordinates are defined in the screen coordinate system, i.e. the origin is at
the top-left corner of the primary display.

* **`Display.left`**: X-coordinate of this display's left edge.
  * Already exposed via `Window.screenLeft` and `Window.screenX`.
* **`Display.top`**: Y-coordinate of this display's top edge.
  * Already exposed via `Window.screenTop` and `Window.screenY`.
* **`Display.availLeft`**: Leftmost x-coordinate of this display that the window
may occupy.
  * Already exposed via `Screen.availLeft`.
* **`Display.availTop`**: Topmost y-coordinate of this display that the window
may occupy.
  * Already exposed via `Screen.availTop`.
* **`Display.name`**: A human-readable name that uniquely identifies this
display.
* **`Display.scalingFactor`**: The number of hardware pixels per CSS pixel.
  * Already exposed via `Window.devicePixelRatio`.
* **`Display.primary`**: Whether this display is the primary display.
* **`Display.internal`**: Whether this display is internal (built-in) or
external.

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

For in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/spark008/screen-enumeration/blob/master/security_and_privacy.md).

Exposing the details of a user's multi-monitor setup presents a fingerprinting
concern. In order to mitigate the amount of personally identifying information
exposed, while maintaining the usefulness of the API, we can return the displays
ordered by a non-OS-differentiating property, like increasing `width`.

To reduce the chance that the user's screen data gets compromised, we can also
limit the API to secure contexts.

To minimize the fingerprintable space, we can limit the set of display
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
permission prompt. Calling `requestDisplays()` for the first time would prompt
the user to select which displays to share with the application.


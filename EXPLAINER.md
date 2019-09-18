# Screen Enumeration

## Abstract

As computing devices more commonly support and use multiple displays, it becomes
more important to give web developers the right tools to present their content
across the set of connected displays.

Operating systems and their window managers almost univerally offer the ability
for users to connect multiple display devices and virtually arrange them in a 2D
plane, extending the overall screen space, which is useful in many scenarios.

The web platform currently exposes information through the
[`Screen` interface](https://developer.mozilla.org/en-US/docs/Web/API/Screen)
with details about the display device currently hosting the content window. This
and other properties of the
[`Window` interface](https://developer.mozilla.org/en-US/docs/Web/API/Window),
such as `window.screenX` and `window.screenX` surface some information about the
user's overall screen space when the content window is located on a secondary
display, but they provide an incomplete picture.

This proposal aims to give developers access to a list of the available displays
that comprise the overall screen space, as well as the essential properties of
each display. It is foundational for some parts of the
[Window Placement API proposal](https://github.com/spark008/window-placement).

## Use cases

* **Slide show presentation with a laptop screen and a projector**
  * Present slides on the projector, open speaker notes/controls on the laptop
* **Finance applications with multiple windows spread over multiple displays**
  * Launching a dashboard opens a set of windows across multiple displays
  * User interacts with dashboard controls to move windows between displays
* **Windows spanning multiple displays with different hardware properties**
  * Content (e.g. videos) can be optimized for the display where it is rendered

## Goals

### Current goals

* Enumerate available displays from `Window` and service worker execution
  contexts
* Expose display properties needed to show content on the most suitable display

### Future goals

* Surface events when displays are added/removed or when display properties
  change

### Non-goals

* Expose an exhaustive set of display properties known to the OS
* Enable access to remote displays (see the
  [Presentation API](https://www.w3.org/TR/presentation-api/)
  and the [Remote Playback API](https://www.w3.org/TR/remote-playback/))

## Proposal

The leading option proposed here is to introduce a `ScreenManager` interface
that has an asynchronous `getDisplays()` method, which may be gated behind a
permission prompt. Upon success, the method resolves to an array of `Display`
objects, and rejects otherwise. The `Display` object should contain properties
much like those already exposed in the `Screen` interface, and potentially other
properties needed to support window placement features outlined in the proposed
[Window Placement API](https://github.com/spark008/window-placement/blob/master/EXPLAINER.md).
The interface should be implemented by both the `Navigator` and
`WorkerNavigator` interfaces, so that the API is exposed on both `Window` and
`ServiceWorker` execution contexts.

```js
async () => {
  // Get the displays connected to the device.
  //
  // Returns an Array of Display objects on success.
  //
  // Examples of other APIs with a similar shape:
  //   * `Navigator.bluetooth.requestDevice()`
  const displays = await navigator.screen.getDisplays();

  for (const display of displays) {
    // Properties currently exposed in the Screen interface.
    console.log(display.width);             // 1680
    console.log(display.height);            // 1050
    console.log(display.availWidth);        // 1680
    console.log(display.availHeight);       // 1027
    console.log(display.orientation.type);  // "landscape-primary"
    console.log(display.orientation.angle); // 0
    console.log(display.colorDepth);        // 24

    // Unstandardized properties in the Screen interface.
    console.log(display.left);              // 1680
    console.log(display.top);               // 0
    console.log(display.availLeft);         // 0
    console.log(display.availTop);          // 23

    // Properties currently exposed in the Window interface.
    // Exposed as Window.devicePixelRatio.
    console.log(display.scaleFactor);       // 2

    // New properties currently not Web-exposed.
    console.log(display.primary);           // false
    console.log(display.internal);          // false
    console.log(display.name);              // "DELL P2715Q"
  }
}
```

### **Synchronicity**

One advantage of asynchronous APIs is that they are non-blocking. Given the
privacy concerns with screen enumeration, it is possible that the API can only
expose screen information if the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing any
logic that does not depend on the result of the permission check.

### **Container class**: `Display` (new) vs `Screen` (existing)

Some of the desired display properties already exist in the `Screen` interface.
Although extending this interface to include the remaining properties reduces
the surface area of screen-related APIs, it poses a potential privacy concern
since the properties of the window's current display would be exposed without
the user's permission via the existing synchronous `window.screen` API. Thus,
the preferred option is to create a new `Display` object, which duplicates some
properties but ensures that privacy-sensitive properties will always be exposed
asynchronously after potentially checking for the user's permission.

### **Nomenclature and the Coordinate System**

This proposal uses the following multi-display terminology found in other APIs
(e.g. in desktop and mobile operating systems):

**Display**: A physical unit of rendering space, e.g. an external monitor.

**Screen**: The aggregate 2D space occupied by all the connected displays.

The top-left corner of the primary display defines the origin of the coordinate
system used to position each display. The existing screen-related Web APIs
**already** implement this layout, but do not use language that clearly defines
multi-display expectations.

### **Scope**: `WindowOrWorkerGlobalScope` vs `Navigator`/`WorkerNavigator`

Taking inspiration from existing similarly shaped Web APIs, there are a couple
of places where the API could reasonably live.

1. The global scope, `Window`, is appealing as those familiar with the
`window.screen` API might anticipate finding multi-display functionality in a
corresponding `window.screens` API. The `window` object currently contains a
sprawling mishmash of unrelated APIs, however, so tacking on additional weight
may contribute to the disorganization. In order to support the API in service
workers, we'd define the API in the `WindowOrWorkerGlobalScope` mixin, which is
implemented by both the `Window` and `WorkerGlobalScope` interfaces.

2. The `navigator` object nested beneath the global scope is an alternative that
would tuck the API into a less chaotic wing of the global scope. In order to
support the API in service workers, `WorkerNavigator` would additionally need to
implement the API.

## `Display` Properties

Note: Coordinates are defined in the overall screen coordinate system, i.e. the
origin is at the top-left corner of the primary display.

Properties already exposed in the
[`Screen`](https://developer.mozilla.org/en-US/docs/Web/API/Screen) and
[`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) interfaces:
* **`Display.width`**: the width of the display, in pixels.
  * Already exposed via `Screen.width`.
* **`Display.height`**: the height of the display, in pixels.
  * Already exposed via `Screen.height`.
* **`Display.availWidth`**: the width of the display, in pixels, minus permanent
or semipermanent user interface features displayed by the operating system, such
as the Taskbar on Windows.
  * Already exposed via `Screen.availWidth`.
* **`Display.availHeight`**: the height of the display, in pixels, minus
permanent or semipermanent user interface features displayed by the operating
system, such as the Taskbar on Windows.
  * Already exposed via `Screen.availHeight`.
* **`Display.orientation`**: the
  [`ScreenOrientation`](https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation)
  instance associated with this display.
  * Already exposed via `Screen.orientation`.
* **`Display.colorDepth`**: the color depth of the display.
  * Already exposed via `Screen.colorDepth` and `Screen.pixelDepth`.
* **`Display.left`**: the distance in pixels from the left side of the primary
display to the left side of this display.
  * Unstandardized; already exposed via `Screen.left`.
* **`Display.top`**:  the distance in pixels from the top of the primary display
to the top of this display.
  * Unstandardized; already exposed via `Screen.top`.
* **`Display.availLeft`**: the x-coordinate of the first pixel that is not
allocated to permanent or semipermanent user interface features on this display.
  * Unstandardized; already exposed via `Screen.availLeft`.
* **`Display.availTop`**: the y-coordinate of the first pixel that is not
allocated to permanent or semipermanent user interface features on this display.
  * Unstandardized; already exposed via `Screen.availTop`.
* **`Display.scaleFactor`**: the ratio between physical pixels and device
independent pixels in the current display.
  * Unstandardized; already exposed as `Window.devicePixelRatio`

New properties currently not Web-exposed that may be important to priorize.
* **`Display.primary`**: True if this display is the primary display.
  * May be useful for determining the placement of prominent dashboard windows.
  * Can be inferred from information already exposed; i.e. `Screen.left` and
    `Screen.top` are both zero only on the primary display.
* **`Display.internal`**: True if this display is internal (built-in).
  * May be useful for showing slides on external displays (projector) and notes
    on internal displays (laptop screen).
* **`Display.name`**: A human-readable name that identifies this display.
  * May be useful for prompting/notifying users about window placement actions.
  * May be useful for persisting placement info specific to certain displays.

New properties currently not Web-exposed that may be worth considering.
* **`Display.id`**: The Extended Display Identification Data or another ID.
  * May be useful for persisting placement info specific to certain displays.
* **`Display.touchSupport`**: True if the display supports touch input.
  * May be useful for presenting touch-specific UI layouts.
* **`Display.accelerometer`**: True if the display has an accelerometer.
  * May be useful for showing immersive controls (e.g. game steering wheel).
* **`Display.dpi`**: The number of pixels per inch.
  * May be useful for presenting content with tailored physical scale factors.
* **`Display.interlaced`**: True if the display's mode is interlaced.
  * May be useful for adapting content presentation for some display technology.
* **`Display.refreshRate`**: The display's refresh rate in hertz.
  * May be useful for adapting content presentation for some display technology.
* **`Display.overscan`**: The display's insets within its screen's bounds.
  * May be useful for adapting content presentation for some display technology.
* **`Display.hidden`**: True if the display is not visible (e.g. closed laptop).
  * May be useful for recognzing when displays may be active but not visible.

## Alternative proposals

```js
async () => {
  // Option 1: Add async `getDisplays()` to an interface implemented by
  //   `WindowOrWorkerGlobalScope`
  //
  // Examples of other APIs with a similar shape:
  //   * `WindowOrWorkerGlobalScope.caches.keys()`
  //   * `WindowOrWorkerGlobalScope.indexedDB.databases()`
  //
  // Note: The interface cannot be bound to a property named `screen` as the
  //   `window.screen` property already exists.
  const displaysV1 = await self.screens.getDisplays();

  // Option 2: Add async `getDisplays()` to `Navigator`/`WorkerNavigator`
  //
  // Examples of other APIs with a similar shape:
  //   * `Navigator.getVRDisplays()`
  const displaysV2 = await navigator.getDisplays();

  // Option 3: Add async `getDisplays()` to `WindowOrWorkerGlobalScope`
  const displaysV3 = await self.getDisplays();

  // Option 4: Add async `getScreens()` to `WindowOrWorkerGlobalScope`
  const displaysV4 = await self.getScreens();

  // Option 5: Add `screens` property to `WindowOrWorkerGlobalScope`
  //
  // Examples of other APIs with a similar shape:
  //   * `Window.screen`
  const displaysV5 = self.screens;
}
```

## Open Questions

* How would `Display` objects correlate with the existing `Screen` API?
  * How would one find the `Display` object corresponds to the local `Screen`?
    * Should there be a `window.display` alongside of `window.screen`?
* Which properties should `Display` include? There are many potential properties
and alternative ways to convey the same information. Care should be taken to
choose a concise and valuable set of properties.
  * Should `Display` be a strict superset or match of the `Screen` object?

## Privacy & Security

For in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/spark008/screen-enumeration/blob/master/security_and_privacy.md).

Exposing the details of a user's multi-display setup presents a fingerprinting
concern. In order to mitigate the amount of personally identifying information
exposed, while maintaining the usefulness of the API, implementrs can return the
displays ordered by a non-OS-differentiating property, like increasing `width`.

It should be noted that some information is already exposed through exisisting
API surfaces. Beyond explicitly providing information about the single display
currently hosting the content window, the `Screen` interface's `left` and `top`
values and the `Window` interface's `screenX` and `screenY` values are generally
given in the overall screen space coordinate system. This means that sites can
infer whether the current display is primary or not, and potentially infer at
least one dimension of the primary display, or the potential presence of more
than two displays, given the standard resolutions offered by display devices.

To minimize the fingerprintable space, it's prudent to limit the set of display
properties exposed to the minimum needed to support common use cases.

To reduce the chance that the user's screen data gets compromised, implementers
could limit the API to secure contexts. To ensure that the user is aware of the
data they are sharing and has control over which displays a site can access,
implementers could gate the success of enumeration upon the granting of explicit
permission through a prompt. Calling `getDisplays()` for the first time could
prompt the user to select which displays (if any) to share with the site.

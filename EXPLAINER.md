# Screen Enumeration

## Abstract

Operating systems and their window managers almost universally offer the ability
for users to connect multiple physical displays to a single computing device and
virtually arrange them in a 2D plane, extending the overall visual workspace.

The web platform currently offers some information through the [`Screen`][1]
interface, but this only provides details about the physical display currently
hosting the associated content [`Window`][2].

As multi-display computing becomes more a common and critical part of user
experiences, it becomes more important to give web developers information and
tools to leverage that expanded visual environment.

This proposal aims to extend the web platform, giving developers information
about the set of connected physical displays, and considers additional display
properties that may be useful beyond the current [`Screen`][1] interface.

This proposal is foundational for the [Window Placement API proposal][3], which
is also critical for some of the use cases explored in this document.

## Use cases

* **Slide show presentation with a laptop screen and a projector**
  * Present slides on the projector, open speaker notes/controls on the laptop
* **Finance applications with multiple windows spread over multiple displays**
  * Launch a dashboard that opens a set of windows across multiple displays
  * User interacts with dashboard controls to move windows between displays
* **Windows spanning multiple displays with different hardware properties**
  * Content can be optimized for the display hardware where it is rendered

## Goals

### Current goals

* Enumerate available displays from relevant execution contexts
* Expose properties needed to show content on the most suitable display

### Future goals

* Surface events when the set of displays or their properties change

### Non-goals

* Expose an exhaustive set of display properties known to the OS
* Enable access to remote displays (see the
  [Presentation](https://www.w3.org/TR/presentation-api/)
  and [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs)

## Proposal

The leading option proposed here is to introduce a new `ScreenManager` interface
with a `getScreens()` method. The method resolves to an array of [`Screen`][1]
objects on success, and rejects otherwise. The interface should be implemented
by both the `Navigator` and `WorkerNavigator` interfaces, so that the API is
exposed on both `Window` and `ServiceWorker` execution contexts.

Additionally, this proposal would introduce new properties to the [`Screen`][1]
interface, providing information to help optimize content presentation.

```js
async () => {
  // NEW: Returns an Array of Screen objects connected to the device on success.
  const screens = await navigator.screen.getScreens();

  for (const screen of screens) {
    // Standardized properties in the Screen interface.
    console.log(screen.width);             // 1680
    console.log(screen.height);            // 1050
    console.log(screen.availWidth);        // 1680
    console.log(screen.availHeight);       // 1027
    console.log(screen.orientation.type);  // "landscape-primary"
    console.log(screen.orientation.angle); // 0
    console.log(screen.colorDepth);        // 24

    // Unstandardized properties in the Screen interface.
    console.log(screen.left);              // 1680
    console.log(screen.top);               // 0
    console.log(screen.availLeft);         // 0
    console.log(screen.availTop);          // 23

    // NEW: Currently exposed as Window.devicePixelRatio.
    console.log(screen.devicePixelRatio);  // 2

    // NEW: Properties currently not Web-exposed.
    console.log(screen.internal);          // false
    console.log(screen.primary);           // false (discernable from top/left)
    console.log(screen.name);              // "DELL P2715Q"
  }
}
```

Inspiration for the shape of this API comes from similar APIs, such as
[`Bluetooth.requestDevice()`](https://developer.mozilla.org/en-US/docs/Web/API/Bluetooth/requestDevice).
Using an interface facilitates encapsulation of future screen-related APIs, e.g.
managing event handling for when the set of screens or their properties change.

### **Synchronicity**

One advantage of asynchronous APIs is that they are non-blocking. With potential
privacy concerns with screen enumeration, it is possible that the API should
only expose screen information if the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing any
logic that does not depend on the result of the permission check.

Additionally, system window managers themselves often provide asynchronous APIs,
and this pattern would allow implementers to gather the latest available
information without blocking other application processing or requiring a cache.

### **Nomenclature**

Existing native APIs use a variety of nomenclatures to describe the distinction
between physical display devices and the overall space composed by their virtual
arrangement. As the web platform already uses the [`Screen`][1] interface to
describe a single physical unit of rendering space, it seems natural to follow
this convention and work in terms of a multi-`Screen` display environment.

### **Multi-Screen Coordinates**

By common convention, the top-left corner of the system's primary display
defines the origin of the coordinate system used to position other displays
in a two-dimensional plane, relative to the primary display.

Although it is not standardized, the [`Screen`][1] interface already follows
this same pattern for multi-screen environments in practice. The unstandardized
properties `Top`, `Left`, `availTop`, and `availLeft` match the system's virtual
arrangement of separate physical displays in practice. So, the `Screen` object
for the primary display has `Top` and/or `Left` values of zero, while `Screen`
objects for secondary displays have non-zero `Top` and/or `Left` values,
denoting their placement relative to the primary display.

Unstandardized aspects of the [`Window`][2] interface follow the same pattern.
The [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenX)
and [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenY)
properties are given relative to the origin of the primary display, which is not
necessarily the host of the content `Window`. Further, parameters passed into
[`moveTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo) are
taken to be in the same coordinate space, relative to the primary display.

It may be beneficial to actually standardize this pattern, or to provide some
non-normative notes encouraging implementers to follow this common convention.

### **Scope**: `Navigator`/`WorkerNavigator` vs `WindowOrWorkerGlobalScope`

Taking inspiration from existing similarly shaped Web APIs, there are a couple
of places where the API could reasonably live.

1. The [`Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
object nested beneath the global scope may be a good potential location for this
API, as the set of connected displays could be reasonably considered an aspect
of the user agent's environment. In order to support the API in service workers,
[`WorkerNavigator`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerNavigator)
would also need to implement the API.

2. The global scope, `Window`, is appealing as those familiar with the
`window.screen` API might anticipate finding multi-display functionality in a
corresponding `window.screens` API. However, the `Window` object currently
contains a sprawling mishmash of unrelated APIs, so tacking on additional weight
may contribute to the disorganization. In order to support the API in service
workers, we'd define the API in the `WindowOrWorkerGlobalScope` mixin, which is
implemented by both the `Window` and `WorkerGlobalScope` interfaces.

## New [`Screen`][1] Properties

The [`Screen`][1] interface supplies a fairly comprehensive set of display
properties, but there are use cases for additional properties, considered below.

New properties already Web-exposed in another manner:
* **`Screen.devicePixelRatio`**: the ratio of the resolution in physical pixels
to the resolution in CSS pixels on this screen.
  * Unstandardized; already exposed as `Window.devicePixelRatio`
  * May be useful for customizing the appearance of content when the hosting
    window spans across multiple displays with different pixel ratios.

New properties currently not Web-exposed that may be important to prioritize.
* **`Screen.internal`**: True if this display is internal (built-in).
  * May be useful for showing slideshows on external displays (projector) and
    controls/notes on internal displays (laptop screen).
* **`Screen.primary`**: True if this display is the primary display.
  * May be useful for determining the placement of prominent dashboard windows.
  * Can be inferred from unstandardized properties already exposed; i.e.
    `Screen.left` and `Screen.top` are both zero only on the primary display.
* **`Screen.name`**: A human-readable name that identifies this display.
  * May be useful for prompting/notifying users about window placement actions.
  * May be useful for persisting window placements for certain displays.
  * The user agent could use basic names, like "Display 1" in place of more
    sensitive device-specific names, reducing the fingerprintable surface.

New properties currently not Web-exposed that may be worth considering.
* **`Screen.id`**: The Extended Display Identification Data or another ID.
  * May be useful for persisting window placements for certain displays.
  * The user agent could map sensitive EDIDs to a index for the device-unique or
    user-unique set of known displays, reducing the fingerprintable surface.
* **`Screen.touchSupport`**: True if the display supports touch input.
  * May be useful for presenting touch-specific UI layouts.
* **`Screen.accelerometer`**: True if the display has an accelerometer.
  * May be useful for showing immersive controls (e.g. game steering wheel).
* **`Screen.dpi`**: The number of pixels per inch.
  * May be useful for presenting content with tailored physical scale factors.
* **`Screen.interlaced`**: True if the display's mode is interlaced.
  * May be useful for adapting content presentation for some display technology.
* **`Screen.refreshRate`**: The display's refresh rate in hertz.
  * May be useful for adapting content presentation for some display technology.
* **`Screen.overscan`**: The display's insets within its screen's bounds.
  * May be useful for adapting content presentation for some display technology.
* **`Screen.hidden`**: True if the display is not visible (e.g. closed laptop).
  * May be useful for recognizing when displays may be active but not visible.

## Alternative proposals

### Alternative names and locations for getScreens()

Besides the leading proposal for `Navigator` and `WorkerNavigator` to implement
a new `ScreenManager` interface with a `getScreens()` method, some alternative
names and locations for this method are listed below.

```js
async () => {
  // 1: `ScreenManager` interface on `WindowOrWorkerGlobalScope`, like:
  //   * `WindowOrWorkerGlobalScope.caches.keys()`
  //   * `WindowOrWorkerGlobalScope.indexedDB.databases()`
  // Note: This cannot use `screen`, per the existing `window.screen` property.
  const screensV1 = await self.screens.getScreens();

  // 2: 'getScreens()' directly from `Navigator`/`WorkerNavigator`, like:
  //   * `Navigator.getVRDisplays()`
  // Note: This inhibits encapsulation of related screen event APIs later.
  const screensV2 = await navigator.getScreens();

  // 3: 'getScreens()' directly from `WindowOrWorkerGlobalScope`.
  // Note: This inhibits encapsulation of related screen event APIs later.
  const screensV3 = await self.getScreens();

  // Alternative names:
  foo.getDisplays();
  foo.requestScreens();
  foo.requestDisplays();
}
```

### Alternative synchronous access pattern

The leading proposal is an asynchronous interface. Alternatively, a synchronous
API would match the existing `Window.screen` API, but may require implementers
to cache system information that would not otherwise be required. It would also
prevent access control using a permission model, or require implementers to stop
script execution while the permission model is queried or the user is prompted.

```js
  // Add a property on `WindowOrWorkerGlobalScope`, like `Window.screen`
  const screensV5 = self.screens;
```

### Alternative property access or a new `Display` container class

The existing [`Screen`][1] object seems like a natural and appropriate container
for conveying information about the connected display devices, but using the
`Screen` interface to convey **new** properties poses a potential privacy
concern with regard to fingerprinting, since the properties of the window's
current display would be exposed synchronously by the `window.screen` API. User
agents would not be able to gate access to these new properties on an
asynchronously-queried permission model.

Alternatively, the new properties could be accessed by new asynchronous methods
on the `Screen` or `ScreenManager` interfaces, or resolve to undefined if access
has not been granted:

```js
  // 1: Add an asynchronous method on `Screen`:
  const value = window.screen.getNewProperty();

  // 2: Add an asynchronous method on `ScreenManager`:
  const value = navigator.screen.getNewPropertyForScreen(window.screen);

  // 3: Properties are undefined before access is granted.
  console.log(screen.newProperty);  // Resolves to `undefined`.
  navigator.screen.getScreens();    // Requests access, which is granted.
  console.log(screen.newProperty);  // Resolves to the actual value.
```

Yet another alternative is introducing a new `Display` interface as a container
class that supercedes or parallels the `Screen` interface. This would duplicate
some properties but ensure that new, potentially privacy-sensitive properties
would only be exposed asynchronously, after potentially checking for permission.
This may cause some confusion or difficulty correlating the existing `Screen`
object with a new given `Display` object.

### Using `Screen` to represent the entire screen space.

Representing the entire combined screen space with the existing `Screen`
interface is inadvisable, as it would come with many complications, for example:
* The union of separate physical display bounds may be an irregular shape,
  comprised of rectangles with different sizes and un-aligned positions. This
  cannot be adequately represented by the current `Screen` interface.
* The set of connected physical displays may have different `Screen` properties.

## Privacy & Security

For an in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/spark008/screen-enumeration/blob/master/security_and_privacy.md).

Exposing the details of a user's multi-screen setup presents a fingerprinting
concern. To minimize the fingerprintable space, it's prudent to limit the set of
display properties exposed to the minimum needed to support common use cases.

It should be noted that some information is already exposed through existing API
surfaces. Beyond explicitly providing information about the single display
currently hosting the content window, the `Screen` interface's `left` and `top`
values and the `Window` interface's `screenX` and `screenY` values are generally
given in the overall screen space coordinate system. This means that sites can
already **possibly** infer whether the current display is primary or not;
placement of the current secondary display relative to the primary display;
dimensions of the primary display when content is on a secondary display, and
perhaps more, given the standard resolutions offered by display devices.

It is also currently possible for some sites to brute-force the detection of
additional displays on some user agent implementations by moving a popup window
to coordinates outside the current screen's bounds, and detecting the resulting
position and `Screen` object available to the window. See more thoughts around
this in the [Window Placement API proposal][3].

To reduce the chance that the user's screen data gets compromised, implementers
could limit the API to secure contexts. To ensure that the user is aware of the
data they are sharing and has control over which displays a site can access,
implementers could gate the success of enumeration upon the granting of explicit
permission through a prompt. Calling `getScreens()` for the first time could
prompt the user to select which screens (if any) to share with the site.

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Screen
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[3]: https://github.com/spark008/window-placement

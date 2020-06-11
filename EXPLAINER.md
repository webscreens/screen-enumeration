# Screen Enumeration on the web

## Abstract

Operating systems and their window managers almost universally offer the ability
for users to connect multiple physical displays to a single computing device and
virtually arrange them in a 2D plane, extending the overall visual workspace.

The web platform currently offers some information through the [`Screen`][1]
interface, but this only provides details about the physical display currently
hosting the associated content [`Window`][2].

As multi-display computing becomes a more common and critical part of user
experiences, it becomes more important to give web developers information and
tools to leverage that expanded visual environment.

This proposal aims to extend the web platform, giving developers information
about the set of connected physical displays, and considers additional display
properties that may be useful beyond the current [`Screen`][1] interface.

This proposal is foundational for the [Window Placement API proposal][3], which
is also critical for some of the use cases explored in this document.

## Use cases

This API exposes information useful to applications that wish to use multiple
screens to show windows:
* Present slides on a projector, open speaker notes on the laptop's screen
* Launch and manage a dashboard of financial windows across multiple monitors
* Open windows to view medical images (eg. x-rays) on the appropriate screens
* Creativity apps showing pallete/preview windows on multiple screens
* Optimize content when a window spans multiple screens with varying properties

See the [Window Placement API][3] for proposed functionality that would make use
of information exposed by this API.

## Goals

### Current goals

* Enumerate available displays from relevant execution contexts
* Standardize properties needed to show content on the most suitable display
* Surface events when the set of displays or their properties change

### Non-goals

* Expose an extraneous or especially identifying information (eg. display EDID)
* Expose APIs to control the configuration of display devices.
* Enumerate remote displays connected to other devices
  * See the [Presentation](https://www.w3.org/TR/presentation-api/) and
    [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs

## Proposal

The leading option proposed here is to introduce a `getScreens()` method, which
resolves to an array of [`Screen`][1] objects on success, and rejects otherwise.
The method could be implemented on the `Window` interface, making multi-screen
info accessible to execution contexts using the existing single-screen
`Window.screen` attribute.

Additionally, this proposal would introduce new properties to the [`Screen`][1]
interface, providing information to help optimize content presentation.

```js
async () => {
  // NEW: Returns an Array of Screen objects connected to the device on success.
  const screens = await window.getScreens();

  for (const screen of screens) {
    // Properties specified in https://www.w3.org/TR/cssom-view/#screen
    console.log(screen.width);         // 1680
    console.log(screen.height);        // 1050
    console.log(screen.availWidth);    // 1680
    console.log(screen.availHeight);   // 1027
    console.log(screen.colorDepth);    // 24
    console.log(screen.pixelDepth);    // 24

    // Properties specified in https://www.w3.org/TR/screen-orientation
    console.log(screen.orientation);   // { type: "landscape-primary", ... }

    // NEW: Properties implemented by some browsers that should be standardized.
    // See MDN docs: https://developer.mozilla.org/en-US/docs/Web/API/Screen
    console.log(screen.left);          // 1680
    console.log(screen.top);           // 0
    console.log(screen.availLeft);     // 0
    console.log(screen.availTop);      // 23

    // NEW: Properties proposed here that should be standardized.
    console.log(screen.scaleFactor);   // 2
    console.log(screen.internal);      // false
    console.log(screen.primary);       // false
    console.log(screen.id);            // 1
    console.log(screen.touchSupport);  // false
  }
}
```

This proposal also aims to introduce a `screenschange` event, fired when the set
of screens or their properties change. The event would be available on the
`Window` and `WorkerGlobalScope` objects, which both implement `EventTarget`.

```js
self.addEventListener('screenschange', function(event) {
  const biggestScreen = getBiggestScreen(event.screens);
  if (window.screen != biggestScreen)
    informUserOfAvailableScreen(biggestScreen);
});
```

Inspiration for the shape of this API comes from the existing `Window.screen`
attribute location, with expanded access to worker contexts. Similar APIs exist,
like `navigator.languages` and `self.addEventListener("onlanguagechange", ...)`.

See the Alternative Proposals section for some other possible API shapes.

### Synchronicity

One advantage of asynchronous APIs is that they are non-blocking. With potential
privacy concerns with screen enumeration, it is possible that the API should
only expose screen information if the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing any
logic that does not depend on the result of the permission check.

Additionally, system window managers themselves often provide asynchronous APIs,
and this pattern would allow implementers to gather the latest available
information without blocking other application processing or requiring a cache.

### Nomenclature

Existing native APIs use a variety of nomenclatures to describe the distinction
between physical display devices and the overall space composed by their virtual
arrangement. As the web platform already uses the [`Screen`][1] interface to
describe a single physical unit of rendering space, it seems natural to follow
this convention and work in terms of a multi-`Screen` display environment.

### Multi-Screen Coordinates

By common convention, the top-left corner of the system's primary display
defines the origin of the coordinate system used to position other displays
in a two-dimensional plane, relative to the primary display.

Although it is not standardized, the [`Screen`][1] interface already follows
this same pattern for multi-screen environments in practice. The unstandardized
properties `top`, `left`, `availTop`, and `availLeft` match the system's virtual
arrangement of separate physical displays in practice. So, the `Screen` object
for the primary display has `top` and/or `left` values of zero, while `Screen`
objects for secondary displays have non-zero `top` and/or `left` values,
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

### Scope: `Window`, `WindowOrWorkerGlobalScope`, `Navigator`/`WorkerNavigator`

Taking inspiration from existing Web APIs, there are a few places where the
proposed API could reasonably live.

1. In the `Window` interface, making multi-screen info accessible to execution
contexts using the existing single-screen `window.screen` attribute. Those
familiar with `window.screen` might anticipate a similar access pattern for
information about multi-screen environments. It should be noted that `Window`
already hosts many attributes, functions and interfaces, so care should be taken
not to overburden this surface.

2. The `WindowOrWorkerGlobalScope` mixin, implemented by `Window` and
`WorkerGlobalScope` interfaces, could also be an intuitive host this API. This
offers access to both `Window` and `Worker` execution contexts, extending the
suggestion above with additional access for service workers, which may be
valuable for certain aspects of the the [Window Placement API proposal][3].

2. The [`Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
object nested beneath the global scope may be a good potential location for this
API, as the set of connected displays could be reasonably considered an aspect
of the user agent's environment. In order to support the API in service workers,
[`WorkerNavigator`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerNavigator)
would also need to implement the API.

## New [`Screen`][1] Properties

The [`Screen`][1] interface supplies a fairly comprehensive set of display
properties, but there are use cases for additional properties, considered below.

`Screen` properties already implemented by some browsers, and which should be
standardized; see [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Screen).
These properties are useful for understanding the display layout relative to
content bounds and for the [Window Placement API proposal][3].
* `Screen.left`: The distance from the left side of the primary display to the
  left side of this display.
* `Screen.top`: The distance from the top of the primary display to the top of
  this display.
* `Screen.availLeft`: The distance from the left side of the primary display to
  the left side of the region available for windows on this display.
* `Screen.availTop`: The distance from the top of the primary display to the top
  of the region available for windows on this display.

New `Screen` properties with compelling use cases.
* `Screen.internal`: True if this display is internal (built-in).
  * Useful for showing slideshows on external displays (projector) and controls
    or notes on internal displays (laptop screen).
* `Screen.primary`: True if this display is the primary display.
  * Useful for determining the placement of prominent dashboard windows.
  * Can be inferred from unstandardized properties already exposed; i.e.
    `Screen.left` and `Screen.top` are both zero only on the primary display.
* `Screen.scaleFactor`: The ratio between physical pixels and device independent
  pixels for this display.
  * Useful for customizing the appearance of content when the hosting
    window spans across multiple displays with different pixel ratios.
  * TBD: How to effectively pair this with `window.devicePixelRatio`?
* `Screen.id`: A temporary, generated per-origin unique ID; resets when cookies
    are deleted.
  * Useful for persisting window placements for certain displays.
  * Useful for prompting/notifying users about window placement actions.
  * Useful for notifying users about screen configuration changes.
* `Screen.touchSupport`: True if the display supports touch input.
  * Useful for showing controls on the touch-enabled display in a meeting room.

New `Screen` properties to consider as use cases arise.
* `Screen.accelerometer`: True if the display has an accelerometer.
  * May be useful for showing immersive controls (e.g. game steering wheel).
* `Screen.dpi`: The display density as the number of pixels per inch.
  * May be useful for presenting content with tailored physical scale factors.
* `Screen.subpixelOrder`: The order/orientation of this display's subpixels.
  * May be useful for adapting content presentation for some display technology.
* `Screen.interlaced`: True if the display's mode is interlaced.
  * May be useful for adapting content presentation for some display technology.
* `Screen.refreshRate`: The display's refresh rate in hertz.
  * May be useful for adapting content presentation for some display technology.
* `Screen.overscan`: The display's insets within its screen's bounds.
  * May be useful for adapting content presentation for some display technology.
* `Screen.hidden`: True if the display is not visible (e.g. closed laptop).
  * May be useful for recognizing when displays may be active but not visible.
* `Screen.mirrored`: True if the display is mirrored to another display.
  * May be useful for recognizing when a laptop is mirrored to a projector, etc.

### Changes to `colorDepth` and `pixelDepth`

The [W3C Working Draft](https://www.w3.org/TR/cssom-view/#dom-screen-colordepth)
states that `Screen.colorDepth` and `Screen.pixelDepth` "must return 24" and
even explains that these "attributes are useless", but the latest
[Editor’s Draft](https://drafts.csswg.org/cssom-view/#dom-screen-colordepth)
provides a more useful specification for these values. There is a clear signal
from developers that having meaningful and accurate accurate values for these
properties is useful for selecting the optimal display to present medical and
creative content.

### Handling blocked permissions

It could be reasonable for getScreens() to handle a blocked permission or
similar failure modes in one of several ways:
* Reject the promise and throw an exception
* Fulfill the promise with an empty array or dictionary
* Fulfill the promise with an array only containing the existing window.screen

### Requests for limited information

It may be beneficial to extend the proposed API with a mechanism to request
more limited or granular multi-screen information. This may allow web developers
and user agents to cooperatively request and provide information required for
specific use cases, proactively reducing the fingerprintable information shared,
and potentially allowing user agents to expose more limited information without
explicit user permission prompts (eg. a single multi-screen boolean flag).

One possible approach is that getScreens() could request everything by default,
and take an optional parameter to request limited information. Partial results
could be returned as a Screen array with only the requested values populated, or
with dictionaries of named values including the screen array.

```js
// Request a single bit answering the question: are multiple screens available?
// This informs the value of additional information requests (and user prompts).
let screen_info = await getScreens(['multiScreen']);
if (!screen_info.multiScreen)
  return;
// Request the number of connected screens, either returning an array of 'empty'
// Screen objects with undefined property values, or as a named member of a
// returned dictionary, eg: { multi-screen: true, count: 2, ... }.
screen_info = await getScreens(['count']);
if (screen_info.count <= 1)  // OR: |if (screen_info.length <= 1)|
  return;

// An empty Screen object may suffice for some proposed Window Placement uses.
document.body.requestFullscreen(screens[1]);

// OR: call getScreens() again to request additional information, with only the
// requested information available via corresponding `Screen` attributes.
screen_info = await getScreens(['bounds', 'colorDepth']);
// Use bounds and colorDepth to determine the appropriate display.
document.body.requestFullscreen(
    (screen_info[1].width > screen_info[0].width ||
      screen_info[1].colorDepth > screen_info[0].colorDepth)
    ? screen_info[1] : screen_info[0]);
}
```

### Compatibility with the existing window.screen object

This proposal aims to provide a high degree of compatibility between objects
returned by the getScreens() API and the existing synchronously-accessed
`window.screen` object. Ideally, sites should be able to compare Screens objects
without concern for their origin, enabling a very basic and useful check like:

```js
// Find a Screen from getScreens that is not the current screen.
const otherScreen = (await getScreens()).find((s)=>{return s != window.screen;});
```

There are some options around exposing new properties for each access pattern:
* New properties are only exposed on getScreens() objects
  * window.screen yields undefined values for new properties
  * Comparison may just regard underlying devices, not property values
* New properties are exposed on getScreens() objects and window.screen
  * This requires synchronous access to the new properties on window.screen
  * How to support permission requirements of synchronously exposing new info

The existing Screen (via window.screen) is a live object. A cached Screen object
(eg. `const myScreen = window.screen`) returns updated values after screen changes,
and even updates when the window is moved across displays.

This proposal aims to return a static array of static objects and an event when
any screen information changes. This seems easy for developers to reason about,
easy for browsers to implement, and follows some general advice in the
[Web Platform Design Principles](https://w3ctag.github.io/design-principles/#live-vs-static).

Alternative approaches include returning dynamic collections of live objects
(eg. collection length changes when screens connect/disconnect, and properties
of each Screen in the collection change with device configuration changes), or
static collections of live objects (eg. indivual Screen properties change, but
the overall collection does not change reflect newly connected screens). There
are tradeoffs to each approach, but these seem generally less desirable.

The difference of static screen information snapshots from getScreens() and the
live Screen object from window.screen yields some compatibility questions:
* How static getScreens() objects handle the methods and event handler of
  [screen.orientation](https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation)
  * Leave these inoperative; callers must access window.screen
  * Support these APIs for cross-screen handlers, locks, etc.
* Should ScreenOrientation have another access method?
* Should window.screen ultimately be a static snapshot, instad of a live object?

Some of these topics are explored further in these open issues:
* Should the API return static or live objects?
  ([#12](https://github.com/webscreens/screen-enumeration/issues/12))
* Should the lifetime of a Screen object be limited?
  ([#8](https://github.com/webscreens/screen-enumeration/issues/8))

## Alternative proposals

### Alternative names and scopes for getScreens()

Besides the leading proposal to implement a new `window.getScreens()` method,
some alternative names and locations are listed below.

A `Screens` Web IDL namespace could provide a shared location for `getScreens()`
and related functionality that may be added in the future, but at this time,
there is no obvious additional API surface to motivate that approach. Using a
namespace would be preferable to a non-constructable class or interface.

```js
async () => {
  // 1: Screens namespace with get() on Window/WindowOrWorkerGlobalScope, like:
  //   * WindowOrWorkerGlobalScope.caches.keys()
  //   * WindowOrWorkerGlobalScope.indexedDB.databases()
  // Note: This cannot use `screen`, per the existing window.screen property.
  const screensV1 = await self.screens.get();

  // 2: getScreens() directly on Navigator and WorkerNavigator, like:
  //   * Navigator.getVRDisplays()
  const screensV2 = await navigator.getScreens();

  // 3: Screens namespace with get() on Navigator and WorkerNavigator, like:
  //   * Navigator.bluetooth.requestDevice()
  const screensV3 = await navigator.screens.get();

  // Alternative names:
  foo.getDisplays();
  foo.requestScreens();
  foo.requestDisplays();
}
```

### Alternative synchronous access pattern

The leading proposal is an asynchronous interface. Alternatively, a synchronous
API would match the existing `window.screen` API, but may require implementers
to cache system information that would not otherwise be required. It would also
prevent access control using a permission model, or require implementers to stop
script execution while the permission model is queried or the user is prompted.

```js
// Window or WindowOrWorkerGlobalScope attribute, like `window.screen`
const avialableScreens = self.screens;
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

### Alternatively using `Screen` to represent the entire screen space.

Representing the entire combined screen space with the existing `Screen`
interface is inadvisable, as it would come with many complications, for example:
* The union of separate physical display bounds may be an irregular shape,
  comprised of rectangles with different sizes and un-aligned positions. This
  cannot be adequately represented by the current `Screen` interface.
* The set of connected physical displays may have different `Screen` properties.

### Relation to Presentation and Remote Playback APIs

The [Presentation](https://www.w3.org/TR/presentation-api/) and
[Remote Playback](https://www.w3.org/TR/remote-playback/) APIs provide access to
displays connected on remote devices, and to device-local secondary displays,
but they are geared towards showing a single fullscreen content window on each
external display and have other limitations regarding our described use cases.

This proposal and the associated [Window Placement API][2] proposal aim to offer
compelling features that complement and extend existing Web Platform APIs. For
example this proposal would offer sites the ability to show their own list of
displays to the user, open non-fullscreen windows, limit display selection to
those directly connected to controller device (not those on remote devices),
instantly show content on multiple displays without separate user interaction
steps for each display, and to swap the displays hosting each content window
without re-selecting displays.

## Privacy & Security

For an in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/webscreens/screen-enumeration/blob/master/security_and_privacy.md).

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
prompt the user to select whether to fully block or allow the request, or even
which specific `Screens` and other information, to share with the site.

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Screen
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[3]: https://github.com/webscreens/window-placement

# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This API exposes a set of properties for each of the displays connected to the
user's device. Currently, an application window can access a subset of these
properties, but only for the display it occupies. Having access to a display's
properties without having to place a window in it allows the application to
predict which display is most appropriate for each piece of Web content and
automatically arrange its windows in the optimal layout. For example, a slide
show presentation application could automatically allocate the presentation to
the largest external display, the speaker notes to the next largest display, and
the presentation controls to the built-in display when the user clicks a
"Present" button.

These properties are implemented by some browsers and should be standardized to
convey the relative placement of screens and their available work areas:
* The left screen coordinate
* The top screen coordinate
* The left available coordinate
* The top available coordinate

These new display properties would help web applications predict and persist the
optimal layout for content in the available screen space:
* Whether it is internal (built-in) or external
  * E.g. show speaker notes on the laptop's built-in display and show the
  presentation on the external display
* Whether it is the primary display or a secondary display
  * E.g. show the speaker notes on the primary display and the presentation on
  a secondary display
* Display scale factor
  * E.g. given two screens of the same size, render the presentation on the
  screen with the larger scale factor
* An identifier for the screen
  * E.g. persist the user's preference to show slides on the screen with id 2
* Whether the screen supports touch input
  * E.g. put presenter controls on the display with touch support

These properties warrant consideration, but are not in the current proposal:
* Whether the display supports accelerometer info: useful for showing immersive
  controls (e.g. game steering wheel)
* The dpi of the display (pixels per inch): useful for presenting content with
  tailored physical scale factors
* A set of properties useful for adapting content presentation to certain
  display technology:
  * The display's subpixel order
  * Whether the display's mode is interlaced
  * The display's refresh rate in hertz
  * The display's overscan insets within its screen's bounds.
* Whether the display is not visible (e.g. closed laptop): useful for
  recognizing when displays may be active but not visible
* Whether the screen is mirroring content from another screen: useful for
  recognizing when a laptop is mirrored to a projector

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

This specification exposes information for each connected screen, comparable to
what is already exposed by the existing Screen object (for the one screen
hosting an associated content window, or some critical portion of that window).

This specification also exposes new information about each screen, reflecting
display properties that help web applications choose the appropriate screen
for a given task. The properties have been prioritized and selected for their
potential value to the most common and critical use cases.

The [Privacy & Security](https://github.com/webscreens/screen-enumeration/blob/master/EXPLAINER.md#privacy--security)
section of the explainer briefly mentions the possibility of requesting limited
or granular screen information, and this tactic should be explored further.

It has been suggested that user agents themselves could provide a UI for
selecting where content is presented, but this is cumbersome for users and
prohibitively limiting for web applications. This also offers no meaningful
value over the existing cumbersome manual placement of windows by users.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

This API exposes some new information about the device's display configuration.
Since that information could be used to help identify a device, access can be
gated on user permission.

## 2.4 How does this specification deal with sensitive information?

This API does not expose such information.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

The user agent could persist screen permission grants.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

This API proposes exposing about 9-17 new properties for each connected display,
most of which directly correlate with underyling platform configuration data.
See Section 2.1 for a list of new properties considered by this proposal.

## 2.7 Does this specification allow an origin access to sensors on a user’s device?

No. If anything, this API may expose the presence of touch or accelerometer
sensors associated with each display, but not access to sensor data itself.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

The API exposes a set of [`Screen`](https://developer.mozilla.org/en-US/docs/Web/API/Screen)
objects, one for each connected display, beyond the single Screen object
currently available to each content window. The same set of Screen objects is
already exposed to an origin, if windows for that origin are placed on each of
the connected screens, via aggregating the respective |window.screen| objects.

This API currently proposes exposing these new properties on each Screen object:
* The left screen coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left)
* The top screen coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top)
* The left available coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.availLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availLeft)
* The top available coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.availTop`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availTop)
* Whether it is internal (built-in) or external
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo),
* Whether it is the primary display or a secondary display
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Display scale factor
  * Not standardized, but already exposed by some browsers via
    [`Window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)
* An identifier for the screen
  * Not web-exposed, but a more persistent id is available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Whether the screen supports touch input
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)

This API considers additional new properties, that are not actively proposed:
* Whether the display supports accelerometer info
  * Not web-exposed
* The dpi of the display (pixels per inch)
  * Not standardized, but similar to data already exposed by some browsers via
    [`Window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)
* The display's subpixel order
  * Not web-exposed
* Whether the display's mode is interlaced
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* The display's refresh rate in hertz
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* The display's overscan insets within its screen's bounds.
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Whether the screen is mirroring content from another screen
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Whether the display is not visible (e.g. closed laptop)
  * Not web-exposed

## 2.9 Does this specification enable new script execution/loading mechanisms?

No.

## 2.10 Does this specification allow an origin to access other devices?

This API allows an origin to read properties of displays connected to the
computer. The display may be internally connected (e.g. the built-in monitor in
a laptop) or externally connected (e.g. an external monitor).

An origin cannot use this API to send commands to the displays, so hardening
against malicious input is not a concern.

Enumerating the displays connected to the computer does provide significant
entropy. If multiple computers are connected to the same set of displays, an
attacker may use the display information to deduce that those computers are in
the same physical vicinity. To mitigate this issue, user permission is required
to access the display list, and the API is only available on secure contexts.

## 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

On its own, it does not. In conjunction with the proposed
[Window Placement API](https://github.com/webscreens/window-placement/blob/master/EXPLAINER.md),
this API could enable use cases where a window is opened in fullscreen mode on a
display that does not contain a window accessible to that origin at the time.

## 2.12 What temporary identifiers might this specification create or expose to the web?

None.

## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

Only first-party contexts should be able to request display information.

## 2.14 How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

The behavior should be the same as for regular mode, except that the user agent
should not persist permission data and should request permission every session.

## 2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes.

## 2.16 Does this specification allow downgrading default security characteristics?

No.
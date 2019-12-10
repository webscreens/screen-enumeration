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

The following are some new display properties that would help the application
better predict the optimal content layout given the available screen space:
* Whether it is internal (built-in) or external
  * E.g. show speaker notes on the laptop's built-in display and show the
  presentation on the external display
* Whether it is the primary display
  * E.g. show the speaker notes on the primary display and the presentation on
  a secondary display
* Display scale factor
  * E.g. given two screens of the same size, render the presentation on the
  screen with the larger scale factor

Another proposed new screen property is the display name string. This info would
allow sites to present custom display selection UIs or show a descriptive
preview of a button's action, like "Present to 'Display1'". These recognizable
names would be useful for users to understand options or actions regarding the
presentation of content in a multi-screen environment.

There are other properties considered in the explainer, each offering potential
value, but they might not be as widely applicable, and it may be more important
to limit the set of exposed properties.

* An identifier; useful for persisting window placements by display. The user
  agent could map sensitive EDIDs to indices
* Whether the display supports touch interaction; useful for presenting
  touch-specific UI layouts
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
  recognizing when displays may be active but not visible.

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

Each of the new display properties chosen for this API makes a non-replaceable
contribution. Removal or constraining of any property may degrade the user
experience. Some properties offer more value than others, and some are more
relevant to the proposed use cases than others. Care should be taken to choose
those properties offering the most value to the most important use cases.

It has been suggested that user agents themselves could provide a UI for
selecting where content is presented, but this is cumbersome for users and
prohibitively limiting for web applications. This also offers no meaningful
value over the existing cumbersome manual placement of windows by users.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

This API does not expose such information. Any exposed information may identify
the machine, but not its user.

## 2.4 How does this specification deal with sensitive information?

This API does not expose such information.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

The user agent could persist screen permission grants.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

This API proposes exposing some additional properties for each of the displays
connected to the device; see Section 2.1 for a complete list.

## 2.7 Does this specification allow an origin access to sensors on a user’s device?

No.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

The screen properties in this API proposal that already exist through other Web
APIs are currently exposed:
* synchronously
* in only the document/frame execution context
* for the display containing the window

This API proposes exposing these existing properties *and new ones*:
* asynchronously
* in both the document/frame and worker execution contexts
* for all displays, not just the one containing the window

### Existing properties

The following display properties are currently exposed synchronously only in the
document/frame execution context via the
[`Screen`](https://developer.mozilla.org/en-US/docs/Web/API/Screen) interface:
* color depth, i.e.
[`Screen.colorDepth`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/colorDepth)
* width and height of the entire display, i.e.
[`Screen.width`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/width),
[`Screen.height`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/height)
* width and height of user window area (excluding system UI areas), i.e.
[`Screen.availWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availWidth),
[`Screen.availHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availHeight)
* orientation (landscape vs. portrait and rotation in degrees), i.e.
[`Screen.orientation`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/orientation)

The following unstandardized properties of the `Screen` interface, exposed
synchronously in the document/frame execution context for some browsers:
* the display's origin, or placement relative to the primary display, i.e.
  [`Screen.left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left),
  [`Screen.top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top)
* left and top of user window area (excluding system UI areas), i.e.
  [`Screen.availLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availLeft),
  [`Screen.availTop`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availTop)

The following properties are exposed synchronously in the document/frame
execution context via the
[`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) interface:
* x-,y-coordinates of the window (relative to entire screen space), i.e.
[`Window.screenLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenLeft),
[`Window.screenTop`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenTop)
* scaling factor (unstandardized), i.e.
[`Window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)

### New properties

The following display properties are not currently Web-exposed, but are
available in the Chrome Apps API via the
[`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo),
given the `"system.display"` permission:
* name
* whether the display is the primary display
* whether the display is internal (built-in) or external

The following display properties are not currently Web-exposed:
* An identifier
* Whether the display supports touch interaction
* Whether the display supports accelerometer info
* The dpi of the display (pixels per inch)
* The display's subpixel order
* Whether the display's mode is interlaced
* The display's refresh rate in hertz
* The display's overscan insets within its screen's bounds
* Whether the display is not visible (e.g. closed laptop)

See Section 2.1 for more information about each of the proposed properties.

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
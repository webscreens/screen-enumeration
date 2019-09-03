# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review
Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

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

The following are new display properties that would help the application better
predict the optimal content layout given the available screen space:
* resolution, or scale factor
  * E.g. given 2 screens of the same size, render the presentation on the screen
  with the better resolution
* whether it is the primary display
  * E.g. render the speaker notes on the primary display and the presentation on
  a secondary display
* whether it is internal (built-in) or external
  * E.g. render the speaker notes on the built-in display and the presentation
  on the external display

The remaining new display property is the display name. The name would allow the
user to recognize their displays in a display chooser UI for selecting in which
display to render a piece of content.

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

Each of the new display properties chosen for this API makes a non-replaceable
contribution. Removal or constraining of any property would degrade the user
experience.

For example, the API could expose a display's identity to the end user by its
proper name in a display chooser UI implemented by the user agent, and to
scripts by a hash value or number. This alternative, however, reduces the
ability for applications to tailor the chooser UI to each of their use cases.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

This API does not expose such information. Any exposed information may identify
the machine, but not its user.

## 2.4 How does this specification deal with sensitive information?

This API does not expose such information.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

The user agent could persist screen permission grants.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

This API proposes exposing the following properties for each of the displays
(physical or virtual) connected to the device:
* color depth
* width and height of the entire display
* width and height of the part of the display that the window may occupy (i.e.
excludes space occupied by system UI)
  * NOTE: The available width/height may indirectly expose the operating system.
* orientation
* smallest x-,y-coordinates of this display (relative to entire screen space)
  * NOTE: The overall layout of a multi-monitor setup can be deduced by mapping
  out each display by its coordinates in the screen space.
* smallest x-,y-coordinates of this display that the window may occupy (relative
to entire screen space)
* scaling factor
* name
  * NOTE: The default display name usually exposes the display's make/model.
* whether the display is the primary display
* whether the display is internal (built-in) or external

## 2.7 Does this specification allow an origin access to sensors on a user’s device?

No.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

The display properties in this API proposal that already exist through other Web
APIs are currently exposed:
* synchronously
* in only the document/frame execution context
* for the display containing the window

This API proposes exposing these existing properties and new ones:
* asynchronously
* in both the document/frame and worker execution contexts
* for all displays, not just the one containing the window

### Existing properties

The following display properties are currently exposed synchronously only in the
document/frame execution context via the `Screen` interface:
* color depth, i.e.
[`Screen.colorDepth`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/colorDepth)
* width and height of the entire display, i.e.
[`Screen.width`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/width),
[`Screen.height`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/height)
* width and height of the part of the display that the window may occupy (i.e. excludes
space occupied by system UI), i.e.
[`Screen.availWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availWidth),
[`Screen.availHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availHeight)
* orientation, i.e.
[`Screen.orientation`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/orientation)

The following display properties are currently unstandardized properties of the
`Screen` interface, and thus may be exposed synchronously in the document/frame
execution context for some browsers:
* smallest x-,y-coordinates of this display that the window may occupy (relative
to entire screen space), i.e.
[`Screen.availLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availLeft),
[`Screen.availTop`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availTop)

The following are also unstandardized properties of the `Screen` interface, but
are also exposed synchronously in the document/frame execution context via the
`Window` interface:
* smallest x-,y-coordinates of this display (relative to entire screen space),
i.e.
[`Screen.left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left)
or
[`Window.screenLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenLeft),
and
[`Screen.top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top) or
[`Window.screenTop`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenTop)

The following display property is currently exposed synchronously in the
document/frame execution context via the `Window` interface.
* scaling factor, i.e.
[`Window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)

### New properties

The following display properties are not currently Web-exposed, but are
available in the Chrome Apps API via the
[`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo),
given the `"system.display"` permission:
* name
* whether the display is the primary display
* whether the display is internal (built-in) or external


## 2.9 Does this specification enable new script execution/loading mechanisms?

No.

## 2.10 Does this specification allow an origin to access other devices?

This API allows an origin to read properties of displays connected to the
computer. The display may be internally connected (e.g. the built-in monitor in
a laptop) or externally connected (e.g. an external monitor).

An origin cannot use this API to send commands to the displays, so hardening
against malicious input is not a concern.

Enumerating the displays connected to the computer does provide signficant
entropy. If multiple computers are connected to the same set of displays, an
attacker may use the display information to deduce that those computers are in
the same physical vicinity. To mitigate this issue, user permission is required
to access the display list, and the API is only available on secure contexts.

## 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

On its own, it does not. In conjunction with the proposed
[Window Placement API](https://github.com/spark008/window-placement/blob/master/EXPLAINER.md),
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
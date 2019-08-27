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
the presentation controls to the built-in display when the user clicks the
"Present" button.

The following are new display properties that would help the application better
predict the optimal content layout given the available screen space:
* resolution, or scale factor
  * E.g. given 2 screens of the same size, render the presentation on the screen
  with the better resolution
* whether it is the primary display
  * E.g. render the speaker notes on the primary display and the presentation on
  another display
* whether it is internal (built-in) or external
  * E.g. render the speaker notes on the built-in display and the presentation
  on the external display

The remaining new display property is the display name. The name would allow the
user to recognize their displays in a display chooser UI for selecting in which
display to render a piece of content.

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

By building the display chooser UI into the browser, the API could potentially
expose a (non-human readable) hash value to identify each display rather than
the display's proper name. This alternative, however, reduces the ability for
applications to tailor the chooser UI to each of their use cases.

Each of the remaining new display properties reveals a non-overlapping insight
about the screen configuration that could have a major impact on how well the
application is able to predict the optimal content layout.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

Access to display properties will be gated behind a permission prompt built into
the browser.

## 2.4 How does this specification deal with sensitive information?

Access to display properties will be gated behind a permission prompt built into
the browser.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

The browser could persist whether the user granted screen permission to the
origin.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

The display name usually exposes the make/model of the connected displays. The
resolution and available width/height may indirectly expose the operating
system and device make/model.

## 2.7 Does this specification allow an origin access to sensors on a user’s device

No.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

## 2.9 Does this specification enable new script execution/loading mechanisms?

## 2.10 Does this specification allow an origin to access other devices?

## 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

## 2.12 What temporary identifiers might this this specification create or expose to the web?

## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

## 2.14 How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

## 2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

## 2.16 Does this specification allow downgrading default security characteristics?

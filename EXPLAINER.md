# Screen Enumeration

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays.

This proposal gives developers access to a list of the available screens and the display properties of each screen. It is foundational for some parts of the [Window Placement API proposal](https://github.com/spark008/window-placement) and can be used to enhance other parts.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
  * Move the speaker notes to a specific screen, not in fullscreen mode.
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
* **Video player that optimizes video specs/quality for each screen**
  * Video is optimized for the screen on which it is rendered.

## Goals / Non-goals

## Proposal

### Option 1: Add interface to `Navigator`/`WorkerNavigator`

Add a `ScreenManager` read-only property to `Navigator` and `WorkerNavigator`. The `ScreenManager` interface would provide functionality around requesting access to displays and their properties, listening to display connection changes, etc.
```js
async () => {
  const displays = await navigator.screen.requestDisplays();
);
```

### Option 2: Add `async requestDisplays()` to `Navigator`/`WorkerNavigator`

### Option 3: Add `async requestDisplays()` to `Window`/`WorkerGlobalScope`

### Option 4: Add `screens` read-only property to `Window`/`WorkerGlobalScope`


## Privacy & Security

### Screen enumeration

Exposing the details of a user's multi-screen setup presents a fingerprinting concern.
In order to mitigate the amount of personally identifying information exposed, while maintaining the usefulness of the API, we can implement the following best practices:
* return the screens by increasing `width`
* limit the screen properties exposed to:
  * width
  * height
* limit the API to secure contexts

# Screen Enumeration

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays.

The Screen Enumeration API gives developers access to a list of the available screens and the display properties of each screen. It is foundational for some parts of the [Window Placement API](https://github.com/spark008/window-placement) and can be used to enhance other parts.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
    ```js
    const screens = window.screens;

    // Option 1: Blow up multiple elements living in a single window.
    const presentation = document.querySelector("#presentation");
    const notes        = document.querySelector("#notes");
    const controls     = document.querySelector("#controls");

    presentation.requestFullscreen({ screen: screens[0] });
    notes.requestFullscreen({ screen: screens[1] });
    controls.requestFullscreen({ screen: screens[2] });

    // Option 2: Blow up multiple windows.
    window.open("/presentation", "presentation", "fullscreen", screens[0]);
    window.open("/notes", "notes", "fullscreen", screens[1]);
    window.open("/controls", "controls", "fullscreen", screens[2]);
    ```
  * Move the speaker notes to a specific screen, not in fullscreen mode.
    ```js
    const screen = window.screens[0];
    const notesWindow = window.open("", "notes");
    notesWindow.moveTo(screen);
    notesWindow.resizeTo(100, 100);
    ```
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
    ```js
    // Service worker script
    self.addEventListener("launch", event => {
      event.waitUntil(async () => {
        const screens = self.screens;
        const maxDashboardCount = 5;
        const usedScreenCount = Math.min(screens.length, maxDashboardCount);
        for (let screen = 1; screen < usedScreenCount; ++screen) {
          await clients.openWindow(`/dashboard/${screen}`, screens[screen]);
        }
      });
    });
    ```

## Goals / Non-goals

## Proposal

### Option 1: Screen enumeration

* `Window.screens`
* `ServiceWorkerGlobalScope.screens`

### Option 2: Screen discovery

`navigator.screens.requestScreens()`


## Privacy & Security

### Screen enumeration

Exposing the details of a user's multi-screen setup presents a fingerprinting concern.
In order to mitigate the amount of personally identifying information exposed, while maintaining the usefulness of the API, we can implement the following best practices:
* return the screens by increasing `width`
* limit the screen properties exposed to:
  * width
  * height
* limit the API to secure contexts

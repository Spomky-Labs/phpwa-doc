# Display Override

The `display_override` property in a PWA manifest allows developers to specify a preferred display mode with fallbacks for cases where a certain display mode is not supported by the platform or browser. This ensures that the best possible display mode is used for the PWA without errors.

In the example below, the browser will consider the following display-mode fallback chain in this order: `fullscreen` → `minimal-ui` → `standalone`.

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        display: "standalone"
        display_override: ["fullscreen", "minimal-ui"]
```

Display override objects are display-mode strings, the possible values are:

| Display Mode | Description |
|---|---|
| `fullscreen` | All of the available display area is used and no user agent UI is shown. |
| `standalone` | The application will look and feel like a standalone application. This can include the application having a different window, its own icon in the application launcher, etc. In this mode, the user agent will exclude UI elements for controlling navigation, but can include other UI elements such as a status bar. |
| `minimal-ui` | The application will look and feel like a standalone application, but will have a minimal set of UI elements for controlling navigation. The elements will vary by browser. |
| `browser` | The application opens in a conventional browser tab or new window, depending on the browser and platform. This is the default. |
| `window-controls-overlay` | This display mode only applies when the application is in a separate PWA window and on a desktop operating system. The application will opt-in to the Window Controls Overlay feature, where the full window surface area will be available for the app's web content and the window control buttons (maximize, minimize, close, and other PWA-specific buttons) will appear as an overlay above the web content. |

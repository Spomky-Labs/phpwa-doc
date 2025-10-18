# Install

This controller handles the installation flow of a Progressive Web App (PWA).\
It listens for browser installation events, detects if the app is already installed, and provides an easy way to trigger the “Add to Home Screen” prompt when supported.

**Usage**

```twig
<div {{ stimulus_controller("@pwa/install") }}>
  <button {{ stimulus_action("@pwa/install", "install") }}>Install App</button>
</div
```

### Parameters

None

### Actions

install

### Targets

None

### Events

`installed`: The PWA is installed or detected as already running in standalone mode. It is emitted on startup if the app is already installed.

`not-installed`: The PWA is not installed but can potentially be installed.

`installing`: The user triggers the `install()` action and the install prompt is shown.

`cancelled`: The user dismisses the install prompt or an error occurs during installation.

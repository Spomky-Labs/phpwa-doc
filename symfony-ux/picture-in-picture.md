# Picture In Picture

This controller opens a floating Picture-in-Picture window for a chosen DOM element.\
It works with (classic PiP) and can also float any DOM subtree (Document PiP where supported).\
When the PiP window is closed, the floated element is put back into its original position in the container (when possible).

**Usage**

```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
  <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle') }}>
    Toggle Picture-in-Picture
  </button>

  <div {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
    <video controls {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}>
      <source src="..." type="video/mp4">
    </video>
  </div>
</div>
```

### Parameters

None

### Actions

`toggle`: Enters PiP if not active; exits PiP if already active.

### Targets

None

### Events

`unsupported`: PiP is not available (feature missing, permission not granted, platform denies).

`enter`: The element successfully entered PiP.

`exit`: The PiP session has ended (user closed it or you exited programmatically).

`error`: Entering or exiting PiP fails (e.g., constraint, blocked, promise rejection).

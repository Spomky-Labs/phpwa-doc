# Device Motion

This controller listens to devicemotion events from the Device Motion API\
and emits a normalized updated event containing acceleration, acceleration including gravity, rotation rate, and sampling interval. It also manages the permission flow when required.

**Usage**

```twig
<div {{ stimulus_controller('@pwa/device-motion') }}>
  <p id="motion-status">Waiting for motion data…</p>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const out = document.getElementById('motion-status');

  host.addEventListener('device-motion:unavailable', () => {
    out.textContent = 'Device Motion API is not available on this device/browser.';
  });

  host.addEventListener('device-motion:permission-granted', () => {
    out.textContent = 'Permission granted. Receiving motion data…';
  });

  host.addEventListener('device-motion:permission-denied', () => {
    out.textContent = 'Permission denied. Cannot access motion sensors.';
  });

  host.addEventListener('device-motion:updated', (e) => {
    const { acceleration, rotationRate, interval } = e.detail;
    out.textContent =
      `Acc: x=${acceleration.x?.toFixed(2)} y=${acceleration.y?.toFixed(2)} z=${acceleration.z?.toFixed(2)} | ` +
      `Rot: α=${rotationRate.alpha?.toFixed(1)} β=${rotationRate.beta?.toFixed(1)} γ=${rotationRate.gamma?.toFixed(1)} | ` +
      `dt=${interval}ms`;
  });
</script>
```

### Parameters

`throttleValue`: Intended to control how often `updated` events are dispatched to avoid spamming the UI.

### Actions

None

### Targets

None

### Events

`unavailable`: the feature is not supported by the device.

`permission-granted`: permission prompt was granted; listener is attached.

`permission-denied`: User denied permission, or an error occurred during the permission request.

`updated`: On every device motion event (as delivered by the browser).

Contains the following object:

```json
{
  acceleration: {
    x: number | null,
    y: number | null,
    z: number | null
  },
  accelerationIncludingGravity: {
    x: number | null,
    y: number | null,
    z: number | null
  },
  rotationRate: {
    alpha: number | null, // deg/s around Z
    beta: number | null,  // deg/s around X
    gamma: number | null  // deg/s around Y
  },
  interval: number // ms between samples
}
```

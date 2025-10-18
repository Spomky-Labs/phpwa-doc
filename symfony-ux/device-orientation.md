# Device Orientation

This controller uses the Device Orientation API\
to listen to changes in the physical orientation of a device (phone, tablet, etc.).\
It emits an updated event at a throttled rate containing the current rotation angles.

### ⚠️ Notes

* This API is available only on **mobile and tablet devices** equipped with motion sensors.
*   On iOS (Safari), users must **grant permission** via a dialog triggered by a **user gesture**:

    ```js
    if (typeof DeviceOrientationEvent.requestPermission === 'function') {
      await DeviceOrientationEvent.requestPermission();
    }
    ```
* On desktop browsers, events may never fire or return `null` values.
* For performance reasons, events are **throttled** by default (see `throttleValue`).

**Usage**

```twig

<div {{ stimulus_controller('@pwa/device-orientation', {throttle: 25}) }}>
  <p id="orientation-status">Waiting for orientation data…</p>
</div>

<script type="module">
  const el = document.querySelector('[data-controller="pwa__device-orientation"]');

  el.addEventListener('device-orientation:updated', (e) => {
    const { alpha, beta, gamma } = e.detail;
    document.getElementById('orientation-status').textContent =
      `α: ${alpha?.toFixed(1)}°, β: ${beta?.toFixed(1)}°, γ: ${gamma?.toFixed(1)}°`;
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

`updated`: Emitted each time the device orientation changes (subject to the throttle delay).

Contains an object with the information (values are in degrees):

* `absolute`: boolean,
* `alpha`: number | null,
* `beta`: number | null,
* `gamma`: number | null

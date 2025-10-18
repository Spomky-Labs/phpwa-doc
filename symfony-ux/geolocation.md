# Geolocation

TO BE WRITTEN

**Usage**

```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
  <button {{ stimulus_action('@pwa/geolocation', 'watch', 'click', {enableHighAccuracy: true, maximumAge: 500, timeout: 1000}) }}>Follow me</button>
</div
```

### Parameters

None

### Actions

`locate`: Retrieves the user’s current position once.

`watch`: Starts continuous position tracking.

`clearWatch`: Stops continuous tracking (if active).

### Targets

None

### Events

`position`: A position update is received.

Contains the `position` object includes:

* `coords.latitude`
* `coords.longitude`
* `coords.accuracy`
* `coords.altitude`
* `coords.speed`
* `timestamp`

`error`: The API call fails or the user denies permission.

`unsupported`: The browser does not support the Geolocation API.

`watch:cleared`: The ongoing position watch is stopped.

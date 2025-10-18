# Wake Lock

The Screen Wake Lock API enables web apps to prevent devices from dimming or locking the screen when the app needs to keep running.

### Parameters

None

### Actions

`lock`

`unlock`

`toggle`

### Targets

None

### Events

`pwa--wake-loc:updated` : the payload property `wakeLock` contains either `null` (unlocked) or an object (locked) with keys `type` (string) and `released` (bool)

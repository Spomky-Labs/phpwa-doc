# Service Worker

The Service Worker controller provides useful methods to facilitate interaction with the service worker lifecycle. Currently, it offers a method to manually update the service worker when the [`skipWaiting`](../the-service-worker/configuration.md#skip-waiting) option is set to `false`.

**Usage**

```twig
<div {{ stimulus_controller('@pwa/service-worker')>
    <button {{ stimulus_action('@pwa/service-worker', 'update') }}>
        Update
    </button>
</div>
```

### Parameters

None

### Actions

`update`: tries to update the service worker. Will refresh the page when done.

### Targets

None

### Events

`pwa--service-worker:update-available`: indicates an update is available. Can be used to update the UI and show the update button.

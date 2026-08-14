# Sync Broadcast

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Sync Broadcast controller (`backgroundsync-queue`) monitors background sync queue status using the BroadcastChannel API. It allows your application to display real-time information about pending sync operations and trigger replays.

## Installation

Enable the controller in your `assets/controllers.json`:

{% code title="assets/controllers.json" lineNumbers="true" %}
```json
{
    "controllers": {
        "@spomky-labs/pwa-bundle": {
            "backgroundsync-queue": {
                "enabled": true
            }
        }
    }
}
```
{% endcode %}

## Usage

{% code title="templates/components/sync-status.html.twig" lineNumbers="true" %}
```twig
<div {{ stimulus_controller('pwa/backgroundsync-queue', {
    channel: 'form-sync'
}) }}
    {{ stimulus_action('pwa/backgroundsync-queue', 'replay', 'click') }}
>
    <span {{ stimulus_target('pwa/backgroundsync-queue', 'output') }}>
        Checking sync status...
    </span>
    <button type="button">Retry pending items</button>
</div>
```
{% endcode %}

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `channel` | String | Yes | The BroadcastChannel name to listen on. Must match the `broadcast_channel` configured in your background sync strategy. |

## Actions

| Action | Description |
|--------|-------------|
| `replay` | Requests a replay/resync of all pending operations in the background sync queue. |

## Targets

| Target | Description |
|--------|-------------|
| `output` | Element to display queue status information. |

## Events

| Event | Detail | Description |
|-------|--------|-------------|
| `error` | `{ reason }` | BroadcastChannel initialization failed. |
| `status` | Status data from service worker | Status message received from the background sync queue. |

## How It Works

1. The controller opens a `BroadcastChannel` with the specified name
2. It requests the current sync queue status
3. When the service worker processes queued requests, it broadcasts status updates
4. The controller dispatches `status` events that you can listen for to update the UI
5. The `replay` action tells the service worker to retry all pending requests

## Example: Showing Pending Request Count

{% code title="templates/components/sync-indicator.html.twig" lineNumbers="true" %}
```twig
<div {{ stimulus_controller('pwa/backgroundsync-queue', {
    channel: 'form-sync'
}) }}
    data-action="pwa--backgroundsync-queue:status->updateStatus"
>
    <span class="badge" data-sync-count></span>
    <button {{ stimulus_action('pwa/backgroundsync-queue', 'replay', 'click') }}>
        Sync now
    </button>
</div>

<script>
function updateStatus(event) {
    const badge = document.querySelector('[data-sync-count]');
    badge.textContent = event.detail.pendingCount ?? '0';
}
</script>
```
{% endcode %}

## Related Configuration

This controller works with the [Background Sync](../the-service-worker/workbox/background-sync.md) service worker feature. Make sure the `broadcast_channel` name in your service worker configuration matches the `channel` parameter:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            background_sync:
                - queue_name: 'api-queue'
                  match_callback: 'regex: /\/api\//'
                  method: 'POST'
                  broadcast_channel: 'form-sync'  # Must match controller's channel param
```
{% endcode %}

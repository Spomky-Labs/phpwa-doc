# BackgroundSync Queue

When the BackgroundSync broadcast channel is configured, an event is sent to indicate the number of requests in the queue. Additionally, the queue can be replayed on-demand.&#x20;

The channel name on the Stimulus Controller should match the one used in the configuration.

**Usage**

{% code title="" overflow="wrap" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            enabled: true
            background_sync:
                - queue_name: 'form'
                  #... other parameter here
                  broadcast_channel: 'form-list'
```
{% endcode %}

{% code overflow="wrap" lineNumbers="true" %}
```twig
<script type="module">
    const target = document.getElementById('remaining-items');
    const update = (e) => {
        if (Number.isInteger(e.detail.remaining)) {
            target.innerText = e.detail.remaining;
        }
    };
    document.addEventListener('pwa--backgroundsync-queue:status', (e) => update(e));
</script>

Number of requests in the queue: <span id="remaining-items">--</span>.
<button
    {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: "form-list"}) }}
    {{ stimulus_action('@pwa/backgroundsync-queue', 'replay')}}>
    Replay!
</button>
```
{% endcode %}

### Parameters

`channel`: the channel to listen to. Should correspond to a broadcast channel set in the configuration file.

### Actions

**Replay Function**

`replay`: attempts to process the pending requests in the queue.

### Targets

None

### Events

`pwa--backgroundsync-queue:status`: the payload is an object with the parameters `name` (name of the queue) and `remaining` (integer)

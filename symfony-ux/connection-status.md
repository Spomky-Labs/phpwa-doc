# Connection Status

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Connection Status component is designed to monitor and react to changes in the user's internet connection status. This component provides a simple way to show a notification or change the state of your application when the user's device goes offline or comes back online.

**Usage**

To use the Connection Status component, include it in your application and initialize it with the necessary options. Below is an example of how to integrate the Connection Status component into your project:

```twig
<div class="mx-auto max-w-screen-xl text-center px-4" {{ stimulus_controller('@pwa/connection-status') }}>
    <div
        {{ stimulus_target('@pwa/connection-status', 'attribute') }}
        class="flex items-center p-4 mb-4 text-sm border rounded-lg online:text-green-800 online:bg-green-50 online:dark:bg-gray-800 online:dark:text-green-400 offline:text-yellow-800 offline:bg-yellow-50 offline:dark:bg-gray-800 offline:dark:text-yellow-300"
        role="alert"
    >
        <div>
            <span class="font-medium">
                Connection status
            </span>: 
            <span {{ stimulus_target('@pwa/connection-status', 'message') }}>
                We are trying to guess what is the current status of your Internet connection
            </span>
        </div>
    </div>
</div>
```

### Parameters

* `onlineMessage`: Message displayed when online
* `offlineMessage`: Message displayed when offline

### Actions

None

### Targets

* `message`: HTML tag to update with the connection status message. Multiple targets allowed.
* `attribute`: HTML attribute `data-connection-status="ONLINE"` (or `"OFFLINE"`) will be set depending on the connection status. In the example above, this attribute is used to change the applicable classes

### Events

On status change, the event `status-changed` is dispatched. The payload property contains the following data:

* status: `"OFFLINE"` or `"ONLINE"`
* `message`: the message set as parameter or the default value.

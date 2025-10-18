# BackgroundSync Form

The Background Sync API enables requests to be sent even when offline. These requests are queued and automatically dispatched once the user is online. While online, normal form submission behaviors remain unchanged.

The configuration below ensures all requests starting with `/form` are queued and sent upon reconnection. Ensure the queue name is unique. The request method can be customized to differentiate, for instance, between `POST` and `PUT` requests, allowing for distinct retention periods.

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
                  match_callback: 'startsWith: /form'
                  method: POST
                  max_retention_time: 7_200 # 5 days in minutes
                  force_sync_fallback: true #Optional
```
{% endcode %}

{% code overflow="wrap" lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }} method="post">
    ...
    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send')}} type="submit">
        Send!
    </button>
</form>
```
{% endcode %}

### Parameters

`params`: an object with the `fetch` parameters to use. Default value:&#x20;

```json
{
    mode: 'cors',
    cache: 'no-cache',
    credentials: 'same-origin',
    redirect: 'follow',
    referrerPolicy: 'no-referrer'
}
```

`headers`: header parameters to be passed to the request. Default value: `{}`

`redirection`: when offline and after the form submission, the redirection URI. Default value: `null`.

### Actions

`send`: To be applied on the form submit button.

### Targets

None

### Events

`pwa--backgroundsync-form:before:send`: contains the `URL` and the combined `fetch` parameters. Sent just before the request submission.

`pwa--backgroundsync-form:after:send`: contains the `URL` , the combined `fetch` parameters and the `response`. Sent right after the request submission.

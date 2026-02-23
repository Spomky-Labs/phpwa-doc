# BackgroundSync Form

The BackgroundSync Form controller handles form submission with offline support. When the user is online, forms are submitted normally. When offline, form data is queued and automatically retried when the connection is restored.

## Installation

Enable the controller in your `assets/controllers.json`:

{% code title="assets/controllers.json" lineNumbers="true" %}
```json
{
    "controllers": {
        "@spomky-labs/pwa-bundle": {
            "backgroundsync-form": {
                "enabled": true
            }
        }
    }
}
```
{% endcode %}

## Usage

{% code title="templates/contact/form.html.twig" lineNumbers="true" %}
```twig
<form {{ stimulus_controller('pwa/backgroundsync-form', {
    redirection: path('app_contact_success'),
    headers: { 'X-Requested-With': 'XMLHttpRequest' },
}) }}
    action="{{ path('app_contact_submit') }}"
    method="POST"
    {{ stimulus_action('pwa/backgroundsync-form', 'send', 'submit') }}
>
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message" required></textarea>
    <button type="submit">Send</button>
</form>
```
{% endcode %}

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `params` | Object | `{ mode: 'cors', cache: 'no-cache', credentials: 'same-origin', redirect: 'follow', referrerPolicy: 'no-referrer' }` | Fetch request options. |
| `headers` | Object | `{}` | Custom HTTP headers to include in the request. |
| `redirection` | String | `null` | URL to redirect to after successful form submission. |
| `authenticating` | Boolean | `false` | Whether to sign the request with JWT authentication. |
| `keyIdIndex` | String | `'default'` | The key ID to use for JWT signing (when `authenticating` is `true`). |

## Actions

| Action | Description |
|--------|-------------|
| `send` | Triggered on form submission. Validates the form, builds the request, and sends it. |

## Events

| Event | Detail | Description |
|-------|--------|-------------|
| `invalid-data` | - | Form validation failed (HTML5 validation). |
| `unsupported-enctype` | - | Form encoding type is not supported. |
| `auth-missing-key` | `{ keyIdIndex }` | JWT authentication enabled but required key is missing. |
| `before:send` | `{ url, params }` | Dispatched before the fetch request is sent. |
| `after:send` | `{ url, params, response }` | Dispatched after the fetch response is received. |
| `error` | `{ error }` | Fetch request failed. |

## Supported Content Types

The controller supports the following form encoding types:

- `multipart/form-data` - For file uploads
- `application/json` - JSON-encoded form data
- `application/x-www-form-urlencoded` - Standard URL-encoded form data

## How It Works

1. User submits the form
2. The controller validates the form using HTML5 validation
3. Form data is serialized based on the form's `enctype`
4. A `fetch` request is made with the configured options
5. If the network is available, the request is sent immediately
6. If the service worker intercepts the request and the user is offline, the request is queued via Background Sync
7. When the connection is restored, the service worker replays the queued request
8. On success, the user is optionally redirected to the `redirection` URL

## Example: Contact Form with Offline Support

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            background_sync:
                - queue_name: 'contact-form'
                  match_callback: 'regex: /\/contact/'
                  method: 'POST'
                  max_retention_time: 2880  # 2 days
                  broadcast_channel: 'contact-sync'
```
{% endcode %}

{% code title="templates/contact/index.html.twig" lineNumbers="true" %}
```twig
<form {{ stimulus_controller('pwa/backgroundsync-form', {
    redirection: path('app_contact_success'),
}) }}
    action="{{ path('app_contact_submit') }}"
    method="POST"
    {{ stimulus_action('pwa/backgroundsync-form', 'send', 'submit') }}
    data-action="
        pwa--backgroundsync-form:before:send->showSpinner
        pwa--backgroundsync-form:after:send->hideSpinner
        pwa--backgroundsync-form:error->showError
    "
>
    {{ form_widget(form) }}
    <button type="submit">Send Message</button>
    <div class="spinner" hidden>Sending...</div>
    <div class="error" hidden>An error occurred. Your message will be sent when you're back online.</div>
</form>
```
{% endcode %}

## Example: JSON API Form

{% code title="templates/api/form.html.twig" lineNumbers="true" %}
```twig
<form {{ stimulus_controller('pwa/backgroundsync-form', {
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
    },
}) }}
    action="{{ path('api_submit') }}"
    method="POST"
    enctype="application/json"
    {{ stimulus_action('pwa/backgroundsync-form', 'send', 'submit') }}
>
    <input type="text" name="title" required>
    <textarea name="body" required></textarea>
    <button type="submit">Submit</button>
</form>
```
{% endcode %}

## Related Documentation

- [Background Sync](../the-service-worker/workbox/background-sync.md) - Service worker background sync configuration
- [Sync Broadcast](sync-broadcast.md) - Monitor sync queue status

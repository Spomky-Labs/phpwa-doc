# Protocol Handlers

Progressive Web Apps (PWAs) have the ability to handle web protocols, which means they can respond to specific URL schemes. For example, a PWA can register to handle `mailto:` links, so that clicking on an e-mail link can open a compose window in the PWA instead of opening the default mail client.

The application can declare custom protocols using the prefix `web+`.

With protocol handlers, your PWA can provide a more integrated user experience, functioning more like a native application.

### Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        protocol_handlers:
            - protocol: "mailto"
              url: "/compose?to=%s"
            - protocol: "web+custom"
              url:
                  path: "app_feature1"
                  params:
                      foo: "bar"
```
{% endcode %}

### Protocol Parameter

The `protocol` parameter specifies the protocol scheme your PWA can handle. It must follow these rules:

**Standard protocols:**
* `mailto` - Email addresses
* `tel` - Telephone numbers
* `sms` - SMS messages

**Custom protocols:**
* Must start with `web+` prefix
* Example: `web+jngl`, `web+music`, `web+game`

{% hint style="warning" %}
Custom protocols must use the `web+` prefix to be valid. Protocols without this prefix are reserved for standard web protocols.
{% endhint %}

### URL Parameter

The `url` parameter defines where to redirect when the protocol is triggered. The URL can include a placeholder `%s` that will be replaced with the protocol's argument.

**Simple string format:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "mailto"
      url: "/mail/compose?to=%s"
```
{% endcode %}

**Advanced object format:**

Just like with [shortcuts](shortcuts.md#url-parameter) and [start_url](application-information/start-url.md), you can use the full URL object configuration:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "web+recipe"
      url:
          path: "app_recipe_view"
          params:
              recipe_id: "%s"
              utm_source: "protocol_handler"
          path_type_reference: 1
```
{% endcode %}

The `path_type_reference` option expects an integer:
* `0`: Absolute URL (e.g., `https://app.com/foo/bar`)
* `1`: Absolute path (default, e.g., `/foo/bar`)
* `2`: Relative path (e.g., `../bar`)
* `3`: Network path (e.g., `//app.com/foo/bar`)

### Placeholder Parameter

When you need more control over the placeholder format, you can use the `placeholder` parameter. The bundle will automatically format it as `placeholder=%s`:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "web+jngl"
      placeholder: "type"
      url: "/handle?type=%s"
```
{% endcode %}

This allows you to create more semantic URL patterns for your protocol handlers.

### Considerations When Using Protocol Handlers

* Ensure that your PWA is served over HTTPS, as handling protocols can present security risks if not properly secured.
* Test the protocol handlers thoroughly on different browsers, as support for this feature can vary.
* Respect user choice. Always provide an easy way for users to opt-out of using the PWA as a default handler for specific protocols.

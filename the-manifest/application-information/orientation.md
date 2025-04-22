# Orientation

The `orientation` property in the Progressive Web Application (PWA) manifest dictates the default orientation that the web application will be displayed in. The allowed values are:

* `portrait-primary`: The orientation is in the primary portrait mode.
* `landscape-primary`: The orientation is in the primary landscape mode.
* `portrait`: Either of the portrait orientations.
* `landscape`: Either of the landscape orientations.
* `any`: Any orientation.

Setting the orientation helps ensure that the PWA looks good and functions well on mobile devices.

Example:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        orientation: "landscape"
```
{% endcode %}

With the above manifest configuration, the application will default to `landscape` orientation when launched from a device's home screen.

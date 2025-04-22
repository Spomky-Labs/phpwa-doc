# Development

## Toolbar

The Profiler Page is accessible via the web debug toolbar. It allows developers to quickly identify and fix configuration issues, monitor service performance, and understand how their application is behaving in detail.

The Manifest options, the Service Worker rules, the icons and other interesting information are showed on the tool pages.

{% hint style="info" %}
Please note that due to changing browser requirements, the bundle may incorrectly indicate that the app can be installed.
{% endhint %}

Depending on the manifest configuration, the bundle tries to guess if the PWA can be installed on the device.

<figure><img src=".gitbook/assets/2024-04-24_17h56_15.png" alt=""><figcaption><p>General view of the Manifest tab</p></figcaption></figure>

## Service Worker

It is quite difficult to debug the Service Worker. When in dev mode, Workbox will show debugging messages and the Service Worker file itself contains comments to help you understanding what is going wrong.

In production mode, debugging features and comments are removed.

## Logging

The bundle uses PSR-3 Logging implemention to send debugging messages.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    logger: 'my.psr3.service'
```
{% endcode %}

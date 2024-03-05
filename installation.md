# Installation

To install the bundle, the recommended way is using Composer:

```sh
composer require spomky-labs/pwa-bundle
```

No Flex recipe exists for this bundle, but you can create a configuration file that will be modified as you progress in the integration of the features.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa: ~
```
{% endcode %}

The integration in your application is very simple. You are only required to add a Twig function inside the end of the `<head>` tag of your HTML pages.

{% code lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
  {{ pwa() }}
</head>
<body>
  ...
</body>
</html>
```
{% endcode %}

{% hint style="info" %}
You may need to clear the cache after the bundle is installed
{% endhint %}

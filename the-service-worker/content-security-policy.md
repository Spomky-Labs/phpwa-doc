# Content Security Policy

If your application uses Content Security Policy (CSP) to declare and restrict script execution, the Service Worker will not be loaded as expected.

Fortunately, you are able to pass attributes such as a `nonce` to the script directive.

> **Note:** If using [Nelmio Security Bundle](https://symfony.com/bundles/NelmioSecurityBundle/current/index.html#content-security-policy), no configuration is needed. The nonce is automatically set.

{% code lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
  {{ pwa(swAttributes={nonce: "YOUR-NONCE-HERE"}) }}
</head>
<body>
  ...
</body>
</html>
```
{% endcode %}

# Content Security Policy

If your application uses Content Security Policy (CSP) to declare and restrict script execution, the Service Worker will not be loaded as expected.

Fortunately, you are able to pass attributes such as a `nonce` to the script directive.

In the example below, the nonce attribute is managed by [nelmio/security-bundle](https://symfony.com/bundles/NelmioSecurityBundle/current/index.html#content-security-policy).

{% code lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
  {{ pwa(swAttributes= {nonce: csp_nonce('script')}) }}
</head>
<body>
  ...
</body>
</html>
```
{% endcode %}

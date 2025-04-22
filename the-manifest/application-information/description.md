# Description

The `description` member is a string that allows the developer to describe the purpose of the web application. It serves as the accessible description of an installed web application.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        description: "This application helps you donate to worthy causes."
```
{% endcode %}

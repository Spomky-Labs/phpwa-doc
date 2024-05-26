# Debugging

The bundle uses PSR-3 Logging implemention to send debugging messages.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    logger: 'my.psr3.service'
```
{% endcode %}

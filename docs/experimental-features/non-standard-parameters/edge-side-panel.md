# EDGE Side Panel

Progressive Web Apps can opt-in to be pinned to the sidebar in Microsoft Edge.

The sidebar in Microsoft Edge allows users to easily access popular websites and utilities alongside their browser tabs. The content in the sidebar augments the user's primary task by enabling side-by-side browsing and minimizing the need to switch contexts between browser tabs.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        edge_side_panel: ~
```
{% endcode %}

You can tell EDGE your PWA should only be loaded in the sidebar and the minimum sidebar size.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        display: "browser"
        edge_side_panel:
            preferred_width: 400
```
{% endcode %}

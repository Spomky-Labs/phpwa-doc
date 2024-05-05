# Launch Handler

The Launch Handler API allows developers to control how your PWA is launched. For example if it uses an existing window or creates a new one, and how the app's target launch URL is handled.

This has one sub-field, `client_mode`, which contains a string value specifying how the app should be launched and navigated to.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        launch_handler:
            client_mode: "focus-existing"
```
{% endcode %}

If not specified, the default `client_mode` value is `auto`. Available values are:

* `focus-existing`: The most recently interacted with browsing context in a web app window is chosen to handle the launch. This will populate the target launch URL in the targetURL property of the LaunchParams object passed into the window.launchQueue.setConsumer()'s callback function. As you'll see below, this allows you to set custom launch handing functionality for your app.
* `navigate-existing`: The most recently interacted with browsing context in a web app window is navigated to the target launch URL. The target URL is still made available via window.launchQueue.setConsumer() to allow additional custom launch navigation handling to be implemented.
* `navigate-new`: A new browsing context is created in a web app window to load the target launch URL. The target URL is still made available via window.launchQueue.setConsumer() to allow additional custom launch navigation handling to be implemented.
* `auto`: The user agent decides what works best for the platform. For example, navigate-existing might make more sense on mobile, where single app instances are commonplace, whereas navigate-new might make more sense in a desktop context. This is the default value used if provided values are invalid.

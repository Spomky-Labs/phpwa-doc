# File Handlers

If your PWA is able to load certain type of files, you can declare in the Manifest file the supported MIME types and tell the host system you are able to manage those files.

To declare the supported file types, you can add a `file_handlers` entry to your web app manifest. This entry specifies the types of files that the app can open, along with an `action` URL that handles the opening of those files.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        file_handlers:
            - action: "/open"
              accept:
                  - "image/png": [".png"]
                  - "image/jpeg": [".jpg", ".jpeg"]
```
{% endcode %}

By adding a file handler for the above image types, your PWA will announce to the operating system that it can handle these types of files. When users open files with these extensions, your PWA will be suggested as an application to open them with.

The `action` property refers to the URL within the PWA context that will handle the file interaction. Ensure that your PWA is properly set up to handle file interactions at the specified URL.

The `action` property can be a relative URL, absolute URL or an route name. It is managed the same way as [the url parameter](shortcuts.md#url-parameter) showed in the shortcuts section.

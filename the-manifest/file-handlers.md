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
                  "image/png": [".png"]
                  "image/jpeg": [".jpg", ".jpeg"]
```
{% endcode %}

By adding a file handler for the above image types, your PWA will announce to the operating system that it can handle these types of files. When users open files with these extensions, your PWA will be suggested as an application to open them with.

## Action Parameter

The `action` property refers to the URL within the PWA context that will handle the file interaction. Ensure that your PWA is properly set up to handle file interactions at the specified URL.

The `action` property can be a simple string or a full URL object configuration, just like [shortcuts](shortcuts.md#url-parameter) and [start_url](application-information/start-url.md):

**Simple string format:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
file_handlers:
    - action: "/open-file"
      accept:
          "image/png": [".png"]
```
{% endcode %}

**Route name:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
file_handlers:
    - action: "app_file_handler"
      accept:
          "image/png": [".png"]
```
{% endcode %}

**Advanced object format with parameters:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
file_handlers:
    - action:
          path: "app_file_handler"
          params:
              mode: "edit"
              utm_source: "file_handler"
          path_type_reference: 1
      accept:
          "image/*": [".png", ".jpg", ".jpeg"]
```
{% endcode %}

## Accept Parameter

The `accept` parameter is an object that maps MIME types to arrays of file extensions. This tells the operating system which file types your PWA can handle.

### MIME Type Patterns

You can use specific MIME types or wildcards:

**Specific MIME types:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
file_handlers:
    - action: "/open"
      accept:
          "image/png": [".png"]
          "image/jpeg": [".jpg", ".jpeg"]
          "image/webp": [".webp"]
```
{% endcode %}

**Wildcard MIME types:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
file_handlers:
    - action: "/open"
      accept:
          "image/*": [".png", ".jpg", ".jpeg", ".gif", ".webp"]
          "text/*": [".txt", ".md", ".json"]
```
{% endcode %}

### Common File Types

Here are some common MIME type configurations:

**Images:**
```yaml
accept:
    "image/*": [".png", ".jpg", ".jpeg", ".gif", ".svg", ".webp"]
```

**Documents:**
```yaml
accept:
    "application/pdf": [".pdf"]
    "text/plain": [".txt"]
    "text/markdown": [".md"]
```

**Audio/Video:**
```yaml
accept:
    "audio/*": [".mp3", ".wav", ".ogg"]
    "video/*": [".mp4", ".webm", ".ogv"]
```

**Data files:**
```yaml
accept:
    "application/json": [".json"]
    "text/csv": [".csv"]
    "application/xml": [".xml"]
```

## Multiple File Handlers

You can register multiple file handlers for different file types:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        file_handlers:
            - action: "/edit-image"
              accept:
                  "image/*": [".png", ".jpg", ".jpeg", ".gif"]
            - action: "/edit-document"
              accept:
                  "text/plain": [".txt"]
                  "text/markdown": [".md"]
            - action: "/play-media"
              accept:
                  "audio/*": [".mp3", ".wav"]
                  "video/*": [".mp4", ".webm"]
```
{% endcode %}

## Best Practices

1. **Be specific**: Only declare file types your app can actually handle
2. **Provide fallbacks**: Use wildcards when appropriate, but also list specific extensions
3. **User experience**: Ensure your file handler page provides clear feedback about the opened file
4. **Security**: Validate and sanitize file content before processing
5. **Performance**: Handle large files efficiently with streaming or progressive loading

{% hint style="warning" %}
File handlers are an advanced PWA feature with varying browser support. Always check browser compatibility and provide alternative ways to upload or open files.
{% endhint %}

# File Handlers

File Handlers allow your PWA to register as a handler for specific file types at the operating system level. When properly configured, users can open files directly with your PWA by double-clicking them, using "Open with...", or dragging files onto your app icon.

## Overview

Implementing file handling in your PWA involves two steps:

1. **Manifest Configuration** (this page): Declare which file types your app supports
2. **Client-Side Implementation**: Use the [Symfony UX File Handling component](../symfony-ux/file-handling.md) to receive and process opened files

{% hint style="success" %}
**Looking for something else?**

* **Manifest configuration** (which file types to handle): You're on the right page!
* **Client-side implementation** (how to process files): See [File Handling Symfony UX](../symfony-ux/file-handling.md)
* **Examples**: Image viewers, document processors, media players - See [Symfony UX Examples](../symfony-ux/file-handling.md#usage)
{% endhint %}

This page covers the manifest configuration. For client-side implementation with complete examples, see the [File Handling Symfony UX documentation](../symfony-ux/file-handling.md).

## Quick Start

Here's a minimal example showing both configuration and implementation:

**1. Configure manifest** (`/config/packages/pwa.yaml`):
```yaml
pwa:
    manifest:
        file_handlers:
            - action: "app_image_viewer"
              accept:
                  "image/*": [".jpg", ".png", ".gif"]
```

**2. Create route** (`config/routes.yaml`):
```yaml
app_image_viewer:
    path: /viewer
    controller: App\Controller\ViewerController::index
```

**3. Add Stimulus controller** (`templates/viewer/index.html.twig`):
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <img id="preview" style="max-width: 100%;">
</div>

<script>
    document.addEventListener('pwa--file-handling:selected', (event) => {
        document.getElementById('preview').src = event.detail.data;
    });
</script>
```

Now when users double-click a `.jpg`, `.png`, or `.gif` file, your PWA opens and displays it!

For more advanced examples (multiple files, different file types, editing capabilities), see the [File Handling Symfony UX documentation](../symfony-ux/file-handling.md).

## Configuration

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

## Handling Files in Your Application

Once you've configured file handlers in your manifest, you need to implement the logic to receive and process files when users open them through the operating system. The PWA Bundle provides a Symfony UX component that makes this easy.

### Using the Symfony UX File Handling Controller

The `@pwa/file-handling` Stimulus controller automatically listens for files passed through the Launch Queue API and dispatches events you can handle in your application.

**Basic implementation:**

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div id="file-viewer"></div>
</div>

<script>
    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        // data contains a blob URL to the opened file
        console.log('File opened:', data);

        // Display image
        const img = document.createElement('img');
        img.src = data;
        document.getElementById('file-viewer').appendChild(img);
    });
</script>
```
{% endcode %}

### Complete Workflow

1. **Configure file handlers in manifest** (this page)
2. **Add the Stimulus controller** to your template
3. **Listen for the `pwa--file-handling:selected` event**
4. **Process the file data** as needed

{% hint style="info" %}
For complete examples and advanced usage patterns, including handling multiple files, different file types, and resource cleanup, see the [File Handling Symfony UX documentation](../symfony-ux/file-handling.md).
{% endhint %}

### When Files Are Received

Your application will receive files when users:
* Right-click a file and select "Open with..." your PWA
* Double-click a file associated with your PWA
* Drag a file onto your PWA icon in the taskbar/dock
* Select your PWA from the "Open with" menu

### Example: Image Editor Configuration

**Step 1 - Configure manifest:**
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        file_handlers:
            - action: "app_image_editor"
              accept:
                  "image/*": [".png", ".jpg", ".jpeg", ".gif", ".webp"]
```
{% endcode %}

**Step 2 - Create the route and template:**
{% code title="templates/image_editor.html.twig" lineNumbers="true" %}
```twig
{% extends 'base.html.twig' %}

{% block body %}
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <h1>Image Editor</h1>
    <canvas id="editor-canvas"></canvas>
</div>

<script>
    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        // Load image and display in canvas
        const img = new Image();
        img.onload = () => {
            const canvas = document.getElementById('editor-canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = img.width;
            canvas.height = img.height;
            ctx.drawImage(img, 0, 0);
        };
        img.src = data;
    });
</script>
{% endblock %}
```
{% endcode %}

See the [File Handling Symfony UX component documentation](../symfony-ux/file-handling.md) for more advanced examples including photo galleries, document viewers, media players, and code editors.

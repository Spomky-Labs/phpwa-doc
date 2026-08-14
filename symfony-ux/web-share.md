# Web Share

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Web Share component provides an interface to the Web Share API, allowing web applications to leverage the native share functionality of the user's device. This creates a seamless sharing experience by integrating with the device's built-in sharing options (social media apps, messaging apps, email, etc.).

This component is particularly useful for:

* Sharing articles, blog posts, or news content
* Sharing product pages in e-commerce applications
* Sharing user-generated content (photos, videos, text)
* Enabling social media sharing without third-party widgets
* Sharing files directly from your web application

## Browser Support

The Web Share API is supported in most modern mobile browsers and increasingly in desktop browsers. The API requires user interaction (e.g., click) to trigger the share dialog.

{% hint style="info" %}
Always check for API support before using it, and provide a fallback for browsers that don't support the Web Share API.
{% endhint %}

## Usage

### Basic Text and URL Sharing

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-share') }}>
    <article>
        <h1>Amazing Article Title</h1>
        <p>Read this amazing content...</p>

        <button {{ stimulus_action('@pwa/web-share', 'share', 'click', {
            title: 'Amazing Article Title',
            text: 'Check out this amazing article!',
            url: 'https://example.com/article/123'
        }) }}>
            Share Article
        </button>
    </article>
</div>

<script>
    document.addEventListener('pwa--web-share:success', () => {
        console.log('Content shared successfully!');
    });

    document.addEventListener('pwa--web-share:error', (event) => {
        console.error('Share failed:', event.detail.error);
    });
</script>
```
{% endcode %}

### Sharing Current Page

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-share') }}>
    <button {{ stimulus_action('@pwa/web-share', 'share', 'click', {
        title: document.title,
        text: 'Check out this page!',
        url: window.location.href
    }) }}>
        Share This Page
    </button>
</div>
```
{% endcode %}

### Sharing Files

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-share') }}>
    <input type="file" id="file-input" accept="image/*" multiple>

    <button id="share-files-btn">Share Selected Images</button>
</div>

<script>
    document.getElementById('share-files-btn').addEventListener('click', async function() {
        const fileInput = document.getElementById('file-input');
        const files = Array.from(fileInput.files);

        if (files.length === 0) {
            alert('Please select at least one file');
            return;
        }

        // Trigger share action
        const controller = document.querySelector('[data-controller="@pwa/web-share"]');
        controller.dispatchEvent(new CustomEvent('share', {
            detail: {
                params: {
                    data: {
                        title: 'My Photos',
                        text: 'Check out these photos!',
                        files: files
                    }
                }
            }
        }));
    });

    document.addEventListener('pwa--web-share:success', () => {
        alert('Files shared successfully!');
    });

    document.addEventListener('pwa--web-share:error', (event) => {
        alert('Failed to share files: ' + event.detail.error.message);
    });
</script>
```
{% endcode %}

### Social Media Share with Fallback

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-share') }}>
    <article class="blog-post">
        <h1>{{ post.title }}</h1>
        <div class="content">{{ post.content }}</div>

        <div class="share-buttons">
            <!-- Native share button -->
            <button
                id="native-share-btn"
                class="btn-share-native"
                {{ stimulus_action('@pwa/web-share', 'share', 'click', {
                    title: post.title,
                    text: post.excerpt,
                    url: url('post_show', {id: post.id})
                }) }}
            >
                <span class="icon">📤</span> Share
            </button>

            <!-- Fallback for browsers without Web Share API -->
            <div id="share-fallback" class="share-fallback" style="display:none;">
                <a href="https://twitter.com/intent/tweet?text={{ post.title|url_encode }}&url={{ url('post_show', {id: post.id})|url_encode }}" target="_blank">
                    Twitter
                </a>
                <a href="https://www.facebook.com/sharer/sharer.php?u={{ url('post_show', {id: post.id})|url_encode }}" target="_blank">
                    Facebook
                </a>
            </div>
        </div>
    </article>
</div>

<script>
    // Show fallback if Web Share API is not supported
    if (!navigator.share) {
        document.getElementById('native-share-btn').style.display = 'none';
        document.getElementById('share-fallback').style.display = 'flex';
    }

    document.addEventListener('pwa--web-share:error', (event) => {
        // Show fallback on error if not just user cancellation
        if (event.detail.error.name !== 'AbortError') {
            document.getElementById('native-share-btn').style.display = 'none';
            document.getElementById('share-fallback').style.display = 'flex';
        }
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `share`

Triggers the native share dialog with the provided data. The action requires a `data` parameter containing at least one of the following:

**Required parameters (at least one):**
* `title` (string): The title of the content to be shared
* `text` (string): Text content to be shared
* `url` (string): URL to be shared
* `files` (Array of File objects): Files to be shared

```twig
{{ stimulus_action('@pwa/web-share', 'share', 'click', {
    title: 'Page Title',
    text: 'Description text',
    url: 'https://example.com'
}) }}
```

{% hint style="info" %}
The `title` parameter may be ignored by some sharing targets. Not all apps will display the title you provide.
{% endhint %}

{% hint style="warning" %}
When sharing files, the browser will download them before sharing. **Ensure files are small and of shareable types**. Refer to [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share#shareable_file_types) for acceptable file types.
{% endhint %}

### Shareable File Types

The following file types are commonly supported for sharing:
* Images: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`
* Videos: `.mp4`, `.webm`
* Audio: `.mp3`, `.wav`, `.ogg`
* Documents: `.pdf`, `.txt`

Support varies by browser and operating system.

## Targets

None

## Events

### `pwa--web-share:success`

Dispatched when content is successfully shared (user completed the share action).

**No payload**

Example:
```javascript
document.addEventListener('pwa--web-share:success', () => {
    console.log('Content shared successfully!');
    // Show thank you message
    // Track analytics
});
```

{% hint style="info" %}
This event fires when the user selects a sharing target, not necessarily when the content is actually sent to that target.
{% endhint %}

### `pwa--web-share:error`

Dispatched when sharing fails or is cancelled by the user.

**Payload**: `{error}` - Error object or message

Example:
```javascript
document.addEventListener('pwa--web-share:error', (event) => {
    const error = event.detail.error;
    console.error('Share failed:', error);

    // Handle specific errors
    if (error.name === 'AbortError') {
        // User cancelled the share
        console.log('User cancelled sharing');
    } else if (error.name === 'NotSupportedError') {
        // API not supported, show fallback
        showFallbackShareOptions();
    }
});
```

## Best Practices

1. **Check for support**: Always verify browser support before showing share buttons
2. **Require user interaction**: The share dialog can only be triggered by user actions (clicks, touches)
3. **Provide fallbacks**: Offer traditional share buttons for browsers without Web Share API support
4. **Keep it simple**: Avoid overusing share buttons; place them strategically
5. **Validate URLs**: Ensure URLs are absolute and properly formatted
6. **File size limits**: Keep shared files small to avoid long downloads
7. **Handle errors gracefully**: Always listen for error events and provide feedback

## Error Handling

Common errors you might encounter:

* **AbortError**: User cancelled the share dialog
* **NotAllowedError**: Share was not triggered by user interaction
* **NotSupportedError**: Browser doesn't support Web Share API
* **TypeError**: Invalid data provided (e.g., malformed URL)
* **DataError**: Issues with file sharing (unsupported type, too large, etc.)

## Complete Example

Here's a complete example with proper fallback handling:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-share') }}>
    <article class="content">
        <h1 id="content-title">My Awesome Content</h1>
        <p id="content-description">This is amazing content you should share!</p>

        <!-- Native share button -->
        <button
            id="native-share-btn"
            {{ stimulus_action('@pwa/web-share', 'share', 'click', {
                title: 'My Awesome Content',
                text: 'Check this out!',
                url: 'https://example.com/content/123'
            }) }}
            class="btn-primary"
        >
            📤 Share
        </button>

        <!-- Fallback share options -->
        <div id="fallback-share" style="display:none;">
            <button onclick="shareToTwitter()">🐦 Twitter</button>
            <button onclick="shareToFacebook()">📘 Facebook</button>
            <button onclick="copyToClipboard()">🔗 Copy Link</button>
        </div>

        <!-- Status messages -->
        <div id="share-status" class="status-message"></div>
    </article>
</div>

<script>
    const nativeBtn = document.getElementById('native-share-btn');
    const fallbackDiv = document.getElementById('fallback-share');
    const statusDiv = document.getElementById('share-status');

    // Check for Web Share API support
    if (!navigator.share) {
        nativeBtn.style.display = 'none';
        fallbackDiv.style.display = 'flex';
    }

    document.addEventListener('pwa--web-share:success', () => {
        statusDiv.textContent = '✓ Shared successfully!';
        statusDiv.className = 'status-message success';
        setTimeout(() => statusDiv.textContent = '', 3000);
    });

    document.addEventListener('pwa--web-share:error', (event) => {
        const error = event.detail.error;

        if (error.name === 'AbortError') {
            // User cancelled - don't show error
            return;
        }

        statusDiv.textContent = '✗ Share failed. Please try another method.';
        statusDiv.className = 'status-message error';

        // Show fallback options
        nativeBtn.style.display = 'none';
        fallbackDiv.style.display = 'flex';
    });

    function shareToTwitter() {
        const text = encodeURIComponent('Check this out!');
        const url = encodeURIComponent('https://example.com/content/123');
        window.open(`https://twitter.com/intent/tweet?text=${text}&url=${url}`, '_blank');
    }

    function shareToFacebook() {
        const url = encodeURIComponent('https://example.com/content/123');
        window.open(`https://www.facebook.com/sharer/sharer.php?u=${url}`, '_blank');
    }

    async function copyToClipboard() {
        try {
            await navigator.clipboard.writeText('https://example.com/content/123');
            statusDiv.textContent = '✓ Link copied to clipboard!';
            statusDiv.className = 'status-message success';
            setTimeout(() => statusDiv.textContent = '', 3000);
        } catch (err) {
            statusDiv.textContent = '✗ Failed to copy link';
            statusDiv.className = 'status-message error';
        }
    }
</script>
```
{% endcode %}

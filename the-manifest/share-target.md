# Share Target

The Web Share Target API allows your PWA to receive content shared from other applications, making it appear alongside native apps in the system's share menu. Users can share text, links, and files directly to your PWA.

## Overview

Share Target enables:
- **Content reception**: Receive shared content from other apps
- **Native integration**: Appear in OS share sheet
- **File handling**: Accept images, documents, and other files
- **Seamless UX**: Process shared content within your PWA

## How It Works

1. **User shares content**: From another app, user clicks "Share"
2. **Your PWA appears**: Listed in the share menu
3. **User selects PWA**: Chooses your app as destination
4. **PWA opens**: Your app receives the shared content
5. **Content processed**: Handle the shared data in your application

## Browser Support

{% hint style="info" %}
**Good Support**: Web Share Target is supported in Chrome, Edge, and Samsung Internet on Android. Limited support on Desktop and iOS.
{% endhint %}

**Check support**:
- Chrome/Edge: Full support (Android, Desktop)
- Safari: Limited support
- Firefox: Not supported yet

## Basic Configuration

### Sharing Text and URLs

Simple configuration to receive text and links:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/share"
            method: "GET"
            params:
                title: "title"
                text: "text"
                url: "url"
```
{% endcode %}

**How it works**: When content is shared, your PWA opens at `/share?title=...&text=...&url=...`

### Sharing Files

Accept image files via POST:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/share-image"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                files:
                    - name: "image"
                      accept: ["image/*"]
```
{% endcode %}

### Complete Configuration

Handle text, URLs, and multiple file types:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/share-content"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                title: "title"
                text: "text"
                url: "url"
                files:
                    - name: "media"
                      accept:
                          - "image/*"
                          - "video/*"
                          - "audio/*"
```
{% endcode %}

## Configuration Parameters

### `action`

**Type**: `string` or URL object
**Required**: Yes

The URL endpoint that handles shared content.

```yaml
# Simple path
action: "/share"

# Route name
action:
    route: "app_share"

# With parameters
action:
    route: "app_share"
    params:
        type: "media"
```

### `method`

**Type**: `string`
**Values**: `GET` or `POST`
**Default**: `GET`

HTTP method for sharing.

**Use GET when**:
- Sharing only text/URLs (no files)
- Content is read-only
- No data mutation

**Use POST when**:
- Sharing files
- Creating new entries (bookmarks, posts)
- Mutating application state

```yaml
# GET for simple text/URL sharing
method: "GET"

# POST for file sharing
method: "POST"
```

### `enctype`

**Type**: `string`
**Values**: `application/x-www-form-urlencoded` or `multipart/form-data`
**Default**: `application/x-www-form-urlencoded`

Encoding type for POST requests. Use `multipart/form-data` when accepting files.

```yaml
# For file uploads (required)
enctype: "multipart/form-data"

# For text-only POST (default)
enctype: "application/x-www-form-urlencoded"
```

### `params`

**Type**: `object`

Defines parameter names for shared content.

#### Text Parameters

```yaml
params:
    title: "title"      # Query param name for shared title
    text: "text"        # Query param name for shared text
    url: "url"          # Query param name for shared URL
```

#### File Parameters

```yaml
params:
    files:
        - name: "photos"        # Form field name
          accept:               # Accepted types
              - "image/png"
              - "image/jpeg"
              - "image/gif"
              - "image/webp"
```

## Symfony Implementation

### Controller for Text/URL Sharing (GET)

{% code title="src/Controller/ShareController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ShareController extends AbstractController
{
    #[Route('/share', name: 'app_share', methods: ['GET'])]
    public function share(Request $request): Response
    {
        $title = $request->query->get('title', '');
        $text = $request->query->get('text', '');
        $url = $request->query->get('url', '');

        // Process the shared content
        // For example, pre-fill a form or create a new post

        return $this->render('share/index.html.twig', [
            'title' => $title,
            'text' => $text,
            'url' => $url,
        ]);
    }
}
```
{% endcode %}

### Controller for File Sharing (POST)

{% code title="src/Controller/ShareController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\File\UploadedFile;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ShareController extends AbstractController
{
    #[Route('/share-image', name: 'app_share_image', methods: ['POST'])]
    public function shareImage(Request $request): Response
    {
        /** @var UploadedFile|null $image */
        $image = $request->files->get('image');

        if (!$image) {
            return $this->redirectToRoute('app_home');
        }

        // Validate the file
        if (!$image->isValid()) {
            $this->addFlash('error', 'Invalid file upload');
            return $this->redirectToRoute('app_home');
        }

        // Process the image (save, resize, etc.)
        $filename = $this->saveSharedImage($image);

        // Redirect to avoid resubmission (303 See Other)
        return $this->redirectToRoute('app_image_view', [
            'filename' => $filename
        ], Response::HTTP_SEE_OTHER);
    }

    private function saveSharedImage(UploadedFile $image): string
    {
        $filename = uniqid() . '.' . $image->guessExtension();
        $image->move(
            $this->getParameter('uploads_directory'),
            $filename
        );

        return $filename;
    }
}
```
{% endcode %}

### Complete Handler (Text + Files)

{% code title="src/Controller/ShareController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use App\Entity\Post;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ShareController extends AbstractController
{
    #[Route('/share-content', name: 'app_share_content', methods: ['POST'])]
    public function shareContent(
        Request $request,
        EntityManagerInterface $em
    ): Response {
        // Get text parameters
        $title = $request->request->get('title', '');
        $text = $request->request->get('text', '');
        $url = $request->request->get('url', '');

        // Get shared files
        $media = $request->files->get('media');

        // Create a new post with shared content
        $post = new Post();
        $post->setTitle($title ?: 'Shared Content');
        $post->setContent($text);
        $post->setSourceUrl($url);

        // Handle media files
        if ($media && $media->isValid()) {
            $filename = $this->handleMediaFile($media);
            $post->setMediaFile($filename);
        }

        $em->persist($post);
        $em->flush();

        // Redirect with 303 See Other
        return $this->redirectToRoute('app_post_view', [
            'id' => $post->getId()
        ], Response::HTTP_SEE_OTHER);
    }

    private function handleMediaFile($file): string
    {
        // Handle image, video, or audio based on MIME type
        $mimeType = $file->getMimeType();

        if (str_starts_with($mimeType, 'image/')) {
            return $this->handleImage($file);
        } elseif (str_starts_with($mimeType, 'video/')) {
            return $this->handleVideo($file);
        } elseif (str_starts_with($mimeType, 'audio/')) {
            return $this->handleAudio($file);
        }

        throw new \InvalidArgumentException('Unsupported file type');
    }
}
```
{% endcode %}

## Common Use Cases

### 1. Social Media: Share Posts

Accept text and images for new posts:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/create-post"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                title: "title"
                text: "content"
                url: "link"
                files:
                    - name: "photo"
                      accept: ["image/*"]
```
{% endcode %}

### 2. Bookmarking App

Save shared URLs:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/save-bookmark"
            method: "POST"
            enctype: "application/x-www-form-urlencoded"
            params:
                title: "title"
                url: "url"
                text: "notes"
```
{% endcode %}

{% code title="src/Controller/BookmarkController.php" lineNumbers="true" %}
```php
#[Route('/save-bookmark', name: 'app_save_bookmark', methods: ['POST'])]
public function saveBookmark(Request $request, EntityManagerInterface $em): Response
{
    $bookmark = new Bookmark();
    $bookmark->setTitle($request->request->get('title'));
    $bookmark->setUrl($request->request->get('url'));
    $bookmark->setNotes($request->request->get('notes'));
    $bookmark->setCreatedAt(new \DateTime());

    $em->persist($bookmark);
    $em->flush();

    return $this->redirectToRoute('app_bookmarks', [], Response::HTTP_SEE_OTHER);
}
```
{% endcode %}

### 3. Note-Taking App

Capture shared text and links:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/new-note"
            method: "GET"
            params:
                title: "title"
                text: "content"
                url: "source"
```
{% endcode %}

### 4. Photo Gallery

Accept multiple images:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/upload-photos"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                files:
                    - name: "photos[]"
                      accept:
                          - "image/jpeg"
                          - "image/png"
                          - "image/webp"
                          - "image/heic"
```
{% endcode %}

### 5. Music Player

Share audio files:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        share_target:
            action: "/add-to-playlist"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                files:
                    - name: "audio"
                      accept:
                          - "audio/mpeg"
                          - "audio/mp4"
                          - "audio/ogg"
                          - "audio/wav"
```
{% endcode %}

## Frontend Handling

### Pre-fill Form with Shared Data

{% code title="templates/share/index.html.twig" lineNumbers="true" %}
```twig
{% extends 'base.html.twig' %}

{% block body %}
<div class="container">
    <h1>Share to Our App</h1>

    <form action="{{ path('app_create_post') }}" method="post">
        <div class="form-group">
            <label for="title">Title</label>
            <input type="text"
                   id="title"
                   name="title"
                   class="form-control"
                   value="{{ title }}">
        </div>

        <div class="form-group">
            <label for="content">Content</label>
            <textarea id="content"
                      name="content"
                      class="form-control"
                      rows="5">{{ text }}</textarea>
        </div>

        {% if url %}
        <div class="form-group">
            <label for="source">Source URL</label>
            <input type="url"
                   id="source"
                   name="source"
                   class="form-control"
                   value="{{ url }}"
                   readonly>
        </div>
        {% endif %}

        <button type="submit" class="btn btn-primary">Share</button>
    </form>
</div>
{% endblock %}
```
{% endcode %}

### Handle Shared Files with JavaScript

{% code title="templates/share/upload.html.twig" lineNumbers="true" %}
```twig
<form id="shareForm" method="post" enctype="multipart/form-data">
    <div id="preview"></div>
    <button type="submit">Upload</button>
</form>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const form = document.getElementById('shareForm');
    const preview = document.getElementById('preview');

    // Check if there's a shared file
    const urlParams = new URLSearchParams(window.location.search);
    if (urlParams.has('file')) {
        // File was shared
        showFilePreview();
    }

    form.addEventListener('submit', async (e) => {
        e.preventDefault();

        const formData = new FormData(form);

        const response = await fetch(form.action, {
            method: 'POST',
            body: formData
        });

        if (response.ok) {
            window.location.href = await response.text();
        }
    });
});

function showFilePreview() {
    // Display preview of shared file
    const preview = document.getElementById('preview');
    preview.innerHTML = '<p>File received! Processing...</p>';
}
</script>
```
{% endcode %}

## File Type Handling

### Accept Specific Image Formats

```yaml
params:
    files:
        - name: "image"
          accept:
              - "image/png"
              - "image/jpeg"
              - "image/webp"
```

### Accept Documents

```yaml
params:
    files:
        - name: "document"
          accept:
              - "application/pdf"
              - ".doc"
              - ".docx"
              - "application/vnd.ms-excel"
              - "text/plain"
```

### Accept Videos

```yaml
params:
    files:
        - name: "video"
          accept:
              - "video/mp4"
              - "video/webm"
              - "video/ogg"
```

### Multiple File Types

```yaml
params:
    files:
        - name: "media"
          accept:
              - "image/*"
              - "video/*"
        - name: "docs"
          accept:
              - "application/pdf"
              - ".doc"
              - ".docx"
```

## Testing Share Target

### Manual Testing on Android

1. **Install your PWA** on Android device
2. **Open another app** (e.g., Chrome, Gallery)
3. **Find content to share** (webpage, photo)
4. **Tap Share button**
5. **Select your PWA** from share sheet
6. **Verify content** is received correctly

### Testing in Chrome DevTools

Share Target can be tested using Chrome DevTools on desktop:

```bash
1. Open your PWA
2. Open DevTools (F12)
3. Go to Application → Manifest
4. Click "Share Target" section
5. Use "Test" button to simulate sharing
```

### Programmatic Testing

Test your share endpoint directly:

```bash
# Test GET sharing (text/URL)
curl "http://localhost:8000/share?title=Test&text=Hello&url=https://example.com"

# Test POST sharing (file)
curl -X POST http://localhost:8000/share-image \
  -F "image=@test-image.jpg"
```

## Best Practices

### 1. Use HTTP 303 See Other for POST

Prevent double submissions:

```php
// ✓ Correct - Use 303
return $this->redirectToRoute('app_success', [], Response::HTTP_SEE_OTHER);

// ✗ Wrong - Use default 302
return $this->redirectToRoute('app_success');
```

### 2. Validate Shared Content

Always validate received data:

```php
public function shareContent(Request $request): Response
{
    // Validate text length
    $text = $request->request->get('text', '');
    if (strlen($text) > 5000) {
        $this->addFlash('error', 'Text too long');
        return $this->redirectToRoute('app_home');
    }

    // Validate URL
    $url = $request->request->get('url', '');
    if ($url && !filter_var($url, FILTER_VALIDATE_URL)) {
        $this->addFlash('error', 'Invalid URL');
        return $this->redirectToRoute('app_home');
    }

    // Validate file
    $file = $request->files->get('image');
    if ($file && !$file->isValid()) {
        $this->addFlash('error', 'Invalid file');
        return $this->redirectToRoute('app_home');
    }

    // Process valid content...
}
```

### 3. Handle Missing Parameters

Content might be incomplete:

```php
public function share(Request $request): Response
{
    $title = $request->query->get('title', '');
    $text = $request->query->get('text', '');
    $url = $request->query->get('url', '');

    // At least one parameter should be present
    if (empty($title) && empty($text) && empty($url)) {
        return $this->redirectToRoute('app_home');
    }

    // Use fallbacks
    $title = $title ?: 'Shared Content';

    // Process...
}
```

### 4. Provide User Feedback

Show success/error messages:

```php
public function shareImage(Request $request): Response
{
    try {
        // Process image
        $filename = $this->saveSharedImage($request->files->get('image'));

        $this->addFlash('success', 'Image shared successfully!');

        return $this->redirectToRoute('app_image_view', [
            'filename' => $filename
        ], Response::HTTP_SEE_OTHER);
    } catch (\Exception $e) {
        $this->addFlash('error', 'Failed to share image: ' . $e->getMessage());

        return $this->redirectToRoute('app_home', [], Response::HTTP_SEE_OTHER);
    }
}
```

### 5. Authenticate Users

Require authentication for sharing:

```php
#[Route('/share-content', name: 'app_share_content')]
#[IsGranted('ROLE_USER')]
public function shareContent(Request $request): Response
{
    // User must be logged in to share content
    // ...
}
```

Or redirect to login:

```php
public function shareContent(Request $request): Response
{
    if (!$this->getUser()) {
        // Store shared content in session
        $this->get('session')->set('pending_share', [
            'title' => $request->request->get('title'),
            'text' => $request->request->get('text'),
            'url' => $request->request->get('url'),
        ]);

        return $this->redirectToRoute('app_login');
    }

    // Process share...
}
```

## Debugging

### Check Manifest

Verify share_target in manifest:

```bash
1. Open your PWA
2. DevTools (F12) → Application → Manifest
3. Check "share_target" section
4. Verify action, method, and params
```

### Log Received Data

Debug what your app receives:

```php
public function share(Request $request): Response
{
    // Log all received data
    $this->logger->info('Share received', [
        'query' => $request->query->all(),
        'request' => $request->request->all(),
        'files' => array_keys($request->files->all()),
    ]);

    // ...
}
```

### Test Endpoint Directly

Test without share sheet:

```html
<!-- Create test form -->
<form action="/share-image" method="post" enctype="multipart/form-data">
    <input type="file" name="image" accept="image/*">
    <button type="submit">Test Share</button>
</form>
```

## Limitations

### Browser Support

- **Android**: Full support in Chrome, Edge, Samsung Internet
- **iOS/Safari**: Not supported yet
- **Desktop**: Limited (must install PWA first)
- **Firefox**: Not supported

### Platform Restrictions

- Requires PWA to be installed
- Only works on supported platforms
- File size limits apply
- Some MIME types may not work on all devices

## Security Considerations

### Validate File Types

```php
public function shareImage(Request $request): Response
{
    $image = $request->files->get('image');

    // Check MIME type
    $allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
    if (!in_array($image->getMimeType(), $allowedTypes)) {
        throw new \InvalidArgumentException('Invalid file type');
    }

    // Check file extension
    $extension = $image->guessExtension();
    if (!in_array($extension, ['jpg', 'jpeg', 'png', 'webp'])) {
        throw new \InvalidArgumentException('Invalid file extension');
    }

    // Process...
}
```

### Limit File Size

```php
public function shareImage(Request $request): Response
{
    $image = $request->files->get('image');

    // Check file size (10MB max)
    if ($image->getSize() > 10 * 1024 * 1024) {
        throw new \InvalidArgumentException('File too large');
    }

    // Process...
}
```

### Sanitize Input

```php
use Symfony\Component\String\Slugger\SluggerInterface;

public function share(Request $request, SluggerInterface $slugger): Response
{
    $title = $request->query->get('title', '');
    $text = $request->query->get('text', '');

    // Sanitize title
    $title = strip_tags($title);
    $title = $slugger->slug($title);

    // Sanitize text (allow some HTML)
    $text = strip_tags($text, '<p><br><strong><em>');

    // Process...
}
```

## Related Documentation

- [Shortcuts](shortcuts.md) - App shortcuts configuration
- [File Handlers](file-handlers.md) - Handle file types
- [Icons](icons.md) - App icons for share sheet

## Resources

- **MDN Web Share Target**: https://developer.mozilla.org/en-US/docs/Web/Manifest/share_target
- **Web.dev Article**: https://web.dev/web-share-target/
- **Chrome Developers**: https://developer.chrome.com/docs/capabilities/web-apis/web-share-target
- **W3C Spec**: https://w3c.github.io/web-share-target/


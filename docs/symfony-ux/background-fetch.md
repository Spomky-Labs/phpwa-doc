# Background Fetch

The Background Fetch component provides a powerful way to download large files in the background, even when the user closes your application. This component wraps the Background Fetch API and manages file downloads through the service worker, with automatic storage in IndexedDB for offline access. It handles download progress tracking, cancellation, and retrieval of completed downloads.

This component is particularly useful for:

* Downloading large media files (videos, podcasts, high-resolution images)
* Enabling offline access to content (movies, music, documents)
* Bulk downloading of resources for offline use
* Providing podcast or video download features
* Creating offline-first applications with large assets
* Building progressive web apps with downloadable content
* Implementing reliable download managers within web applications
* Handling file downloads that may take longer than the page lifetime

## Browser Support

The Background Fetch API is a relatively new feature with limited but growing support.

**Support level:** Limited - Currently supported in Chromium-based browsers (Chrome, Edge, Opera) on Android and desktop. Not supported in Safari or Firefox as of early 2025.

{% hint style="warning" %}
Always check for Background Fetch API support before using this component. The component will emit an `unsupported` event when the API is not available.
{% endhint %}

{% hint style="info" %}
Background Fetch requires a registered service worker to function. Ensure your PWA has an active service worker before using this component.
{% endhint %}

## Usage

### Basic File Download

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <button {{ stimulus_action('@pwa/background-fetch', 'download', {
    id: 'video-download-1',
    url: '/assets/videos/tutorial.mp4',
    title: 'Downloading Tutorial Video',
    downloadTotal: 52428800
  }) }}>
    Download Tutorial Video (50MB)
  </button>
  <div id="download-status"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const status = document.getElementById('download-status');

  host.addEventListener('background-fetch:unsupported', () => {
    status.innerHTML = '<p class="text-red-600">Background downloads not supported in this browser</p>';
  });

  host.addEventListener('background-fetch:started', (e) => {
    const { id, title } = e.detail;
    status.innerHTML = `<p class="text-blue-600">Download started: ${title}</p>`;
  });

  host.addEventListener('background-fetch:in-progress', (e) => {
    const { id, downloaded, downloadTotal } = e.detail;
    const percent = ((downloaded / downloadTotal) * 100).toFixed(1);
    status.innerHTML = `
      <div class="progress-bar">
        <div class="progress-fill" style="width: ${percent}%"></div>
      </div>
      <p>${percent}% (${(downloaded / 1024 / 1024).toFixed(1)}MB / ${(downloadTotal / 1024 / 1024).toFixed(1)}MB)</p>
    `;
  });

  host.addEventListener('background-fetch:completed', (e) => {
    const { id, name } = e.detail;
    status.innerHTML = `<p class="text-green-600">Download completed: ${name}</p>`;
  });

  host.addEventListener('background-fetch:failed', (e) => {
    status.innerHTML = `<p class="text-red-600">Download failed</p>`;
  });
</script>

<style>
  .progress-bar {
    width: 100%;
    height: 20px;
    background: #e0e0e0;
    border-radius: 10px;
    overflow: hidden;
  }
  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #4A90E2, #357ABD);
    transition: width 0.3s ease;
  }
</style>
```
{% endcode %}

### Multiple File Downloads

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <button {{ stimulus_action('@pwa/background-fetch', 'download', {
    id: 'podcast-series',
    url: [
      '/podcasts/episode-1.mp3',
      '/podcasts/episode-2.mp3',
      '/podcasts/episode-3.mp3'
    ],
    title: 'Downloading Podcast Series',
    icons: [{ src: '/icons/podcast.png', sizes: '192x192' }],
    downloadTotal: 157286400
  }) }}>
    Download All Episodes (150MB)
  </button>
  <div id="multi-download-status"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const status = document.getElementById('multi-download-status');

  host.addEventListener('background-fetch:started', (e) => {
    const { urls, title } = e.detail;
    status.innerHTML = `
      <div class="download-card">
        <h3>${title}</h3>
        <p>Files: ${urls.length}</p>
        <div class="progress-container"></div>
      </div>
    `;
  });

  host.addEventListener('background-fetch:in-progress', (e) => {
    const { downloaded, downloadTotal } = e.detail;
    const percent = ((downloaded / downloadTotal) * 100).toFixed(1);
    const container = document.querySelector('.progress-container');
    container.innerHTML = `
      <div class="progress-bar">
        <div class="progress-fill" style="width: ${percent}%"></div>
      </div>
      <p class="text-sm">${percent}% complete</p>
    `;
  });

  host.addEventListener('background-fetch:completed', (e) => {
    status.innerHTML = `
      <div class="download-card success">
        <h3>Download Complete!</h3>
        <p>All episodes are now available offline</p>
      </div>
    `;
  });
</script>
```
{% endcode %}

### Download Management with Cancel

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <div id="download-manager">
    <button id="start-download" {{ stimulus_action('@pwa/background-fetch', 'download', {
      id: 'large-file',
      url: '/assets/movie.mp4',
      title: 'Downloading Movie',
      downloadTotal: 1073741824
    }) }}>
      Start Download (1GB)
    </button>
    <button id="cancel-download" class="hidden" {{ stimulus_action('@pwa/background-fetch', 'cancel', {
      id: 'large-file'
    }) }}>
      Cancel Download
    </button>
  </div>
  <div id="manager-status"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const startBtn = document.getElementById('start-download');
  const cancelBtn = document.getElementById('cancel-download');
  const status = document.getElementById('manager-status');

  host.addEventListener('background-fetch:started', () => {
    startBtn.classList.add('hidden');
    cancelBtn.classList.remove('hidden');
    status.innerHTML = '<p>Download in progress...</p>';
  });

  host.addEventListener('background-fetch:in-progress', (e) => {
    const { downloaded, downloadTotal } = e.detail;
    const percent = ((downloaded / downloadTotal) * 100).toFixed(1);
    status.innerHTML = `
      <div class="flex items-center gap-4">
        <div class="flex-1 bg-gray-200 rounded-full h-4">
          <div class="bg-blue-600 h-4 rounded-full transition-all" style="width: ${percent}%"></div>
        </div>
        <span class="text-sm font-medium">${percent}%</span>
      </div>
      <p class="text-sm text-gray-600 mt-2">
        ${(downloaded / 1024 / 1024).toFixed(0)}MB / ${(downloadTotal / 1024 / 1024).toFixed(0)}MB
      </p>
    `;
  });

  host.addEventListener('background-fetch:aborted', () => {
    startBtn.classList.remove('hidden');
    cancelBtn.classList.add('hidden');
    status.innerHTML = '<p class="text-yellow-600">Download cancelled</p>';
  });

  host.addEventListener('background-fetch:completed', () => {
    startBtn.classList.add('hidden');
    cancelBtn.classList.add('hidden');
    status.innerHTML = '<p class="text-green-600">Download completed!</p>';
  });

  host.addEventListener('background-fetch:exists', (e) => {
    status.innerHTML = '<p class="text-blue-600">This download is already in progress</p>';
  });
</script>
```
{% endcode %}

### Retrieving Downloaded Files

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <h2>My Downloaded Files</h2>
  <div id="files-list"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const filesList = document.getElementById('files-list');

  // Listen for completed downloads
  host.addEventListener('background-fetch:completed', (e) => {
    addFileToList(e.detail);
  });

  function addFileToList(file) {
    const fileDiv = document.createElement('div');
    fileDiv.className = 'file-item p-4 border rounded mb-2 flex justify-between items-center';
    fileDiv.innerHTML = `
      <div>
        <p class="font-semibold">${file.name || 'Downloaded File'}</p>
        <p class="text-sm text-gray-600">${(file.size / 1024 / 1024).toFixed(2)}MB</p>
      </div>
      <div class="flex gap-2">
        <button class="btn-open bg-blue-500 text-white px-4 py-2 rounded">Open</button>
        <button class="btn-download bg-green-500 text-white px-4 py-2 rounded">Download</button>
        <button class="btn-delete bg-red-500 text-white px-4 py-2 rounded">Delete</button>
      </div>
    `;

    // Open file in new tab
    fileDiv.querySelector('.btn-open').addEventListener('click', () => {
      host.controller.get({
        preventDefault: () => {},
        params: { url: file.id }
      });
    });

    // Download file
    fileDiv.querySelector('.btn-download').addEventListener('click', () => {
      host.controller.get({
        preventDefault: () => {},
        params: { url: file.id, name: file.name }
      });
    });

    // Delete file
    fileDiv.querySelector('.btn-delete').addEventListener('click', async () => {
      await host.controller.delete({ params: { url: file.id } });
      fileDiv.remove();
    });

    filesList.appendChild(fileDiv);
  }
</script>
```
{% endcode %}

## Best Practices

### Always Provide downloadTotal

{% hint style="success" %}
Always specify the `downloadTotal` parameter when starting a background fetch. This allows the browser to display accurate progress information to the user in the system UI.
{% endhint %}

{% code lineNumbers="true" %}
```twig
{# Good - with downloadTotal #}
<button {{ stimulus_action('@pwa/background-fetch', 'download', {
  id: 'file-1',
  url: '/large-file.zip',
  title: 'Downloading File',
  downloadTotal: 104857600  {# 100MB #}
}) }}>Download</button>

{# Less ideal - without downloadTotal (progress will be indeterminate) #}
<button {{ stimulus_action('@pwa/background-fetch', 'download', {
  id: 'file-2',
  url: '/unknown-size.zip',
  title: 'Downloading File'
}) }}>Download</button>
```
{% endcode %}

### Use Unique IDs

{% hint style="warning" %}
Each background fetch requires a unique ID. If you attempt to start a fetch with an ID that's already in use, the component will emit an `exists` event instead of starting a new download.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
// Generate unique IDs for downloads
const downloadId = `download-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

// Or use meaningful IDs based on content
const downloadId = `video-${videoId}`;
```
{% endcode %}

### Handle Browser Support Gracefully

{% hint style="info" %}
Always check for the `unsupported` event and provide fallback behavior for browsers that don't support the Background Fetch API.
{% endhint %}

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <button id="download-btn" data-url="/file.mp4">Download</button>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const btn = document.getElementById('download-btn');
  let supportsBackgroundFetch = true;

  host.addEventListener('background-fetch:unsupported', () => {
    supportsBackgroundFetch = false;
  });

  btn.addEventListener('click', () => {
    if (!supportsBackgroundFetch) {
      // Fallback to regular download
      window.location.href = btn.dataset.url;
    }
  });
</script>
```
{% endcode %}

### Manage Storage Space

{% hint style="warning" %}
Background-fetched files are stored in IndexedDB, which has storage limits. Implement cleanup strategies to manage storage space:
- Delete old downloads when new ones complete
- Provide UI for users to manually delete downloads
- Monitor storage quota usage
{% endhint %}

{% code lineNumbers="true" %}
```javascript
// Example: Delete old files when storage is low
host.addEventListener('background-fetch:completed', async (e) => {
  const files = await host.controller.getStoredFiles();

  // If more than 10 files, delete oldest
  if (files.length > 10) {
    const oldestFile = files[0];
    await host.controller.delete({ params: { url: oldestFile.id } });
    console.log('Deleted old file to free space');
  }
});
```
{% endcode %}

### Provide User Feedback

{% hint style="success" %}
The Background Fetch API shows system-level notifications, but always provide in-app UI feedback as well for a better user experience.
{% endhint %}

### Icons for Better UX

{% code lineNumbers="true" %}
```twig
{# Provide icons for better visual feedback in system notifications #}
<button {{ stimulus_action('@pwa/background-fetch', 'download', {
  id: 'podcast-1',
  url: '/podcasts/episode.mp3',
  title: 'Downloading Podcast',
  icons: [
    { src: '/icons/icon-72.png', sizes: '72x72', type: 'image/png' },
    { src: '/icons/icon-192.png', sizes: '192x192', type: 'image/png' }
  ],
  downloadTotal: 52428800
}) }}>Download Episode</button>
```
{% endcode %}

## Common Use Cases

### 1. Podcast/Music Download Manager

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <div class="podcast-library">
    {% for episode in podcasts %}
      <div class="episode-card">
        <h3>{{ episode.title }}</h3>
        <button class="download-btn" {{ stimulus_action('@pwa/background-fetch', 'download', {
          id: 'podcast-' ~ episode.id,
          url: episode.audioUrl,
          title: 'Downloading: ' ~ episode.title,
          icons: [{ src: episode.artwork, sizes: '192x192' }],
          downloadTotal: episode.fileSize
        }) }}>
          Download for Offline
        </button>
        <div class="progress-{{ episode.id }} hidden"></div>
      </div>
    {% endfor %}
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');

  host.addEventListener('background-fetch:started', (e) => {
    const episodeId = e.detail.id.replace('podcast-', '');
    const progressDiv = document.querySelector(`.progress-${episodeId}`);
    progressDiv.classList.remove('hidden');
  });

  host.addEventListener('background-fetch:in-progress', (e) => {
    const episodeId = e.detail.id.replace('podcast-', '');
    const progressDiv = document.querySelector(`.progress-${episodeId}`);
    const percent = ((e.detail.downloaded / e.detail.downloadTotal) * 100).toFixed(0);
    progressDiv.innerHTML = `<div class="w-full bg-gray-200 rounded"><div class="bg-green-500 h-2 rounded" style="width: ${percent}%"></div></div>`;
  });

  host.addEventListener('background-fetch:completed', (e) => {
    const episodeId = e.detail.id.replace('podcast-', '');
    const progressDiv = document.querySelector(`.progress-${episodeId}`);
    progressDiv.innerHTML = '<span class="text-green-600">✓ Downloaded</span>';
  });
</script>
```
{% endcode %}

### 2. Video Course Offline Access

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <div class="course-section">
    <h2>Download Entire Course</h2>
    <button {{ stimulus_action('@pwa/background-fetch', 'download', {
      id: 'course-complete',
      url: [
        '/courses/video-1.mp4',
        '/courses/video-2.mp4',
        '/courses/video-3.mp4',
        '/courses/resources.pdf'
      ],
      title: 'Downloading Complete Course',
      icons: [{ src: '/course-icon.png', sizes: '192x192' }],
      downloadTotal: 524288000
    }) }}>
      Download All Lessons (500MB)
    </button>
    <div id="course-download-progress"></div>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const progress = document.getElementById('course-download-progress');

  host.addEventListener('background-fetch:started', (e) => {
    progress.innerHTML = `
      <div class="mt-4 p-4 bg-blue-50 rounded">
        <p class="font-semibold">Downloading ${e.detail.urls.length} files...</p>
        <div class="progress-bar-container mt-2"></div>
      </div>
    `;
  });

  host.addEventListener('background-fetch:in-progress', (e) => {
    const percent = ((e.detail.downloaded / e.detail.downloadTotal) * 100).toFixed(1);
    const mb = (e.detail.downloaded / 1024 / 1024).toFixed(0);
    const totalMb = (e.detail.downloadTotal / 1024 / 1024).toFixed(0);

    progress.querySelector('.progress-bar-container').innerHTML = `
      <div class="w-full bg-gray-300 rounded-full h-6">
        <div class="bg-blue-600 h-6 rounded-full flex items-center justify-center text-white text-sm"
             style="width: ${percent}%">
          ${percent}%
        </div>
      </div>
      <p class="text-sm mt-2">${mb}MB / ${totalMb}MB</p>
    `;
  });

  host.addEventListener('background-fetch:completed', () => {
    progress.innerHTML = `
      <div class="mt-4 p-4 bg-green-50 rounded">
        <p class="font-semibold text-green-700">✓ Course downloaded!</p>
        <p class="text-sm">All lessons are now available offline</p>
      </div>
    `;
  });
</script>
```
{% endcode %}

### 3. Document Library Sync

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <div class="library-header">
    <h2>My Documents</h2>
    <button id="sync-all-btn">Sync All for Offline</button>
  </div>
  <div id="documents-list"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');
  const syncBtn = document.getElementById('sync-all-btn');

  syncBtn.addEventListener('click', async () => {
    // Fetch list of documents from API
    const response = await fetch('/api/documents');
    const documents = await response.json();

    // Calculate total size
    const totalSize = documents.reduce((sum, doc) => sum + doc.size, 0);

    // Start background fetch for all documents
    const urls = documents.map(doc => doc.url);

    syncBtn.dispatchEvent(new CustomEvent('background-fetch:download', {
      bubbles: true,
      detail: {
        params: {
          id: 'library-sync-' + Date.now(),
          url: urls,
          title: `Syncing ${documents.length} documents`,
          icons: [{ src: '/icons/sync.png', sizes: '192x192' }],
          downloadTotal: totalSize
        }
      }
    }));
  });

  host.addEventListener('background-fetch:completed', () => {
    alert('All documents are now available offline!');
  });
</script>
```
{% endcode %}

### 4. Media Gallery Offline Mode

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch') }}>
  <div class="gallery">
    {% for photo in photos %}
      <div class="photo-item" data-photo-id="{{ photo.id }}">
        <img src="{{ photo.thumbnail }}" alt="{{ photo.title }}">
        <button class="save-offline" {{ stimulus_action('@pwa/background-fetch', 'download', {
          id: 'photo-' ~ photo.id,
          url: photo.highResUrl,
          title: 'Saving: ' ~ photo.title,
          downloadTotal: photo.size
        }) }}>
          Save Offline
        </button>
      </div>
    {% endfor %}
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__background-fetch"]');

  host.addEventListener('background-fetch:completed', async (e) => {
    const photoId = e.detail.id.replace('photo-', '');
    const photoItem = document.querySelector(`[data-photo-id="${photoId}"]`);
    const button = photoItem.querySelector('.save-offline');

    button.textContent = '✓ Saved';
    button.disabled = true;
    button.classList.add('saved');
  });

  host.addEventListener('background-fetch:failed', (e) => {
    const photoId = e.detail.id.replace('photo-', '');
    const photoItem = document.querySelector(`[data-photo-id="${photoId}"]`);
    const button = photoItem.querySelector('.save-offline');

    button.textContent = 'Retry';
    button.classList.add('error');
  });
</script>
```
{% endcode %}

## API Reference

### Values

#### `dbName`

**Type:** `String`
**Default:** `'bgfetch-completed'`

The name of the IndexedDB database used to store completed downloads.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/background-fetch', { dbName: 'my-custom-db' }) }}>
</div>
```
{% endcode %}

### Actions

#### `download`

Starts a background fetch for one or more files.

**Parameters:**
- `id` (string, required): Unique identifier for this download
- `url` (string or array, required): URL(s) to download
- `title` (string, optional): Title shown in system UI
- `icons` (array, optional): Icons for system notifications
- `downloadTotal` (number, optional but recommended): Total download size in bytes

{% code lineNumbers="true" %}
```twig
<button {{ stimulus_action('@pwa/background-fetch', 'download', {
  id: 'unique-id',
  url: '/file.mp4',
  title: 'Downloading Video',
  icons: [{ src: '/icon.png', sizes: '192x192' }],
  downloadTotal: 52428800
}) }}>Download</button>
```
{% endcode %}

#### `cancel`

Cancels an in-progress background fetch.

**Parameters:**
- `id` (string, required): ID of the download to cancel

{% code lineNumbers="true" %}
```twig
<button {{ stimulus_action('@pwa/background-fetch', 'cancel', { id: 'download-id' }) }}>
  Cancel
</button>
```
{% endcode %}

#### `list`

Lists all current and completed background fetches. This action is automatically called on controller connection.

{% code lineNumbers="true" %}
```javascript
// Manually trigger list refresh
host.controller.list();
```
{% endcode %}

### Methods

#### `has(params)`

Checks if a file with the given URL is stored in IndexedDB.

**Parameters:**
- `params.url` (string): URL to check

**Returns:** `Promise<boolean>`

{% code lineNumbers="true" %}
```javascript
const isStored = await host.controller.has({ params: { url: '/file.mp4' } });
if (isStored) {
  console.log('File is available offline');
}
```
{% endcode %}

#### `get(event)`

Retrieves a downloaded file from IndexedDB and opens or downloads it.

**Parameters:**
- `event.params.url` (string): URL of the stored file
- `event.params.name` (string, optional): Filename for download (if omitted, file opens in new tab)

{% code lineNumbers="true" %}
```javascript
// Open in new tab
host.controller.get({
  preventDefault: () => {},
  params: { url: '/file.mp4' }
});

// Download with custom filename
host.controller.get({
  preventDefault: () => {},
  params: { url: '/file.mp4', name: 'my-video.mp4' }
});
```
{% endcode %}

#### `delete(params)`

Deletes a stored file from IndexedDB.

**Parameters:**
- `params.url` (string): URL of the file to delete

{% code lineNumbers="true" %}
```javascript
await host.controller.delete({ params: { url: '/file.mp4' } });
console.log('File deleted from storage');
```
{% endcode %}

#### `getStoredFiles()`

Returns all files stored in IndexedDB.

**Returns:** `Promise<Array>`

{% code lineNumbers="true" %}
```javascript
const files = await host.controller.getStoredFiles();
console.log(`You have ${files.length} downloaded files`);
files.forEach(file => {
  console.log(`${file.name}: ${file.size} bytes`);
});
```
{% endcode %}

### Targets

None

### Events

#### `unsupported`

Fired when the Background Fetch API is not supported by the browser.

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:unsupported', () => {
  console.log('Background Fetch not supported');
  // Implement fallback behavior
});
```
{% endcode %}

#### `started`

Fired when a background fetch starts successfully.

**Event detail:**
- `id`: Download ID
- `urls`: Array of URLs being downloaded
- `title`: Download title
- `downloadTotal`: Total size in bytes

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:started', (e) => {
  console.log(`Download started: ${e.detail.title}`);
  console.log(`Files: ${e.detail.urls.length}`);
});
```
{% endcode %}

#### `in-progress`

Fired periodically during download to report progress.

**Event detail:**
- `id`: Download ID
- `downloaded`: Bytes downloaded so far
- `downloadTotal`: Total size in bytes

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:in-progress', (e) => {
  const percent = (e.detail.downloaded / e.detail.downloadTotal) * 100;
  console.log(`Progress: ${percent.toFixed(1)}%`);
});
```
{% endcode %}

#### `completed`

Fired when a download completes successfully and is stored.

**Event detail:**
- `id`: File URL
- `name`: Filename
- `size`: File size in bytes
- `contentType`: MIME type

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:completed', (e) => {
  console.log(`Download completed: ${e.detail.name} (${e.detail.size} bytes)`);
});
```
{% endcode %}

#### `failed`

Fired when a download fails.

**Event detail:**
- `id`: Download ID
- `downloaded`: Bytes downloaded before failure
- `downloadTotal`: Total size
- `urls`: Array of URLs

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:failed', (e) => {
  console.error(`Download failed: ${e.detail.id}`);
});
```
{% endcode %}

#### `aborted`

Fired when a download is successfully cancelled.

**Event detail:**
- `id`: Download ID

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:aborted', (e) => {
  console.log(`Download cancelled: ${e.detail.id}`);
});
```
{% endcode %}

#### `exists`

Fired when attempting to start a download with an ID that's already in use.

**Event detail:**
- `id`: Download ID
- `urls`: URLs of existing download
- `title`: Title of existing download
- `downloadTotal`: Total size

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:exists', (e) => {
  console.log(`Download ${e.detail.id} is already in progress`);
});
```
{% endcode %}

#### `not-found`

Fired when attempting to cancel a download that doesn't exist.

**Event detail:**
- `id`: Download ID

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:not-found', (e) => {
  console.log(`Download ${e.detail.id} not found`);
});
```
{% endcode %}

#### `cancel-refused`

Fired when a download cannot be cancelled (already finished or failed).

**Event detail:**
- `id`: Download ID
- `reason`: Reason why cancellation was refused

{% code lineNumbers="true" %}
```javascript
host.addEventListener('background-fetch:cancel-refused', (e) => {
  console.log(`Cannot cancel: ${e.detail.reason}`);
});
```
{% endcode %}

## Related Components

- [Service Worker](service-worker.md) - Required for Background Fetch to function
- [BackgroundSync Queue](backgroundsync-queue.md) - Queue requests for later execution
- [Sync Broadcast](sync-broadcast.md) - Broadcast messages between tabs and service worker

## Resources

- [MDN: Background Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Fetch_API)
- [Web.dev: Background Fetch](https://web.dev/background-fetch/)
- [Can I Use: Background Fetch](https://caniuse.com/background-fetch)

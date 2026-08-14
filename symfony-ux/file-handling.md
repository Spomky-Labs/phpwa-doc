# File Handling

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The File Handling component provides an interface to the File Handling API (Launch Queue API), enabling your Progressive Web App to receive and process files that users open through the operating system. This makes your PWA behave like a native application that can be associated with specific file types.

This component is particularly useful for:

* Image editors and viewers
* Document processors (PDF, Office files)
* Media players (audio, video)
* Code editors and IDEs
* Design tools (SVG, CAD files)
* Data import/export applications

{% hint style="success" %}
**Looking for something else?**

* **Manifest configuration** (which file types to handle): See [File Handlers documentation](../the-manifest/file-handlers.md)
* **Client-side implementation** (how to process files): You're on the right page!
* **Complete quick start example**: See [Quick Start in File Handlers](../the-manifest/file-handlers.md#quick-start)
{% endhint %}

## How It Works

The File Handling feature consists of two parts:

1. **Manifest Configuration**: Declare which file types your PWA can handle ([see File Handlers documentation](../the-manifest/file-handlers.md))
2. **Client-Side Handling**: Receive and process files using the Stimulus controller (this page)

### Complete Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. User Action                                                  │
│    User double-clicks a .jpg file or right-clicks → Open with  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 2. Operating System                                             │
│    - Checks file type (.jpg)                                    │
│    - Looks for registered PWA handlers                          │
│    - Finds your PWA in manifest file_handlers                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 3. PWA Launch                                                   │
│    - Opens PWA to the URL specified in action parameter         │
│    - Adds file(s) to Launch Queue                               │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 4. Stimulus Controller (@pwa/file-handling)                     │
│    - Detects Launch Queue has files                             │
│    - Creates blob URL for each file                             │
│    - Dispatches pwa--file-handling:selected event               │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│ 5. Your Application Code                                        │
│    - Listens to pwa--file-handling:selected                     │
│    - Receives file data (blob URL)                              │
│    - Processes file (display, edit, convert, etc.)              │
└─────────────────────────────────────────────────────────────────┘
```

**Key Points:**
* The manifest `action` URL must match the page where you add the Stimulus controller
* The `accept` configuration in the manifest determines which files trigger your PWA
* Each file generates a separate `pwa--file-handling:selected` event
* The event provides a blob URL that you can use immediately

This page focuses on **steps 4-5** (client-side implementation). For **step 3** (manifest configuration), see the [File Handlers manifest documentation](../the-manifest/file-handlers.md).

## Browser Support

The File Handling API is currently supported in Chromium-based browsers (Chrome, Edge) on desktop platforms. Support on mobile and other browsers is limited or not available.

## Prerequisites

### Step 1: Configure Manifest File Handlers

Before your PWA can receive files, you must declare the file types it can handle in the manifest. This tells the operating system to associate your PWA with specific file extensions.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        file_handlers:
            - action: "app_file_handler"  # Route name or URL
              accept:
                  "image/png": [".png"]
                  "image/jpeg": [".jpg", ".jpeg"]
                  "image/webp": [".webp"]
```
{% endcode %}

{% hint style="info" %}
The `action` parameter should point to the route/page where you'll add the Stimulus controller (Step 2). See the [File Handlers manifest documentation](../the-manifest/file-handlers.md) for detailed configuration options including wildcards, multiple handlers, and advanced URL configuration.
{% endhint %}

### Step 2: Add the Stimulus Controller

Add the `@pwa/file-handling` controller to the template/page specified in the manifest's `action` parameter:

{% code title="templates/file_handler.html.twig" lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <!-- Your file handling UI -->
</div>
```
{% endcode %}

The controller automatically listens for files passed through the Launch Queue API and dispatches the `pwa--file-handling:selected` event for each file.

## Usage

### Basic Image Viewer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="viewer">
        <p id="status">Waiting for files to open...</p>
        <div id="image-container"></div>
    </div>
</div>

<script>
    const status = document.getElementById('status');
    const container = document.getElementById('image-container');

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        status.textContent = 'Loading image...';

        // Create image element
        const img = document.createElement('img');
        img.src = data;
        img.style.maxWidth = '100%';
        img.onload = () => {
            status.textContent = 'Image loaded successfully!';
        };

        // Clear previous content and add new image
        container.innerHTML = '';
        container.appendChild(img);
    });
</script>
```
{% endcode %}

### Photo Editor with Multiple Files

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="photo-editor">
        <h2>Photo Editor</h2>

        <div id="gallery" class="image-gallery"></div>

        <div class="editor-tools">
            <button id="apply-filter">Apply Filter</button>
            <button id="rotate">Rotate</button>
            <button id="save">Save</button>
        </div>
    </div>
</div>

<script>
    const gallery = document.getElementById('gallery');
    const loadedImages = [];

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        // Create thumbnail
        const wrapper = document.createElement('div');
        wrapper.className = 'thumbnail';

        const img = document.createElement('img');
        img.src = data;
        img.onclick = () => selectImage(loadedImages.length);

        wrapper.appendChild(img);
        gallery.appendChild(wrapper);

        // Store image data
        loadedImages.push({
            url: data,
            element: img
        });

        console.log(`Loaded ${loadedImages.length} image(s)`);
    });

    function selectImage(index) {
        // Highlight selected image
        document.querySelectorAll('.thumbnail').forEach((el, i) => {
            el.classList.toggle('selected', i === index);
        });

        // Load image in editor
        console.log('Editing image:', loadedImages[index].url);
    }

    document.getElementById('apply-filter').addEventListener('click', () => {
        // Apply filter to selected image
        const selected = document.querySelector('.thumbnail.selected img');
        if (selected) {
            selected.style.filter = 'grayscale(100%)';
        }
    });

    document.getElementById('rotate').addEventListener('click', () => {
        // Rotate selected image
        const selected = document.querySelector('.thumbnail.selected img');
        if (selected) {
            const currentRotation = selected.dataset.rotation || 0;
            const newRotation = (parseInt(currentRotation) + 90) % 360;
            selected.style.transform = `rotate(${newRotation}deg)`;
            selected.dataset.rotation = newRotation;
        }
    });
</script>

<style>
    .image-gallery {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
        gap: 10px;
        margin: 20px 0;
    }

    .thumbnail {
        border: 2px solid transparent;
        padding: 5px;
        cursor: pointer;
        transition: border-color 0.2s;
    }

    .thumbnail.selected {
        border-color: #007bff;
    }

    .thumbnail img {
        width: 100%;
        height: 150px;
        object-fit: cover;
        border-radius: 4px;
    }

    .editor-tools {
        display: flex;
        gap: 10px;
        margin-top: 20px;
    }

    .editor-tools button {
        padding: 10px 20px;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Document Viewer (PDF, Text)

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="document-viewer">
        <div class="toolbar">
            <h2 id="document-title">No document loaded</h2>
            <button id="download">Download</button>
        </div>

        <div class="document-content">
            <iframe id="document-frame" style="width:100%; height:600px; border:none;"></iframe>
        </div>
    </div>
</div>

<script>
    const frame = document.getElementById('document-frame');
    const title = document.getElementById('document-title');
    const downloadBtn = document.getElementById('download');
    let currentFileUrl = null;

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;
        currentFileUrl = data;

        // Display document in iframe
        frame.src = data;

        // Extract filename from URL if possible
        try {
            const url = new URL(data);
            const filename = url.pathname.split('/').pop() || 'document';
            title.textContent = decodeURIComponent(filename);
        } catch {
            title.textContent = 'Document loaded';
        }

        downloadBtn.style.display = 'block';
    });

    downloadBtn.addEventListener('click', () => {
        if (currentFileUrl) {
            const a = document.createElement('a');
            a.href = currentFileUrl;
            a.download = title.textContent || 'document';
            a.click();
        }
    });
</script>
```
{% endcode %}

### Media Player with Playlist

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="media-player">
        <h2>Media Player</h2>

        <video id="player" controls style="width:100%; max-width:800px;"></video>

        <div class="playlist">
            <h3>Playlist</h3>
            <ul id="playlist-items"></ul>
        </div>
    </div>
</div>

<script>
    const player = document.getElementById('player');
    const playlistEl = document.getElementById('playlist-items');
    const playlist = [];

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        // Add to playlist
        const index = playlist.length;
        playlist.push({ url: data, title: `Video ${index + 1}` });

        // Create playlist item
        const li = document.createElement('li');
        li.textContent = `Video ${index + 1}`;
        li.onclick = () => playVideo(index);
        playlistEl.appendChild(li);

        // Auto-play first video
        if (playlist.length === 1) {
            playVideo(0);
        }

        console.log(`Added video to playlist (${playlist.length} total)`);
    });

    function playVideo(index) {
        if (index >= 0 && index < playlist.length) {
            player.src = playlist[index].url;
            player.play();

            // Highlight current item
            document.querySelectorAll('#playlist-items li').forEach((li, i) => {
                li.style.fontWeight = i === index ? 'bold' : 'normal';
                li.style.color = i === index ? '#007bff' : 'inherit';
            });
        }
    }

    // Auto-play next video when current ends
    player.addEventListener('ended', () => {
        const currentIndex = playlist.findIndex(item => item.url === player.src);
        const nextIndex = currentIndex + 1;
        if (nextIndex < playlist.length) {
            playVideo(nextIndex);
        }
    });
</script>

<style>
    .playlist {
        margin-top: 20px;
    }

    #playlist-items {
        list-style: none;
        padding: 0;
    }

    #playlist-items li {
        padding: 10px;
        cursor: pointer;
        border-bottom: 1px solid #eee;
        transition: background 0.2s;
    }

    #playlist-items li:hover {
        background: #f5f5f5;
    }
</style>
```
{% endcode %}

### Code Editor with Syntax Highlighting

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="code-editor">
        <div class="editor-header">
            <span id="filename">No file opened</span>
            <button id="save-file">Save</button>
        </div>

        <textarea id="code-content" spellcheck="false"></textarea>

        <div class="editor-status">
            <span id="line-count">0 lines</span>
            <span id="char-count">0 characters</span>
        </div>
    </div>
</div>

<script>
    const codeContent = document.getElementById('code-content');
    const filename = document.getElementById('filename');
    const lineCount = document.getElementById('line-count');
    const charCount = document.getElementById('char-count');
    let currentFileHandle = null;

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        try {
            // Fetch file content
            const response = await fetch(data);
            const text = await response.text();

            // Display in editor
            codeContent.value = text;
            filename.textContent = 'Opened file';

            updateStats();
        } catch (error) {
            console.error('Error loading file:', error);
            alert('Failed to load file');
        }
    });

    // Update statistics
    function updateStats() {
        const lines = codeContent.value.split('\n').length;
        const chars = codeContent.value.length;

        lineCount.textContent = `${lines} lines`;
        charCount.textContent = `${chars} characters`;
    }

    codeContent.addEventListener('input', updateStats);

    document.getElementById('save-file').addEventListener('click', async () => {
        // In a real app, you would use the File System Access API to save
        const blob = new Blob([codeContent.value], { type: 'text/plain' });
        const url = URL.createObjectURL(blob);

        const a = document.createElement('a');
        a.href = url;
        a.download = filename.textContent || 'file.txt';
        a.click();

        URL.revokeObjectURL(url);
    });
</script>

<style>
    .code-editor {
        border: 1px solid #ccc;
        border-radius: 4px;
        overflow: hidden;
    }

    .editor-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px;
        background: #f5f5f5;
        border-bottom: 1px solid #ccc;
    }

    #code-content {
        width: 100%;
        height: 500px;
        padding: 10px;
        font-family: 'Courier New', monospace;
        font-size: 14px;
        border: none;
        resize: vertical;
    }

    .editor-status {
        display: flex;
        gap: 20px;
        padding: 10px;
        background: #f5f5f5;
        border-top: 1px solid #ccc;
        font-size: 12px;
        color: #666;
    }
</style>
```
{% endcode %}

### CSV Data Importer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="csv-importer">
        <h2>CSV Data Importer</h2>

        <div id="file-info">
            <p>Waiting for CSV file...</p>
        </div>

        <div id="data-preview" style="display:none;">
            <h3>Data Preview</h3>
            <table id="preview-table">
                <thead id="table-head"></thead>
                <tbody id="table-body"></tbody>
            </table>

            <button id="import-data">Import Data</button>
        </div>
    </div>
</div>

<script>
    const fileInfo = document.getElementById('file-info');
    const dataPreview = document.getElementById('data-preview');
    const tableHead = document.getElementById('table-head');
    const tableBody = document.getElementById('table-body');
    let parsedData = null;

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        try {
            // Fetch and parse CSV
            const response = await fetch(data);
            const text = await response.text();

            parsedData = parseCSV(text);

            displayPreview(parsedData);
        } catch (error) {
            console.error('Error loading CSV:', error);
            fileInfo.innerHTML = '<p style="color:red;">Failed to load CSV file</p>';
        }
    });

    function parseCSV(text) {
        const lines = text.trim().split('\n');
        const headers = lines[0].split(',').map(h => h.trim());
        const rows = lines.slice(1).map(line =>
            line.split(',').map(cell => cell.trim())
        );

        return { headers, rows };
    }

    function displayPreview(data) {
        fileInfo.innerHTML = `<p>Found ${data.rows.length} rows</p>`;
        dataPreview.style.display = 'block';

        // Headers
        tableHead.innerHTML = '<tr>' +
            data.headers.map(h => `<th>${h}</th>`).join('') +
            '</tr>';

        // Preview first 10 rows
        const previewRows = data.rows.slice(0, 10);
        tableBody.innerHTML = previewRows.map(row =>
            '<tr>' + row.map(cell => `<td>${cell}</td>`).join('') + '</tr>'
        ).join('');

        if (data.rows.length > 10) {
            fileInfo.innerHTML += `<p><em>Showing first 10 of ${data.rows.length} rows</em></p>`;
        }
    }

    document.getElementById('import-data').addEventListener('click', () => {
        if (parsedData) {
            console.log('Importing data:', parsedData);
            alert(`Importing ${parsedData.rows.length} rows...`);
            // Send to server or process locally
        }
    });
</script>

<style>
    #preview-table {
        width: 100%;
        border-collapse: collapse;
        margin: 20px 0;
    }

    #preview-table th,
    #preview-table td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }

    #preview-table th {
        background: #f5f5f5;
        font-weight: bold;
    }

    #import-data {
        padding: 10px 20px;
        background: #28a745;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
</style>
```
{% endcode %}

## Parameters

None

## Actions

None - This controller automatically listens for files passed through the Launch Queue API.

## Targets

None

## Events

### `pwa--file-handling:selected`

Dispatched for each file that the app receives through the Launch Queue API. This event is triggered when:
* User right-clicks a file and selects "Open with..." your PWA
* User double-clicks a file associated with your PWA
* File is dragged onto your PWA icon

**Payload**: `{data}`
* `data` (string): A blob URL or data URL representing the file content

Example:
```javascript
document.addEventListener('pwa--file-handling:selected', async (event) => {
    const { data } = event.detail;

    console.log('Received file:', data);

    // Process image
    if (data.startsWith('data:image/') || data.includes('.jpg') || data.includes('.png')) {
        const img = document.createElement('img');
        img.src = data;
        document.body.appendChild(img);
    }

    // Process text/code file
    else if (data.startsWith('data:text/') || data.includes('.txt') || data.includes('.js')) {
        const response = await fetch(data);
        const text = await response.text();
        console.log('File content:', text);
    }
});
```

## Best Practices

1. **Validate file types**: Always validate that received files match expected types
2. **Handle errors gracefully**: File loading can fail for various reasons
3. **Show loading states**: Provide feedback while processing files
4. **Support multiple files**: The event fires once per file, handle multiple files properly
5. **Cleanup resources**: Revoke blob URLs when done to free memory
6. **Declare file handlers**: Properly configure manifest file handlers
7. **Test thoroughly**: Test with various file types and sizes

## File Type Detection

```javascript
document.addEventListener('pwa--file-handling:selected', async (event) => {
    const { data } = event.detail;

    // Fetch to get actual file type
    const response = await fetch(data);
    const blob = await response.blob();
    const mimeType = blob.type;

    console.log('MIME type:', mimeType);

    // Handle based on type
    if (mimeType.startsWith('image/')) {
        handleImage(data);
    } else if (mimeType.startsWith('video/')) {
        handleVideo(data);
    } else if (mimeType === 'application/pdf') {
        handlePDF(data);
    } else if (mimeType.startsWith('text/')) {
        handleText(data);
    } else {
        console.warn('Unsupported file type:', mimeType);
    }
});
```

## Resource Cleanup

Always clean up blob URLs to prevent memory leaks:

```javascript
const loadedFiles = [];

document.addEventListener('pwa--file-handling:selected', (event) => {
    const { data } = event.detail;

    // Store for later cleanup
    loadedFiles.push(data);

    // Use the file...
});

// Clean up when done
function cleanup() {
    loadedFiles.forEach(url => {
        if (url.startsWith('blob:')) {
            URL.revokeObjectURL(url);
        }
    });
    loadedFiles.length = 0;
}

// Call cleanup when appropriate (e.g., when closing document)
window.addEventListener('beforeunload', cleanup);
```

## Complete Example: Multi-Format File Viewer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
    <div class="file-viewer">
        <div class="viewer-header">
            <h2>Universal File Viewer</h2>
            <div id="file-tabs"></div>
        </div>

        <div class="viewer-content">
            <div id="viewer-display"></div>
        </div>

        <div class="viewer-footer">
            <span id="file-count">No files loaded</span>
        </div>
    </div>
</div>

<script>
    const tabsContainer = document.getElementById('file-tabs');
    const display = document.getElementById('viewer-display');
    const fileCount = document.getElementById('file-count');
    const openFiles = [];

    document.addEventListener('pwa--file-handling:selected', async (event) => {
        const { data } = event.detail;

        // Fetch file to get metadata
        const response = await fetch(data);
        const blob = await response.blob();
        const mimeType = blob.type;

        const fileIndex = openFiles.length;
        const file = {
            url: data,
            type: mimeType,
            blob: blob,
            name: `File ${fileIndex + 1}`
        };

        openFiles.push(file);
        createTab(file, fileIndex);
        displayFile(file, fileIndex);
        updateFileCount();
    });

    function createTab(file, index) {
        const tab = document.createElement('button');
        tab.className = 'file-tab';
        tab.textContent = file.name;
        tab.onclick = () => displayFile(file, index);

        const closeBtn = document.createElement('span');
        closeBtn.textContent = '×';
        closeBtn.className = 'close-tab';
        closeBtn.onclick = (e) => {
            e.stopPropagation();
            closeFile(index);
        };

        tab.appendChild(closeBtn);
        tabsContainer.appendChild(tab);
    }

    function displayFile(file, index) {
        // Highlight active tab
        document.querySelectorAll('.file-tab').forEach((tab, i) => {
            tab.classList.toggle('active', i === index);
        });

        // Display based on type
        display.innerHTML = '';

        if (file.type.startsWith('image/')) {
            const img = document.createElement('img');
            img.src = file.url;
            img.style.maxWidth = '100%';
            display.appendChild(img);
        }
        else if (file.type.startsWith('video/')) {
            const video = document.createElement('video');
            video.src = file.url;
            video.controls = true;
            video.style.maxWidth = '100%';
            display.appendChild(video);
        }
        else if (file.type === 'application/pdf') {
            const iframe = document.createElement('iframe');
            iframe.src = file.url;
            iframe.style.width = '100%';
            iframe.style.height = '600px';
            iframe.style.border = 'none';
            display.appendChild(iframe);
        }
        else if (file.type.startsWith('text/')) {
            const pre = document.createElement('pre');
            file.blob.text().then(text => {
                pre.textContent = text;
                pre.style.padding = '20px';
                pre.style.background = '#f5f5f5';
                pre.style.overflow = 'auto';
            });
            display.appendChild(pre);
        }
        else {
            display.innerHTML = `<p>Unsupported file type: ${file.type}</p>`;
        }
    }

    function closeFile(index) {
        // Revoke blob URL
        if (openFiles[index].url.startsWith('blob:')) {
            URL.revokeObjectURL(openFiles[index].url);
        }

        // Remove file and tab
        openFiles.splice(index, 1);
        tabsContainer.children[index].remove();

        updateFileCount();

        // Display previous file or clear
        if (openFiles.length > 0) {
            const newIndex = Math.max(0, index - 1);
            displayFile(openFiles[newIndex], newIndex);
        } else {
            display.innerHTML = '<p>No files loaded</p>';
        }
    }

    function updateFileCount() {
        fileCount.textContent = openFiles.length === 0
            ? 'No files loaded'
            : `${openFiles.length} file(s) loaded`;
    }
</script>

<style>
    .file-viewer {
        border: 1px solid #ddd;
        border-radius: 8px;
        overflow: hidden;
    }

    .viewer-header {
        background: #f5f5f5;
        padding: 15px;
        border-bottom: 1px solid #ddd;
    }

    .viewer-header h2 {
        margin: 0 0 10px 0;
    }

    #file-tabs {
        display: flex;
        gap: 5px;
        overflow-x: auto;
    }

    .file-tab {
        padding: 8px 12px;
        background: white;
        border: 1px solid #ddd;
        border-radius: 4px;
        cursor: pointer;
        display: flex;
        align-items: center;
        gap: 8px;
        white-space: nowrap;
    }

    .file-tab.active {
        background: #007bff;
        color: white;
        border-color: #007bff;
    }

    .close-tab {
        font-size: 18px;
        line-height: 1;
        opacity: 0.7;
    }

    .close-tab:hover {
        opacity: 1;
    }

    .viewer-content {
        padding: 20px;
        min-height: 400px;
    }

    #viewer-display {
        min-height: 300px;
    }

    .viewer-footer {
        padding: 10px 15px;
        background: #f5f5f5;
        border-top: 1px solid #ddd;
        font-size: 12px;
        color: #666;
    }
</style>
```
{% endcode %}

## Browser Compatibility

| Feature | Chrome | Edge | Safari | Firefox |
|---------|--------|------|--------|---------|
| File Handling API | ✅ 102+ | ✅ 102+ | ❌ | ❌ |
| Launch Queue | ✅ 102+ | ✅ 102+ | ❌ | ❌ |

Always provide alternative ways to open files (file input, drag-and-drop) for browsers that don't support the File Handling API.

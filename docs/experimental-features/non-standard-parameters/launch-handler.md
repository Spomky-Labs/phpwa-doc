# Launch handler

!!! warning

    **Experimental Feature**: The Launch Handler API is experimental and currently only supported in Chromium-based browsers (Chrome, Edge). The specification may change.

## Overview

The Launch Handler API allows you to control how your PWA is launched and navigated. It determines whether the app uses an existing window or creates a new one, and how the target launch URL is handled.

**Key Capabilities**:
- Control window reuse behavior
- Handle navigation to launch URLs programmatically
- Receive launch parameters in JavaScript
- Implement custom launch logic

## Configuration

### Basic configuration

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        launch_handler:
            client_mode: "focus-existing"
```

### Multiple client modes (fallback)

Specify multiple modes with fallback behavior:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        launch_handler:
            client_mode:
                - "focus-existing"
                - "auto"
```

The browser will try the first mode, falling back to the next if unsupported.

### Complete example

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Task Manager"
        start_url: "/"
        display: "standalone"
        launch_handler:
            client_mode: "focus-existing"
```

## Client modes

### focus-existing

**Description**: Focuses the most recently used window without navigating. The target URL is passed to JavaScript via `LaunchQueue` for custom handling.

**Behavior**:
- ✅ Reuses existing window
- ✅ Brings window to foreground
- ❌ Does NOT navigate automatically
- ✅ Target URL available via `window.launchQueue`

**Use Cases**:
- Email clients - focus existing window, load new message
- Task managers - focus app, add new task from URL
- Note-taking apps - focus window, create note
- Chat applications - focus window, open conversation
- Single-instance apps that handle URLs programmatically

**Example**:
```yaml
launch_handler:
    client_mode: "focus-existing"
```

**JavaScript Handler**:
```javascript
window.launchQueue.setConsumer((launchParams) => {
    const targetURL = launchParams.targetURL;
    console.log('App launched/focused with URL:', targetURL);

    // Custom navigation logic
    if (targetURL.includes('/task/')) {
        openTask(targetURL);
    } else {
        showDashboard();
    }
});
```

### navigate-existing

**Description**: Navigates the most recently used window to the target URL, and also provides the URL via `LaunchQueue` for additional handling.

**Behavior**:
- ✅ Reuses existing window
- ✅ Automatically navigates to target URL
- ✅ Target URL also available via `window.launchQueue`

**Use Cases**:
- Web browsers - navigate to new URL
- Documentation viewers - load new page
- Content platforms - navigate to shared content
- Apps where automatic navigation is desired

**Example**:
```yaml
launch_handler:
    client_mode: "navigate-existing"
```

**JavaScript Handler**:
```javascript
window.launchQueue.setConsumer((launchParams) => {
    // Window already navigated, but you can do additional setup
    console.log('Navigated to:', launchParams.targetURL);

    // Additional custom logic
    trackNavigation(launchParams.targetURL);
    restoreAppState();
});
```

### navigate-new

**Description**: Creates a new window/tab and navigates to the target URL. Useful for apps where multiple instances make sense.

**Behavior**:
- ✅ Creates new window
- ✅ Automatically navigates to target URL
- ✅ Multiple concurrent instances possible
- ✅ Target URL available via `window.launchQueue`

**Use Cases**:
- Document editors - each document in its own window
- Image editors - open each image separately
- Development tools - multiple project windows
- Apps benefiting from multiple instances

**Example**:
```yaml
launch_handler:
    client_mode: "navigate-new"
```

**JavaScript Handler**:
```javascript
window.launchQueue.setConsumer((launchParams) => {
    console.log('New window created for:', launchParams.targetURL);

    // Initialize new instance
    setupNewWindow();
    loadDocument(launchParams.targetURL);
});
```

### auto (default)

**Description**: Browser chooses the best behavior for the platform.

**Behavior**:
- Mobile: Typically uses `navigate-existing` (single instance)
- Desktop: May use `navigate-new` (multiple instances)
- Platform-optimized behavior

**Use Cases**:
- Default mode if not specified
- When you want platform-optimized behavior
- Apps that should work well everywhere

**Example**:
```yaml
launch_handler:
    client_mode: "auto"
```

## Browser support

| Browser | Version | Support |
|---------|---------|---------|
| Chrome (Desktop) | 92+ | ✅ Full support |
| Chrome (Android) | 92+ | ✅ Full support |
| Edge (Desktop) | 92+ | ✅ Full support |
| Safari | All | ❌ Not supported |
| Firefox | All | ❌ Not supported |

!!! info

    **Fallback**: When not supported, the browser falls back to default launch behavior (typically opens new window).

## LaunchQueue API

The LaunchQueue API allows you to handle launches programmatically in your JavaScript code.

### Basic usage

```javascript title="public/app.js" linenums="1"
// Set up launch handler
if ('launchQueue' in window) {
    window.launchQueue.setConsumer((launchParams) => {
        handleLaunch(launchParams);
    });
}

function handleLaunch(launchParams) {
    const targetURL = launchParams.targetURL;

    console.log('App launched with URL:', targetURL);

    // Parse URL to determine action
    const url = new URL(targetURL);

    if (url.pathname.startsWith('/task/')) {
        const taskId = url.pathname.split('/')[2];
        openTask(taskId);
    } else if (url.searchParams.has('share')) {
        handleSharedContent(url);
    } else {
        showHome();
    }
}
```

### Complete example: Task manager

```javascript title="public/task-manager.js" linenums="1"
class TaskManager {
    constructor() {
        this.setupLaunchHandler();
    }

    setupLaunchHandler() {
        if ('launchQueue' in window) {
            window.launchQueue.setConsumer((launchParams) => {
                this.handleLaunch(launchParams);
            });
        }
    }

    handleLaunch(launchParams) {
        const url = new URL(launchParams.targetURL);

        // Handle different launch scenarios
        if (url.pathname === '/new-task') {
            this.createTask();
        } else if (url.pathname.startsWith('/task/')) {
            const taskId = url.pathname.split('/').pop();
            this.openTask(taskId);
        } else if (url.searchParams.has('notification')) {
            const notificationId = url.searchParams.get('notification');
            this.handleNotification(notificationId);
        } else {
            this.showDashboard();
        }
    }

    createTask() {
        console.log('Creating new task');
        document.getElementById('task-form').style.display = 'block';
    }

    openTask(taskId) {
        console.log('Opening task:', taskId);
        fetch(`/api/tasks/${taskId}`)
            .then(r => r.json())
            .then(task => this.displayTask(task));
    }

    handleNotification(notificationId) {
        console.log('Handling notification:', notificationId);
        // Custom notification handling
    }

    showDashboard() {
        console.log('Showing dashboard');
        window.location.hash = '#dashboard';
    }

    displayTask(task) {
        // Display task details
        document.getElementById('task-title').textContent = task.title;
        document.getElementById('task-description').textContent = task.description;
    }
}

// Initialize
new TaskManager();
```

### Handling file launches

When combined with File Handling API:

```javascript title="public/document-editor.js" linenums="1"
if ('launchQueue' in window) {
    window.launchQueue.setConsumer(async (launchParams) => {
        // Handle file launches
        if (launchParams.files && launchParams.files.length > 0) {
            for (const fileHandle of launchParams.files) {
                const file = await fileHandle.getFile();
                await openFile(file);
            }
        } else {
            // Regular URL launch
            const url = new URL(launchParams.targetURL);
            navigateToURL(url);
        }
    });
}

async function openFile(file) {
    const contents = await file.text();
    document.getElementById('editor').value = contents;
    console.log(`Opened file: ${file.name}`);
}
```

## Use cases

### 1. Email client

Focus existing window and open specific email:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Mail Client"
        launch_handler:
            client_mode: "focus-existing"
```

```javascript title="public/mail-client.js" linenums="1"
window.launchQueue.setConsumer((launchParams) => {
    const url = new URL(launchParams.targetURL);

    // Extract email ID from URL
    if (url.pathname.startsWith('/email/')) {
        const emailId = url.pathname.split('/')[2];
        openEmail(emailId);
    }
});

function openEmail(emailId) {
    fetch(`/api/emails/${emailId}`)
        .then(r => r.json())
        .then(email => {
            document.getElementById('email-viewer').innerHTML = email.body;
            document.querySelector('.email-subject').textContent = email.subject;
        });
}
```

### 2. Multi-document editor

Allow multiple document windows:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Document Editor"
        launch_handler:
            client_mode: "navigate-new"
```

### 3. Single-instance media player

Reuse window and queue new media:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Media Player"
        launch_handler:
            client_mode: "focus-existing"
```

```javascript title="public/media-player.js" linenums="1"
const playlist = [];

window.launchQueue.setConsumer((launchParams) => {
    const url = new URL(launchParams.targetURL);

    // Add media to playlist
    if (url.searchParams.has('media')) {
        const mediaURL = url.searchParams.get('media');
        playlist.push(mediaURL);

        if (!isPlaying()) {
            playNext();
        }
    }
});

function playNext() {
    if (playlist.length > 0) {
        const media = playlist.shift();
        document.getElementById('player').src = media;
    }
}
```

### 4. Note-taking app

Focus window and create note from shared text:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Notes"
        launch_handler:
            client_mode: "focus-existing"
        share_target:
            action: "/share"
            method: "GET"
            params:
                title: "title"
                text: "text"
```

```javascript title="public/notes-app.js" linenums="1"
window.launchQueue.setConsumer((launchParams) => {
    const url = new URL(launchParams.targetURL);

    // Handle shared content
    if (url.pathname === '/share') {
        const title = url.searchParams.get('title') || 'New Note';
        const text = url.searchParams.get('text') || '';

        createNote(title, text);
    }
});

function createNote(title, content) {
    const note = {
        id: Date.now(),
        title,
        content,
        created: new Date()
    };

    // Save note
    saveNote(note);

    // Display in editor
    document.getElementById('note-title').value = title;
    document.getElementById('note-content').value = content;
}
```

### 5. Platform-optimized behavior

Let browser decide best behavior:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Universal App"
        launch_handler:
            client_mode: "auto"
```

## Testing

### 1. Feature detection

Check if LaunchQueue API is available:

```javascript
if ('launchQueue' in window && window.launchQueue) {
    console.log('LaunchQueue API supported');

    window.launchQueue.setConsumer((launchParams) => {
        console.log('Launch params:', {
            targetURL: launchParams.targetURL,
            files: launchParams.files
        });
    });
} else {
    console.log('LaunchQueue API not supported');
    // Fallback behavior
}
```

### 2. Test launch scenarios

```javascript
// Create test function
function testLaunch() {
    const testURLs = [
        '/task/123',
        '/new-task',
        '/task/456?notification=true',
        '/?source=notification'
    ];

    console.log('Testing launch URLs:');
    testURLs.forEach(url => {
        console.log(`- ${url}`);
        // In actual testing, navigate to these URLs
    });
}
```

### 3. DevTools testing

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Launch Handler" section
4. Verify client_mode is configured
5. Test by opening app from different sources
```

### 4. Real-world testing

```bash
# Desktop
1. Install PWA
2. Close app window
3. Click link that launches app (e.g., from email)
4. Observe: new window or existing window focused?
5. Check console for LaunchQueue events

# Mobile (Android Chrome)
1. Install PWA
2. Press home button (don't close app)
3. Tap app icon again
4. Observe launch behavior
5. Try launching from notifications or other apps
```

## Best practices

### 1. Always set launch consumer

```javascript
// ✓ Good - set consumer early
if ('launchQueue' in window) {
    window.launchQueue.setConsumer((launchParams) => {
        handleLaunch(launchParams);
    });
}

// ✗ Bad - may miss early launches
setTimeout(() => {
    window.launchQueue.setConsumer(...);
}, 1000);
```

### 2. Handle all launch scenarios

```javascript
// ✓ Good - handles different cases
window.launchQueue.setConsumer((launchParams) => {
    const url = new URL(launchParams.targetURL);

    if (url.pathname.startsWith('/task/')) {
        openTask(url);
    } else if (url.searchParams.has('share')) {
        handleShare(url);
    } else {
        showDefault();
    }
});
```

### 3. Provide fallback for unsupported browsers

```javascript
// ✓ Good - works with and without API
function setupApp() {
    if ('launchQueue' in window) {
        window.launchQueue.setConsumer(handleLaunch);
    } else {
        // Traditional URL-based routing
        routeBasedOnCurrentURL();
    }
}
```

### 4. Log launch events

```javascript
// ✓ Good - helps with debugging
window.launchQueue.setConsumer((launchParams) => {
    console.log('Launch event:', {
        url: launchParams.targetURL,
        timestamp: new Date(),
        files: launchParams.files?.length || 0
    });

    handleLaunch(launchParams);
});
```

### 5. Combine with other APIs

```javascript
// ✓ Good - works with File Handling API
window.launchQueue.setConsumer(async (launchParams) => {
    // Handle file launches
    if (launchParams.files) {
        for (const handle of launchParams.files) {
            await openFile(handle);
        }
        return;
    }

    // Handle URL launches
    navigateToURL(launchParams.targetURL);
});
```

## Common mistakes

### 1. Not setting consumer early

**Problem**:
```javascript
// Consumer set too late, misses initial launch
window.addEventListener('load', () => {
    window.launchQueue.setConsumer(...);
});
```

**Solution**:
```javascript
// Set consumer in main script, before load
if ('launchQueue' in window) {
    window.launchQueue.setConsumer(...);
}
```

### 2. Assuming API availability

**Problem**:
```javascript
// Crashes if API not supported
window.launchQueue.setConsumer(...);
```

**Solution**:
```javascript
// Feature detection
if ('launchQueue' in window) {
    window.launchQueue.setConsumer(...);
}
```

### 3. Not handling targetURL properly

**Problem**:
```javascript
// Assumes specific URL format
const taskId = launchParams.targetURL.split('/')[2];
```

**Solution**:
```javascript
// Parse URL properly
const url = new URL(launchParams.targetURL);
if (url.pathname.startsWith('/task/')) {
    const taskId = url.pathname.split('/').pop();
}
```

### 4. Ignoring files parameter

**Problem**:
```javascript
// Only handles URL
window.launchQueue.setConsumer((launchParams) => {
    navigateTo(launchParams.targetURL);
});
```

**Solution**:
```javascript
// Handle both files and URLs
window.launchQueue.setConsumer((launchParams) => {
    if (launchParams.files) {
        handleFiles(launchParams.files);
    } else {
        navigateTo(launchParams.targetURL);
    }
});
```

## Troubleshooting

### Launch handler not working

**Problem**: LaunchQueue consumer not receiving events

**Checklist**:
- ✓ Check browser support (Chrome/Edge 92+)
- ✓ Verify PWA is installed (not running in browser tab)
- ✓ Ensure `launch_handler` is in manifest
- ✓ Set consumer before first launch completes
- ✓ Check DevTools console for errors
- ✓ Verify manifest is valid JSON

### Multiple Windows opening

**Problem**: New window opens instead of reusing existing

**Solutions**:
- Check `client_mode` is set to `"focus-existing"` or `"navigate-existing"`
- Verify PWA is launched from installed app (not browser tab)
- Ensure manifest is correctly loaded
- Test on supported browser

### targetURL not correct

**Problem**: LaunchQueue receives wrong URL

**Solutions**:
- Check `start_url` in manifest
- Verify URL used to launch app
- Inspect `launchParams.targetURL` value
- Ensure proper URL encoding

## Related documentation

- [File Handlers](../../the-manifest/file-handlers.md) - Handle file opens
- [Share Target](../../the-manifest/share-target.md) - Receive shared content
- [Display Override](display-override.md) - Control app window display
- [Protocol Handlers](../../the-manifest/protocol-handlers.md) - Handle custom protocols

## Resources

- **Launch Handler API**: https://developer.chrome.com/docs/web-platform/launch-handler/
- **LaunchQueue Spec**: https://wicg.github.io/sw-launch/
- **MDN LaunchQueue**: https://developer.mozilla.org/en-US/docs/Web/API/LaunchQueue
- **Chrome Platform Status**: https://chromestatus.com/feature/5722383233056768

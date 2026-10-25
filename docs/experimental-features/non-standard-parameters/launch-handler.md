# Launch Handler

{% hint style="warning" %}
**Experimental Feature**: The Launch Handler API is experimental and currently only supported in Chromium-based browsers (Chrome, Edge). The specification may change.
{% endhint %}

## Overview

The Launch Handler API allows you to control how your PWA is launched and navigated. It determines whether the app uses an existing window or creates a new one, and how the target launch URL is handled.

**Key Capabilities**:
- Control window reuse behavior
- Handle navigation to launch URLs programmatically
- Receive launch parameters in JavaScript
- Implement custom launch logic

## Configuration

### Basic Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        launch_handler:
            client_mode: "focus-existing"
```
{% endcode %}

### Multiple Client Modes (Fallback)

Specify multiple modes with fallback behavior:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        launch_handler:
            client_mode:
                - "focus-existing"
                - "auto"
```
{% endcode %}

The browser will try the first mode, falling back to the next if unsupported.

### Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Task Manager"
        start_url: "/"
        display: "standalone"
        launch_handler:
            client_mode: "focus-existing"
```
{% endcode %}

## Client Modes

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

## Browser Support

| Browser | Version | Support |
|---------|---------|---------|
| Chrome (Desktop) | 92+ | ✅ Full support |
| Chrome (Android) | 92+ | ✅ Full support |
| Edge (Desktop) | 92+ | ✅ Full support |
| Safari | All | ❌ Not supported |
| Firefox | All | ❌ Not supported |

{% hint style="info" %}
**Fallback**: When not supported, the browser falls back to default launch behavior (typically opens new window).
{% endhint %}

## LaunchQueue API

The LaunchQueue API allows you to handle launches programmatically in your JavaScript code.

### Basic Usage

{% code title="public/app.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

### Complete Example: Task Manager

{% code title="public/task-manager.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

### Handling File Launches

When combined with File Handling API:

{% code title="public/document-editor.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

## Use Cases

### 1. Email Client

Focus existing window and open specific email:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Mail Client"
        launch_handler:
            client_mode: "focus-existing"
```
{% endcode %}

{% code title="public/mail-client.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

### 2. Multi-Document Editor

Allow multiple document windows:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Document Editor"
        launch_handler:
            client_mode: "navigate-new"
```
{% endcode %}

### 3. Single-Instance Media Player

Reuse window and queue new media:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Media Player"
        launch_handler:
            client_mode: "focus-existing"
```
{% endcode %}

{% code title="public/media-player.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

### 4. Note-Taking App

Focus window and create note from shared text:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

{% code title="public/notes-app.js" lineNumbers="true" %}
```javascript
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
{% endcode %}

### 5. Platform-Optimized Behavior

Let browser decide best behavior:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Universal App"
        launch_handler:
            client_mode: "auto"
```
{% endcode %}

## Testing

### 1. Feature Detection

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

### 2. Test Launch Scenarios

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

### 3. DevTools Testing

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Launch Handler" section
4. Verify client_mode is configured
5. Test by opening app from different sources
```

### 4. Real-World Testing

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

## Best Practices

### 1. Always Set Launch Consumer

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

### 2. Handle All Launch Scenarios

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

### 3. Provide Fallback for Unsupported Browsers

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

### 4. Log Launch Events

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

### 5. Combine with Other APIs

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

## Common Mistakes

### 1. Not Setting Consumer Early

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

### 2. Assuming API Availability

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

### 3. Not Handling targetURL Properly

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

### 4. Ignoring Files Parameter

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

### Launch Handler Not Working

**Problem**: LaunchQueue consumer not receiving events

**Checklist**:
- ✓ Check browser support (Chrome/Edge 92+)
- ✓ Verify PWA is installed (not running in browser tab)
- ✓ Ensure `launch_handler` is in manifest
- ✓ Set consumer before first launch completes
- ✓ Check DevTools console for errors
- ✓ Verify manifest is valid JSON

### Multiple Windows Opening

**Problem**: New window opens instead of reusing existing

**Solutions**:
- Check `client_mode` is set to `"focus-existing"` or `"navigate-existing"`
- Verify PWA is launched from installed app (not browser tab)
- Ensure manifest is correctly loaded
- Test on supported browser

### targetURL Not Correct

**Problem**: LaunchQueue receives wrong URL

**Solutions**:
- Check `start_url` in manifest
- Verify URL used to launch app
- Inspect `launchParams.targetURL` value
- Ensure proper URL encoding

## Related Documentation

- [File Handlers](../../the-manifest/file-handlers.md) - Handle file opens
- [Share Target](../../the-manifest/share-target.md) - Receive shared content
- [Display Override](display-override.md) - Control app window display
- [Protocol Handlers](../../the-manifest/protocol-handlers.md) - Handle custom protocols

## Resources

- **Launch Handler API**: https://developer.chrome.com/docs/web-platform/launch-handler/
- **LaunchQueue Spec**: https://wicg.github.io/sw-launch/
- **MDN LaunchQueue**: https://developer.mozilla.org/en-US/docs/Web/API/LaunchQueue
- **Chrome Platform Status**: https://chromestatus.com/feature/5722383233056768

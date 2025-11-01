# Connection Status

The Connection Status component monitors and reacts to changes in the user's internet connection status. This component provides a simple yet powerful way to adapt your application's behavior when the user's device goes offline or comes back online, ensuring a seamless user experience regardless of network conditions.

This component is particularly useful for:

* Displaying connection status notifications to users
* Disabling features that require internet connectivity when offline
* Queueing actions for later when connection is restored
* Showing offline-specific UI and content
* Preventing data loss during connectivity issues
* Providing feedback about network-dependent operations
* Implementing offline-first application strategies
* Enhancing Progressive Web App capabilities

## Browser Support

The Connection Status API is based on the `navigator.onLine` property and online/offline events, which are supported by all modern browsers on desktop and mobile platforms.

**Support level:** Universal - works on all major browsers including Chrome, Firefox, Safari, Edge, and their mobile counterparts.

{% hint style="info" %}
The `navigator.onLine` property only indicates whether the browser has a network connection, not whether internet access is actually available. A device might be connected to a local network but have no internet access.
{% endhint %}

## Usage

### Basic Connection Status Display

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status', {
    onlineMessage: 'You are online',
    offlineMessage: 'You are offline'
}) }}>
    <div {{ stimulus_target('@pwa/connection-status', 'attribute') }}
         class="connection-banner online:bg-green-100 online:text-green-800 offline:bg-red-100 offline:text-red-800"
         role="alert">
        <strong>Connection status:</strong>
        <span {{ stimulus_target('@pwa/connection-status', 'message') }}>
            Detecting connection...
        </span>
    </div>
</div>
```
{% endcode %}

### Styled Connection Notification

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status', {
    onlineMessage: '✓ Connected to the internet',
    offlineMessage: '⚠ No internet connection'
}) }}>
    <div {{ stimulus_target('@pwa/connection-status', 'attribute') }}
         {{ stimulus_target('@pwa/connection-status', 'message') }}
         class="fixed top-4 right-4 p-4 rounded-lg shadow-lg transition-all duration-300
                online:bg-green-50 online:text-green-900 online:border-green-200
                offline:bg-yellow-50 offline:text-yellow-900 offline:border-yellow-200"
         style="border-width: 2px;">
    </div>
</div>

<style>
    [data-connection-status="ONLINE"] {
        animation: slideIn 0.3s ease-out;
    }

    [data-connection-status="OFFLINE"] {
        animation: slideIn 0.3s ease-out, pulse 2s infinite;
    }

    @keyframes slideIn {
        from {
            transform: translateX(100%);
            opacity: 0;
        }
        to {
            transform: translateX(0);
            opacity: 1;
        }
    }

    @keyframes pulse {
        0%, 100% {
            opacity: 1;
        }
        50% {
            opacity: 0.7;
        }
    }
</style>
```
{% endcode %}

### Conditional Feature Display

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status') }}>
    <!-- Main content area with connection-aware features -->
    <div {{ stimulus_target('@pwa/connection-status', 'attribute') }}>

        <!-- Online-only features -->
        <div class="online:block offline:hidden">
            <h2>Live Features</h2>
            <button class="btn-primary">Sync Now</button>
            <button class="btn-primary">Upload Files</button>
            <button class="btn-primary">Share Content</button>
        </div>

        <!-- Offline message -->
        <div class="online:hidden offline:block">
            <div class="offline-notice">
                <h2>Offline Mode</h2>
                <p>Some features are unavailable while offline.</p>
                <p>Your changes will be saved and synced when you reconnect.</p>
            </div>
        </div>

        <!-- Always available features -->
        <div class="mt-6">
            <h2>Available Offline</h2>
            <button class="btn-secondary">View Saved Content</button>
            <button class="btn-secondary">Edit Drafts</button>
        </div>
    </div>

    <!-- Status indicator -->
    <div class="status-bar" {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
        <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
    </div>
</div>

<style>
    .online\:block { display: none; }
    [data-connection-status="ONLINE"] .online\:block { display: block; }

    .online\:hidden { display: block; }
    [data-connection-status="ONLINE"] .online\:hidden { display: none; }

    .offline\:block { display: none; }
    [data-connection-status="OFFLINE"] .offline\:block { display: block; }

    .offline\:hidden { display: block; }
    [data-connection-status="OFFLINE"] .offline\:hidden { display: none; }

    .offline-notice {
        padding: 20px;
        background: #fef3c7;
        border: 2px solid #f59e0b;
        border-radius: 8px;
        text-align: center;
    }

    .status-bar {
        position: fixed;
        bottom: 0;
        left: 0;
        right: 0;
        padding: 10px;
        text-align: center;
        font-weight: 500;
        transition: all 0.3s;
    }

    [data-connection-status="ONLINE"] .status-bar {
        background: #d1fae5;
        color: #065f46;
    }

    [data-connection-status="OFFLINE"] .status-bar {
        background: #fef3c7;
        color: #92400e;
    }
</style>
```
{% endcode %}

### Form Behavior Based on Connection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status') }}>
    <form id="contact-form" {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
        <h2>Contact Us</h2>

        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" id="name" name="name" required>
        </div>

        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" name="email" required>
        </div>

        <div class="form-group">
            <label for="message">Message</label>
            <textarea id="message" name="message" required></textarea>
        </div>

        <!-- Online: normal submit button -->
        <button type="submit" class="online:block offline:hidden">
            Send Message
        </button>

        <!-- Offline: save draft button -->
        <button type="button" class="online:hidden offline:block" onclick="saveDraft()">
            Save Draft (Offline)
        </button>

        <!-- Status message -->
        <div class="form-status" {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
            <span class="online:inline offline:hidden">
                ✓ Ready to send
            </span>
            <span class="online:hidden offline:inline">
                ⚠ Offline - your message will be saved as draft
            </span>
        </div>
    </form>
</div>

<script>
    function saveDraft() {
        const formData = {
            name: document.getElementById('name').value,
            email: document.getElementById('email').value,
            message: document.getElementById('message').value,
            savedAt: new Date().toISOString()
        };

        localStorage.setItem('contactFormDraft', JSON.stringify(formData));
        alert('Draft saved! It will be sent when you\'re back online.');
    }

    // Auto-submit drafts when connection is restored
    document.addEventListener('pwa--connection-status:status-changed', (event) => {
        if (event.detail.status === 'ONLINE') {
            const draft = localStorage.getItem('contactFormDraft');
            if (draft) {
                if (confirm('You have a saved draft. Would you like to send it now?')) {
                    const data = JSON.parse(draft);
                    // Submit the form
                    // ... your submit logic here
                    localStorage.removeItem('contactFormDraft');
                }
            }
        }
    });
</script>
```
{% endcode %}

### Action Queue for Offline Operations

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status') }}>
    <div class="task-manager">
        <h2>Task Manager</h2>

        <div class="connection-status" {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
            <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
        </div>

        <div id="pending-queue" class="online:hidden offline:block">
            <p>Pending actions: <span id="queue-count">0</span></p>
        </div>

        <button onclick="performAction('Task completed')">Complete Task</button>
        <button onclick="performAction('New item added')">Add Item</button>
        <button onclick="performAction('Settings updated')">Update Settings</button>

        <div id="action-log"></div>
    </div>
</div>

<script>
    const actionQueue = [];
    let isOnline = navigator.onLine;

    document.addEventListener('pwa--connection-status:status-changed', (event) => {
        isOnline = event.detail.status === 'ONLINE';

        if (isOnline && actionQueue.length > 0) {
            processQueue();
        }

        updateQueueDisplay();
    });

    function performAction(action) {
        const actionItem = {
            action: action,
            timestamp: new Date().toISOString(),
            id: Date.now()
        };

        if (isOnline) {
            // Process immediately
            sendToServer(actionItem);
        } else {
            // Queue for later
            actionQueue.push(actionItem);
            updateQueueDisplay();
            logAction(`⏳ Queued: ${action}`, 'warning');
        }
    }

    function sendToServer(actionItem) {
        // Simulate server request
        console.log('Sending to server:', actionItem);
        logAction(`✓ Sent: ${actionItem.action}`, 'success');

        // In a real app, you would do:
        // fetch('/api/actions', {
        //     method: 'POST',
        //     body: JSON.stringify(actionItem)
        // });
    }

    function processQueue() {
        const count = actionQueue.length;
        logAction(`🔄 Processing ${count} queued action(s)...`, 'info');

        while (actionQueue.length > 0) {
            const action = actionQueue.shift();
            sendToServer(action);
        }

        updateQueueDisplay();
    }

    function updateQueueDisplay() {
        document.getElementById('queue-count').textContent = actionQueue.length;
    }

    function logAction(message, type) {
        const log = document.getElementById('action-log');
        const entry = document.createElement('div');
        entry.className = `log-entry log-${type}`;
        entry.textContent = `${new Date().toLocaleTimeString()}: ${message}`;
        log.insertBefore(entry, log.firstChild);

        // Keep only last 10 entries
        while (log.children.length > 10) {
            log.removeChild(log.lastChild);
        }
    }
</script>

<style>
    .connection-status {
        padding: 10px;
        margin-bottom: 15px;
        border-radius: 6px;
        font-weight: 500;
    }

    [data-connection-status="ONLINE"] .connection-status {
        background: #d1fae5;
        color: #065f46;
    }

    [data-connection-status="OFFLINE"] .connection-status {
        background: #fef3c7;
        color: #92400e;
    }

    #pending-queue {
        padding: 10px;
        background: #fef3c7;
        border-radius: 6px;
        margin-bottom: 15px;
    }

    .log-entry {
        padding: 8px;
        margin: 5px 0;
        border-radius: 4px;
        font-size: 14px;
    }

    .log-success {
        background: #d1fae5;
        color: #065f46;
    }

    .log-warning {
        background: #fef3c7;
        color: #92400e;
    }

    .log-info {
        background: #dbeafe;
        color: #1e40af;
    }
</style>
```
{% endcode %}

### Real-Time Sync Indicator

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status') }}>
    <div class="editor-container">
        <div class="editor-header" {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
            <h2>Document Editor</h2>

            <div class="sync-status">
                <span class="online:inline offline:hidden">
                    <svg class="icon" viewBox="0 0 20 20" fill="currentColor">
                        <path d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"/>
                    </svg>
                    Synced
                </span>

                <span class="online:hidden offline:inline">
                    <svg class="icon" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z"/>
                    </svg>
                    Not synced (offline)
                </span>
            </div>
        </div>

        <textarea id="editor" placeholder="Start typing..."></textarea>

        <div class="editor-footer">
            <span id="last-saved">Never saved</span>
            <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
        </div>
    </div>
</div>

<script>
    const editor = document.getElementById('editor');
    const lastSavedEl = document.getElementById('last-saved');
    let saveTimeout;
    let isOnline = navigator.onLine;

    // Auto-save on typing
    editor.addEventListener('input', () => {
        clearTimeout(saveTimeout);
        saveTimeout = setTimeout(() => {
            saveContent();
        }, 1000);
    });

    // Listen to connection changes
    document.addEventListener('pwa--connection-status:status-changed', (event) => {
        isOnline = event.detail.status === 'ONLINE';

        if (isOnline) {
            syncToServer();
        }
    });

    function saveContent() {
        const content = editor.value;

        if (isOnline) {
            // Save to server
            saveToServer(content);
        } else {
            // Save locally
            localStorage.setItem('editorContent', content);
            lastSavedEl.textContent = `Saved locally at ${new Date().toLocaleTimeString()}`;
        }
    }

    function saveToServer(content) {
        // Simulate server save
        console.log('Saving to server:', content);
        lastSavedEl.textContent = `Synced at ${new Date().toLocaleTimeString()}`;

        // In real app:
        // fetch('/api/save', {
        //     method: 'POST',
        //     body: JSON.stringify({ content })
        // });
    }

    function syncToServer() {
        const localContent = localStorage.getItem('editorContent');
        if (localContent) {
            saveToServer(localContent);
            localStorage.removeItem('editorContent');
        }
    }

    // Load saved content on page load
    window.addEventListener('load', () => {
        const saved = localStorage.getItem('editorContent');
        if (saved) {
            editor.value = saved;
        }
    });
</script>

<style>
    .editor-container {
        max-width: 800px;
        margin: 0 auto;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
        overflow: hidden;
    }

    .editor-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 15px 20px;
        border-bottom: 1px solid #e5e7eb;
    }

    [data-connection-status="ONLINE"] .editor-header {
        background: #f0fdf4;
    }

    [data-connection-status="OFFLINE"] .editor-header {
        background: #fef3c7;
    }

    .sync-status {
        display: flex;
        align-items: center;
        gap: 8px;
        font-weight: 500;
    }

    [data-connection-status="ONLINE"] .sync-status {
        color: #16a34a;
    }

    [data-connection-status="OFFLINE"] .sync-status {
        color: #ca8a04;
    }

    .icon {
        width: 20px;
        height: 20px;
    }

    #editor {
        width: 100%;
        min-height: 300px;
        padding: 20px;
        border: none;
        resize: vertical;
        font-family: monospace;
    }

    .editor-footer {
        display: flex;
        justify-content: space-between;
        padding: 10px 20px;
        background: #f9fafb;
        border-top: 1px solid #e5e7eb;
        font-size: 14px;
        color: #6b7280;
    }
</style>
```
{% endcode %}

### Multiple Status Indicators

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status', {
    onlineMessage: 'Online',
    offlineMessage: 'Offline'
}) }}>
    <nav {{ stimulus_target('@pwa/connection-status', 'attribute') }}
         class="navbar online:bg-white offline:bg-gray-100">
        <div class="navbar-brand">My App</div>

        <!-- Multiple message targets -->
        <div class="status-indicator">
            <span class="status-dot online:bg-green-500 offline:bg-red-500"></span>
            <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
        </div>
    </nav>

    <main {{ stimulus_target('@pwa/connection-status', 'attribute') }}>
        <aside class="sidebar">
            <div class="sidebar-status">
                Connection: <strong {{ stimulus_target('@pwa/connection-status', 'message') }}></strong>
            </div>
        </aside>

        <div class="content">
            <h1>Dashboard</h1>
            <!-- Your content here -->
        </div>
    </main>

    <footer {{ stimulus_target('@pwa/connection-status', 'attribute') }}
            class="footer online:bg-green-50 offline:bg-red-50">
        <p>Status: <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span></p>
    </footer>
</div>

<style>
    .status-dot {
        display: inline-block;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        margin-right: 8px;
    }

    .sidebar-status {
        padding: 10px;
        background: #f3f4f6;
        border-radius: 6px;
        margin-bottom: 20px;
    }

    [data-connection-status="ONLINE"] .sidebar-status {
        border-left: 4px solid #10b981;
    }

    [data-connection-status="OFFLINE"] .sidebar-status {
        border-left: 4px solid #ef4444;
    }
</style>
```
{% endcode %}

## Parameters

### `onlineMessage`

**Type:** `string`
**Default:** `"You are online"`

The message displayed when the user is online. This message is injected into all elements marked with the `message` target.

```twig
{{ stimulus_controller('@pwa/connection-status', {
    onlineMessage: 'Connected to the internet ✓'
}) }}
```

### `offlineMessage`

**Type:** `string`
**Default:** `"You are offline"`

The message displayed when the user is offline. This message is injected into all elements marked with the `message` target.

```twig
{{ stimulus_controller('@pwa/connection-status', {
    offlineMessage: 'No internet connection ⚠'
}) }}
```

## Actions

None - This component automatically monitors connection status without requiring explicit actions.

## Targets

### `message`

HTML elements that will display the connection status message. The content of these elements will be replaced with either `onlineMessage` or `offlineMessage` depending on the current connection status.

**Multiple targets allowed:** Yes - you can have multiple `message` targets throughout your application.

```twig
<span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
```

### `attribute`

HTML elements that will receive a `data-connection-status` attribute set to either `"ONLINE"` or `"OFFLINE"`. This is particularly useful for conditional styling with CSS.

**Multiple targets allowed:** Yes - you can mark multiple elements with this target.

```twig
<div {{ stimulus_target('@pwa/connection-status', 'attribute') }}
     class="online:bg-green-100 offline:bg-red-100">
    <!-- Content -->
</div>
```

The `data-connection-status` attribute can be used with CSS attribute selectors:

```css
[data-connection-status="ONLINE"] {
    background-color: #d1fae5;
}

[data-connection-status="OFFLINE"] {
    background-color: #fee2e2;
}
```

Or with Tailwind CSS variants:

```html
<div class="online:bg-green-100 offline:bg-red-100">
```

## Events

### `pwa--connection-status:status-changed`

Dispatched whenever the connection status changes (online ↔ offline).

**Payload:**
* `status` (string): Either `"ONLINE"` or `"OFFLINE"`
* `message` (string): The current message (either `onlineMessage` or `offlineMessage`)

Example:
```javascript
document.addEventListener('pwa--connection-status:status-changed', (event) => {
    const { status, message } = event.detail;

    console.log('Connection status:', status);
    console.log('Status message:', message);

    if (status === 'ONLINE') {
        // Connection restored - sync data
        syncPendingChanges();
    } else {
        // Connection lost - enable offline mode
        enableOfflineMode();
    }
});
```

## Best Practices

1. **Provide clear feedback**: Always inform users about their connection status
2. **Graceful degradation**: Disable online-only features when offline
3. **Queue operations**: Store user actions locally and sync when connection is restored
4. **Save drafts**: Auto-save content locally to prevent data loss
5. **Visual indicators**: Use clear, consistent visual cues for connection status
6. **Don't block**: Allow users to continue working offline when possible
7. **Optimize for mobile**: Connection status is particularly important on mobile devices
8. **Test thoroughly**: Test your application's behavior in various connectivity scenarios
9. **Handle edge cases**: Consider slow connections, intermittent connectivity
10. **Inform, don't alarm**: Frame offline messages positively when possible

## Connection-Aware CSS Patterns

### Using Tailwind CSS

If you're using Tailwind CSS, you can create custom variants for connection status:

```javascript
// tailwind.config.js
module.exports = {
    theme: {
        extend: {}
    },
    plugins: [
        function({ addVariant }) {
            addVariant('online', '[data-connection-status="ONLINE"] &');
            addVariant('offline', '[data-connection-status="OFFLINE"] &');
        }
    ]
}
```

Then use these variants in your HTML:

```html
<div class="online:bg-green-100 online:text-green-800
            offline:bg-red-100 offline:text-red-800">
    Connection status indicator
</div>
```

### Using Standard CSS

```css
/* Base styles */
.connection-indicator {
    padding: 10px;
    border-radius: 6px;
    font-weight: 500;
    transition: all 0.3s;
}

/* Online state */
[data-connection-status="ONLINE"] .connection-indicator {
    background-color: #d1fae5;
    color: #065f46;
    border-left: 4px solid #10b981;
}

/* Offline state */
[data-connection-status="OFFLINE"] .connection-indicator {
    background-color: #fee2e2;
    color: #991b1b;
    border-left: 4px solid #ef4444;
}

/* Show/hide based on status */
[data-connection-status="ONLINE"] .online-only {
    display: block;
}

[data-connection-status="OFFLINE"] .online-only {
    display: none;
}

[data-connection-status="ONLINE"] .offline-only {
    display: none;
}

[data-connection-status="OFFLINE"] .offline-only {
    display: block;
}
```

## Complete Example: Offline-First Todo App

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/connection-status', {
    onlineMessage: 'Synced',
    offlineMessage: 'Working offline'
}) }}>
    <div class="todo-app">
        <!-- Header with sync status -->
        <header {{ stimulus_target('@pwa/connection-status', 'attribute') }}
                class="app-header online:bg-green-50 offline:bg-amber-50">
            <h1>Todo App</h1>
            <div class="sync-status">
                <span class="status-icon online:text-green-600 offline:text-amber-600">●</span>
                <span {{ stimulus_target('@pwa/connection-status', 'message') }}></span>
            </div>
        </header>

        <!-- Offline notice -->
        <div {{ stimulus_target('@pwa/connection-status', 'attribute') }}
             class="online:hidden offline:block offline-notice">
            <p>📱 You're working offline. Your changes will sync when you're back online.</p>
        </div>

        <!-- Todo form -->
        <form id="todo-form" onsubmit="addTodo(event)">
            <input type="text" id="todo-input" placeholder="Add a new todo..." required>
            <button type="submit">Add</button>
        </form>

        <!-- Pending sync indicator -->
        <div id="pending-sync" class="online:hidden offline:block pending-sync" style="display: none;">
            <p>⏳ <span id="pending-count">0</span> item(s) waiting to sync</p>
        </div>

        <!-- Todo list -->
        <ul id="todo-list"></ul>
    </div>
</div>

<script>
    let todos = JSON.parse(localStorage.getItem('todos') || '[]');
    let pendingSync = JSON.parse(localStorage.getItem('pendingSync') || '[]');
    let isOnline = navigator.onLine;

    // Listen to connection changes
    document.addEventListener('pwa--connection-status:status-changed', (event) => {
        isOnline = event.detail.status === 'ONLINE';

        if (isOnline && pendingSync.length > 0) {
            syncPendingTodos();
        }

        updatePendingCount();
    });

    function addTodo(event) {
        event.preventDefault();

        const input = document.getElementById('todo-input');
        const todo = {
            id: Date.now(),
            text: input.value,
            completed: false,
            createdAt: new Date().toISOString(),
            synced: isOnline
        };

        todos.push(todo);
        saveTodos();

        if (!isOnline) {
            pendingSync.push({ action: 'create', todo });
            savePendingSync();
        } else {
            syncToServer({ action: 'create', todo });
        }

        input.value = '';
        renderTodos();
    }

    function toggleTodo(id) {
        const todo = todos.find(t => t.id === id);
        if (todo) {
            todo.completed = !todo.completed;
            saveTodos();

            if (!isOnline) {
                pendingSync.push({ action: 'update', todo });
                savePendingSync();
            } else {
                syncToServer({ action: 'update', todo });
            }

            renderTodos();
        }
    }

    function deleteTodo(id) {
        const todo = todos.find(t => t.id === id);
        todos = todos.filter(t => t.id !== id);
        saveTodos();

        if (!isOnline) {
            pendingSync.push({ action: 'delete', todo });
            savePendingSync();
        } else {
            syncToServer({ action: 'delete', todo });
        }

        renderTodos();
    }

    function saveTodos() {
        localStorage.setItem('todos', JSON.stringify(todos));
    }

    function savePendingSync() {
        localStorage.setItem('pendingSync', JSON.stringify(pendingSync));
        updatePendingCount();
    }

    function updatePendingCount() {
        const count = pendingSync.length;
        document.getElementById('pending-count').textContent = count;
        document.getElementById('pending-sync').style.display = count > 0 ? 'block' : 'none';
    }

    function syncToServer(change) {
        // Simulate server sync
        console.log('Syncing to server:', change);

        // In a real app:
        // fetch('/api/todos', {
        //     method: 'POST',
        //     headers: { 'Content-Type': 'application/json' },
        //     body: JSON.stringify(change)
        // });
    }

    function syncPendingTodos() {
        console.log(`Syncing ${pendingSync.length} pending changes...`);

        pendingSync.forEach(change => {
            syncToServer(change);
        });

        pendingSync = [];
        savePendingSync();

        // Mark all todos as synced
        todos.forEach(todo => todo.synced = true);
        saveTodos();
        renderTodos();
    }

    function renderTodos() {
        const list = document.getElementById('todo-list');
        list.innerHTML = todos.map(todo => `
            <li class="todo-item ${todo.completed ? 'completed' : ''}">
                <input type="checkbox"
                       ${todo.completed ? 'checked' : ''}
                       onchange="toggleTodo(${todo.id})">
                <span class="todo-text">${todo.text}</span>
                ${!todo.synced ? '<span class="sync-badge">Not synced</span>' : ''}
                <button onclick="deleteTodo(${todo.id})" class="delete-btn">×</button>
            </li>
        `).join('');
    }

    // Initial render
    renderTodos();
    updatePendingCount();
</script>

<style>
    .todo-app {
        max-width: 600px;
        margin: 0 auto;
        padding: 20px;
    }

    .app-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 20px;
        border-radius: 8px;
        margin-bottom: 20px;
    }

    .sync-status {
        display: flex;
        align-items: center;
        gap: 8px;
        font-weight: 500;
    }

    .status-icon {
        font-size: 20px;
        animation: pulse 2s infinite;
    }

    .offline-notice {
        padding: 15px;
        background: #fef3c7;
        border-left: 4px solid #f59e0b;
        border-radius: 6px;
        margin-bottom: 20px;
    }

    .pending-sync {
        padding: 10px;
        background: #fef3c7;
        border-radius: 6px;
        margin-bottom: 15px;
        text-align: center;
        font-weight: 500;
    }

    #todo-form {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
    }

    #todo-input {
        flex: 1;
        padding: 10px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 16px;
    }

    #todo-form button {
        padding: 10px 20px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: 500;
    }

    #todo-list {
        list-style: none;
        padding: 0;
    }

    .todo-item {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 12px;
        background: white;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        margin-bottom: 8px;
    }

    .todo-item.completed .todo-text {
        text-decoration: line-through;
        color: #9ca3af;
    }

    .todo-text {
        flex: 1;
    }

    .sync-badge {
        padding: 4px 8px;
        background: #fef3c7;
        color: #92400e;
        border-radius: 4px;
        font-size: 12px;
        font-weight: 500;
    }

    .delete-btn {
        width: 24px;
        height: 24px;
        border: none;
        background: #ef4444;
        color: white;
        border-radius: 4px;
        cursor: pointer;
        font-size: 18px;
        line-height: 1;
    }

    @keyframes pulse {
        0%, 100% {
            opacity: 1;
        }
        50% {
            opacity: 0.5;
        }
    }
</style>
```
{% endcode %}

## Troubleshooting

### Status not updating

**Issue**: Connection status doesn't change when network connectivity changes

**Solutions**:
- Ensure the controller is properly initialized
- Check browser console for JavaScript errors
- Verify that event listeners are attached correctly
- Test with browser DevTools network throttling

### False positives

**Issue**: Shows "online" even when internet is not accessible

**Cause**: `navigator.onLine` only checks browser's network connection, not actual internet access

**Solution**: Implement additional checks:
```javascript
async function checkRealConnectivity() {
    try {
        const response = await fetch('/ping', {
            method: 'HEAD',
            cache: 'no-cache'
        });
        return response.ok;
    } catch {
        return false;
    }
}
```

### Message not displaying

**Issue**: Connection status message doesn't appear

**Solutions**:
- Verify the `message` target is properly set
- Check that parameters `onlineMessage` and `offlineMessage` are configured
- Inspect the element to ensure it's receiving content

### Styling not applying

**Issue**: CSS classes based on `data-connection-status` not working

**Solutions**:
- Ensure the `attribute` target is set on the correct element
- Check CSS selector specificity
- Verify Tailwind variants are configured if using Tailwind CSS
- Use browser inspector to confirm the attribute is being set

## Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome | ✓ Full support |
| Firefox | ✓ Full support |
| Safari | ✓ Full support |
| Edge | ✓ Full support |
| Opera | ✓ Full support |
| Mobile browsers | ✓ Full support |

{% hint style="success" %}
The Connection Status component works reliably across all modern browsers and is based on well-established web platform APIs.
{% endhint %}

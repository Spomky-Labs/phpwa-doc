---
description: Display a badge on the app icon
---

# Badge

The Badge component provides an interface to the Badging API, enabling your Progressive Web App to display badges on the application icon. These badges are perfect for notifying users of new content, unread messages, pending tasks, or any other count-based information that requires attention.

This component is particularly useful for:

* Displaying unread message or email counts
* Showing notification counters
* Indicating pending tasks or updates
* Alerting users to new content availability
* Displaying shopping cart item counts

## Browser Support

The Badging API is supported in Chromium-based browsers (Chrome, Edge, Opera) on desktop and Android. The badge appears on the PWA's icon when it's installed on the device.

{% hint style="info" %}
Badges only appear when the PWA is installed. Users must install your app to their home screen or desktop to see badges.
{% endhint %}

## Usage

### Basic Badge Counter

{% code lineNumbers="true" %}
```twig
<body {{ stimulus_controller('@pwa/badge') }}>
    <div class="notifications">
        <h2>Notifications</h2>
        <p>You have <span id="notification-count">5</span> new notifications</p>

        <button {{ stimulus_action('@pwa/badge', 'update', 'click', {counter: 5}) }}>
            Set Badge to 5
        </button>

        <button {{ stimulus_action('@pwa/badge', 'clear', 'click') }}>
            Clear Badge
        </button>
    </div>
</body>

<script>
    document.addEventListener('pwa--badge:updated', (event) => {
        console.log('Badge updated:', event.detail.counter);
    });
</script>
```
{% endcode %}

### Dynamic Badge Update on Page Load

{% code lineNumbers="true" %}
```twig
<body
    {{ stimulus_controller('@pwa/badge') }}
    {{ stimulus_action('@pwa/badge', 'update', 'load@window', {counter: 42}) }}
>
    <h1>Welcome Back!</h1>
    <p>You have 42 unread messages</p>
</body>
```
{% endcode %}

### Real-time Badge Updates

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/badge') }}>
    <div class="inbox">
        <h2>Inbox</h2>
        <div id="message-list">
            <!-- Messages here -->
        </div>

        <p>Unread messages: <span id="unread-count">0</span></p>
    </div>
</div>

<script>
    let unreadCount = 0;

    // Update badge when new message arrives
    function onNewMessage(message) {
        if (!message.read) {
            unreadCount++;
            updateBadge();
        }
    }

    function onMessageRead(messageId) {
        unreadCount = Math.max(0, unreadCount - 1);
        updateBadge();
    }

    function updateBadge() {
        document.getElementById('unread-count').textContent = unreadCount;

        // Trigger badge update
        const controller = document.querySelector('[data-controller="@pwa/badge"]');
        controller.dispatchEvent(new CustomEvent('update', {
            detail: {
                params: { counter: unreadCount }
            }
        }));
    }

    // Listen for badge updates
    document.addEventListener('pwa--badge:updated', (event) => {
        console.log('Badge now shows:', event.detail.counter);
    });
</script>
```
{% endcode %}

### Badge with WebSocket Updates

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/badge') }}>
    <div class="notifications-panel">
        <h2>Live Notifications</h2>
        <div id="notification-list"></div>
    </div>
</div>

<script>
    // Connect to WebSocket for real-time updates
    const ws = new WebSocket('wss://example.com/notifications');
    let notificationCount = 0;

    ws.onmessage = function(event) {
        const data = JSON.parse(event.data);

        if (data.type === 'new_notification') {
            notificationCount++;
            addNotificationToList(data.notification);
            updateAppBadge(notificationCount);
        }
    };

    function updateAppBadge(count) {
        const controller = document.querySelector('[data-controller="@pwa/badge"]');
        if (controller) {
            controller.dispatchEvent(new CustomEvent('update', {
                detail: { params: { counter: count } }
            }));
        }
    }

    function addNotificationToList(notification) {
        const list = document.getElementById('notification-list');
        const item = document.createElement('div');
        item.className = 'notification-item';
        item.textContent = notification.message;
        item.onclick = () => markAsRead(notification.id);
        list.prepend(item);
    }

    function markAsRead(id) {
        notificationCount = Math.max(0, notificationCount - 1);
        updateAppBadge(notificationCount);
        // Send read status to server
        fetch(`/notifications/${id}/read`, { method: 'POST' });
    }
</script>
```
{% endcode %}

### E-commerce Shopping Cart Badge

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/badge') }}>
    <header>
        <nav>
            <a href="/cart">
                🛒 Cart
                <span id="cart-count" class="badge-inline">0</span>
            </a>
        </nav>
    </header>
</div>

<script>
    class ShoppingCart {
        constructor() {
            this.items = JSON.parse(localStorage.getItem('cart') || '[]');
            this.updateBadge();
        }

        addItem(product) {
            this.items.push(product);
            this.save();
            this.updateBadge();
        }

        removeItem(productId) {
            this.items = this.items.filter(item => item.id !== productId);
            this.save();
            this.updateBadge();
        }

        save() {
            localStorage.setItem('cart', JSON.stringify(this.items));
        }

        updateBadge() {
            const count = this.items.length;

            // Update inline badge
            document.getElementById('cart-count').textContent = count;

            // Update app icon badge
            const controller = document.querySelector('[data-controller="@pwa/badge"]');
            if (controller) {
                if (count > 0) {
                    controller.dispatchEvent(new CustomEvent('update', {
                        detail: { params: { counter: count } }
                    }));
                } else {
                    controller.dispatchEvent(new CustomEvent('clear'));
                }
            }
        }
    }

    const cart = new ShoppingCart();

    // Example: Add item to cart
    document.querySelectorAll('.add-to-cart-btn').forEach(btn => {
        btn.addEventListener('click', (e) => {
            const product = {
                id: e.target.dataset.productId,
                name: e.target.dataset.productName
            };
            cart.addItem(product);
        });
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `update`

Sets the badge counter to a specified value. The action requires a `counter` parameter.

**Parameters:**
* `counter` (number): The value to display on the badge. Must be a positive integer.

```twig
{{ stimulus_action('@pwa/badge', 'update', 'click', {counter: 10}) }}
```

{% hint style="warning" %}
Some browsers may display "99+" or similar for very large numbers. Keep badge counts reasonable for better user experience.
{% endhint %}

### `clear`

Removes the badge from the application icon.

```twig
{{ stimulus_action('@pwa/badge', 'clear', 'click') }}
```

This is useful when:
* User has read all notifications
* All tasks are completed
* Shopping cart is empty
* Any count-based metric reaches zero

## Targets

None

## Events

### `pwa--badge:updated`

Dispatched whenever the badge value changes (either updated with a new count or cleared).

**Payload**: `{counter}`
* `counter` (number|null): The new badge value, or `null` if the badge was cleared

Example:
```javascript
document.addEventListener('pwa--badge:updated', (event) => {
    const { counter } = event.detail;

    if (counter === null) {
        console.log('Badge cleared');
    } else {
        console.log('Badge updated to:', counter);
    }

    // Update UI to match badge state
    updateUIBadge(counter);
});
```

## Best Practices

1. **Keep counts accurate**: Ensure badge numbers reflect actual unread/pending items
2. **Clear when appropriate**: Remove badges when counts reach zero
3. **Use reasonable numbers**: Very large numbers may be truncated by the browser
4. **Sync with server**: For multi-device users, sync badge counts across devices
5. **Update immediately**: Update badges as soon as content changes, not on page reload
6. **Provide context**: Combine app icon badges with in-app indicators
7. **Don't overuse**: Reserve badges for truly important notifications

## Badge vs Push Notifications

While badges and push notifications both alert users, they serve different purposes:

| Feature | Badge | Push Notification |
|---------|-------|-------------------|
| Visibility | Always visible on app icon | Temporary notification |
| Content | Number only | Rich text and images |
| User interaction | None required | Can be dismissed |
| Best for | Counts and totals | Time-sensitive alerts |
| Timing | Persistent until cleared | Appears once |

Use both together for the best user experience: push notifications for new content, badges for ongoing counts.

## Integration with Service Worker

Badges can be updated from the service worker, enabling background updates:

```javascript
// In your service worker
self.addEventListener('push', (event) => {
    const data = event.data.json();

    // Update badge
    if (data.unreadCount) {
        navigator.setAppBadge(data.unreadCount);
    }

    // Show notification
    self.registration.showNotification(data.title, {
        body: data.body,
        badge: '/icons/badge.png'
    });
});
```

## Complete Example: Messaging App

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/badge') }}>
    <div class="messaging-app">
        <header>
            <h1>Messages</h1>
            <span id="unread-indicator"></span>
        </header>

        <div id="conversation-list">
            <!-- Conversations loaded here -->
        </div>

        <button id="mark-all-read">Mark All as Read</button>
    </div>
</div>

<script>
    class MessagingApp {
        constructor() {
            this.unreadConversations = new Set();
            this.controller = document.querySelector('[data-controller="@pwa/badge"]');
            this.loadConversations();
            this.setupEventListeners();
        }

        loadConversations() {
            // Fetch conversations from API
            fetch('/api/conversations')
                .then(res => res.json())
                .then(conversations => {
                    conversations.forEach(conv => {
                        if (conv.unread) {
                            this.unreadConversations.add(conv.id);
                        }
                    });
                    this.updateBadge();
                });
        }

        markConversationRead(conversationId) {
            this.unreadConversations.delete(conversationId);
            this.updateBadge();

            // Notify server
            fetch(`/api/conversations/${conversationId}/read`, {
                method: 'POST'
            });
        }

        markAllRead() {
            this.unreadConversations.clear();
            this.updateBadge();

            // Notify server
            fetch('/api/conversations/mark-all-read', {
                method: 'POST'
            });
        }

        updateBadge() {
            const count = this.unreadConversations.size;

            // Update in-app indicator
            const indicator = document.getElementById('unread-indicator');
            if (count > 0) {
                indicator.textContent = `${count} unread`;
                indicator.className = 'unread-badge';
            } else {
                indicator.textContent = '';
                indicator.className = '';
            }

            // Update app icon badge
            if (count > 0) {
                this.controller.dispatchEvent(new CustomEvent('update', {
                    detail: { params: { counter: count } }
                }));
            } else {
                this.controller.dispatchEvent(new CustomEvent('clear'));
            }
        }

        setupEventListeners() {
            document.getElementById('mark-all-read').addEventListener('click', () => {
                this.markAllRead();
            });

            // Listen for new messages via WebSocket
            const ws = new WebSocket('wss://example.com/messages');
            ws.onmessage = (event) => {
                const message = JSON.parse(event.data);
                if (message.type === 'new_message') {
                    this.unreadConversations.add(message.conversationId);
                    this.updateBadge();
                }
            };
        }
    }

    const app = new MessagingApp();

    // Listen for badge updates
    document.addEventListener('pwa--badge:updated', (event) => {
        console.log('App badge updated:', event.detail.counter);

        // Optional: Store badge count for analytics
        if (event.detail.counter !== null) {
            localStorage.setItem('lastBadgeCount', event.detail.counter);
        }
    });
</script>
```
{% endcode %}

## Troubleshooting

### Badge not appearing

**Possible causes:**
1. **App not installed**: Badges only work for installed PWAs
2. **Browser support**: Check if the browser supports the Badging API
3. **Permissions**: Some browsers require notification permissions for badges

### Badge not updating

**Common issues:**
1. **Controller not found**: Ensure the controller is properly initialized
2. **Invalid counter value**: Must be a positive integer or 0
3. **Event not dispatched**: Check that events are properly triggered

### Badge count incorrect

**Solutions:**
1. **Sync with server**: Fetch accurate count from server on app launch
2. **Persistent storage**: Store count in localStorage for consistency
3. **Validate updates**: Ensure all increment/decrement operations are correct

## Platform-Specific Behavior

* **Chrome/Edge (Desktop)**: Badge appears as a circle with number
* **Chrome/Edge (Android)**: Badge shown as notification dot or number
* **Safari**: Limited or no support (as of 2024)
* **Firefox**: No support (as of 2024)

Always provide in-app indicators as a fallback for all users.

# BackgroundSync Form

The BackgroundSync Form component transforms regular HTML forms into offline-capable forms that automatically queue submissions when the user is offline. When a form is submitted without internet connectivity, the request is queued by the service worker and automatically retried when the connection is restored. This ensures that user data is never lost due to connectivity issues.

This component is particularly useful for:

* Building reliable data entry forms that work offline
* Creating survey and feedback forms that don't lose responses
* Implementing offline-first contact forms and support tickets
* Building field service applications with unreliable connectivity
* Creating order and checkout forms for e-commerce in low-connectivity areas
* Implementing CRM and sales forms for mobile workers
* Building event registration forms that work anywhere
* Creating time-tracking and expense reporting applications

## Browser Support

The BackgroundSync Form component relies on the Background Sync API and service workers, which are widely supported in modern browsers.

**Support level:** Excellent - Works on all modern browsers that support service workers (Chrome, Firefox, Safari, Edge).

{% hint style="info" %}
This component requires:
- An active service worker with Workbox configured
- Background Sync enabled in your PWA configuration
- Proper queue configuration matching your form endpoints
{% endhint %}

{% hint style="success" %}
When online, forms submit normally with no performance penalty. The background sync mechanism only activates when the network is unavailable.
{% endhint %}

## How It Works

1. **User Submits Form**: User fills out and submits the form
2. **Network Check**: Component attempts to submit via fetch API
3. **If Online**: Form submits normally, response handled, user redirected
4. **If Offline**: Request automatically queued by service worker via Workbox Background Sync
5. **Auto Retry**: Service worker automatically retries when connectivity returns
6. **Success**: User notified when queued submission succeeds

## Configuration

First, configure background sync in your PWA configuration to handle form submissions:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            enabled: true
            background_sync:
                # Queue for contact forms
                - queue_name: 'contact-forms'
                  match_callback: 'startsWith: /contact/submit'
                  method: POST
                  max_retention_time: 10080  # 7 days in minutes
                  force_sync_fallback: true
                  broadcast_channel: 'contact-queue'

                # Queue for order submissions
                - queue_name: 'orders'
                  match_callback: 'startsWith: /order/place'
                  method: POST
                  max_retention_time: 4320   # 3 days in minutes
                  force_sync_fallback: true
                  broadcast_channel: 'order-queue'

                # Queue for data updates
                - queue_name: 'data-updates'
                  match_callback: 'startsWith: /api/update'
                  method: [PUT, PATCH]
                  max_retention_time: 2880   # 2 days in minutes
```
{% endcode %}

Key configuration options:
- `queue_name`: Unique identifier for this queue
- `match_callback`: Pattern to match URLs that should be queued
- `method`: HTTP method(s) to queue (POST, PUT, PATCH, DELETE)
- `max_retention_time`: How long to keep failed requests (in minutes)
- `force_sync_fallback`: Enable fallback for browsers without Background Sync API
- `broadcast_channel`: Optional channel for queue status updates

## Usage

### Basic Form with Background Sync

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/contact/submit"
      method="post">
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

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Send Message
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');

    form.addEventListener('backgroundsync-form:after:send', (e) => {
        if (e.detail.response.ok) {
            alert('Message sent successfully!');
            form.reset();
        }
    });

    form.addEventListener('backgroundsync-form:error', (e) => {
        // Form queued for later - user will be offline
        alert('You are offline. Your message will be sent when you reconnect.');
    });
</script>
```
{% endcode %}

### Form with Custom Redirect

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    redirection: '/thank-you'
}) }}
      action="/contact/submit"
      method="post">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message" required></textarea>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Submit
    </button>
</form>
```
{% endcode %}

After successful submission (or queueing if offline), the user is redirected to `/thank-you`.

### Form with Custom Headers

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    headers: {
        'X-Requested-With': 'XMLHttpRequest',
        'X-CSRF-Token': csrf_token('form_token')
    }
}) }}
      action="/api/submit"
      method="post">
    <input type="text" name="title" required>
    <textarea name="content" required></textarea>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Save
    </button>
</form>
```
{% endcode %}

### Form with Custom Fetch Parameters

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    params: {
        mode: 'cors',
        cache: 'no-cache',
        credentials: 'include',
        redirect: 'follow',
        referrerPolicy: 'strict-origin-when-cross-origin'
    }
}) }}
      action="https://api.example.com/submit"
      method="post">
    <input type="text" name="data" required>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Submit to API
    </button>
</form>
```
{% endcode %}

### JSON Form Submission

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/api/data"
      method="post"
      enctype="application/json">
    <input type="text" name="title" required>
    <input type="text" name="author" required>
    <textarea name="content" required></textarea>
    <input type="number" name="priority" value="1">

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Save as JSON
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');

    form.addEventListener('backgroundsync-form:before:send', (e) => {
        console.log('Sending JSON:', e.detail.params.body);
        // Body will be: {"title":"...","author":"...","content":"...","priority":1}
    });
</script>
```
{% endcode %}

### File Upload Form

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/upload"
      method="post"
      enctype="multipart/form-data">
    <div class="form-group">
        <label for="title">Title</label>
        <input type="text" id="title" name="title" required>
    </div>

    <div class="form-group">
        <label for="file">File</label>
        <input type="file" id="file" name="file" required>
    </div>

    <div class="form-group">
        <label for="description">Description</label>
        <textarea id="description" name="description"></textarea>
    </div>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Upload
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');

    form.addEventListener('backgroundsync-form:after:send', async (e) => {
        if (e.detail.response.ok) {
            const result = await e.detail.response.json();
            alert(`File uploaded! ID: ${result.id}`);
            form.reset();
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        alert('Upload queued. Will be sent when back online.');
    });
</script>
```
{% endcode %}

### Form with Loading State

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/submit"
      method="post"
      id="contact-form">
    <input type="text" name="name" required>
    <input type="email" name="email" required>
    <textarea name="message" required></textarea>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }}
            type="submit"
            id="submit-btn">
        <span class="btn-text">Send</span>
        <span class="btn-loader hidden">Sending...</span>
    </button>
</form>

<script type="module">
    const form = document.getElementById('contact-form');
    const button = document.getElementById('submit-btn');
    const btnText = button.querySelector('.btn-text');
    const btnLoader = button.querySelector('.btn-loader');

    form.addEventListener('backgroundsync-form:before:send', () => {
        button.disabled = true;
        btnText.classList.add('hidden');
        btnLoader.classList.remove('hidden');
    });

    form.addEventListener('backgroundsync-form:after:send', () => {
        button.disabled = false;
        btnText.classList.remove('hidden');
        btnLoader.classList.add('hidden');
    });

    form.addEventListener('backgroundsync-form:error', () => {
        button.disabled = false;
        btnText.classList.remove('hidden');
        btnLoader.classList.add('hidden');
    });
</script>
```
{% endcode %}

### Form with Validation Feedback

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/submit"
      method="post">
    <div id="form-alerts"></div>

    <input type="text" name="username" required minlength="3">
    <input type="email" name="email" required>
    <input type="password" name="password" required minlength="8">

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Register
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');
    const alerts = document.getElementById('form-alerts');

    form.addEventListener('backgroundsync-form:invalid-data', () => {
        alerts.innerHTML = `
            <div class="alert alert-warning">
                Please fill out all required fields correctly.
            </div>
        `;
    });

    form.addEventListener('backgroundsync-form:before:send', () => {
        alerts.innerHTML = '';
    });

    form.addEventListener('backgroundsync-form:after:send', async (e) => {
        if (e.detail.response.ok) {
            alerts.innerHTML = `
                <div class="alert alert-success">
                    Registration successful!
                </div>
            `;
        } else {
            const error = await e.detail.response.json();
            alerts.innerHTML = `
                <div class="alert alert-danger">
                    ${error.message || 'An error occurred'}
                </div>
            `;
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        alerts.innerHTML = `
            <div class="alert alert-info">
                You're offline. Registration will complete when you reconnect.
            </div>
        `;
    });
</script>
```
{% endcode %}

## Best Practices

### Always Configure Matching Routes

{% hint style="warning" %}
Ensure your background sync configuration matches the form action URLs. Mismatched patterns will cause forms to fail offline instead of being queued.
{% endhint %}

{% code lineNumbers="true" %}
```yaml
# Form action: /contact/submit
# Configuration must match:
pwa:
    serviceworker:
        workbox:
            background_sync:
                - queue_name: 'contact'
                  match_callback: 'startsWith: /contact/'  # Matches /contact/*
```
{% endcode %}

### Handle All Submission States

{% hint style="success" %}
Always provide feedback for all three states: loading, success, and offline queueing. Users need to know what's happening with their data.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
// Loading state
form.addEventListener('backgroundsync-form:before:send', () => {
    showLoadingIndicator();
});

// Success state
form.addEventListener('backgroundsync-form:after:send', (e) => {
    if (e.detail.response.ok) {
        showSuccessMessage();
    } else {
        showErrorMessage();
    }
});

// Offline/queued state
form.addEventListener('backgroundsync-form:error', () => {
    showQueuedMessage();
});
```
{% endcode %}

### Validate Before Sending

{% hint style="info" %}
The component automatically calls `form.reportValidity()` before submission. Use HTML5 validation attributes (`required`, `pattern`, `min`, `max`) for client-side validation.
{% endhint %}

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}>
    <input type="email"
           name="email"
           required
           pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
           title="Please enter a valid email address">

    <input type="tel"
           name="phone"
           pattern="[0-9]{10}"
           title="Please enter a 10-digit phone number">

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Submit
    </button>
</form>
```
{% endcode %}

### Choose Appropriate Encoding

{% hint style="info" %}
The component supports three encoding types:
- `multipart/form-data`: For file uploads (default for forms with `<input type="file">`)
- `application/x-www-form-urlencoded`: Standard form encoding (default)
- `application/json`: For API submissions requiring JSON
{% endhint %}

{% code lineNumbers="true" %}
```twig
{# For regular forms (default) #}
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      enctype="application/x-www-form-urlencoded">

{# For file uploads #}
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      enctype="multipart/form-data">
    <input type="file" name="document">
</form>

{# For JSON APIs #}
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      enctype="application/json">
</form>
```
{% endcode %}

### Set Appropriate Retention Times

{% hint style="warning" %}
Configure `max_retention_time` based on the importance and time-sensitivity of your data:
- Critical data (orders, registrations): 7-14 days
- Regular forms (contacts, feedback): 3-5 days
- Non-critical updates: 1-2 days
{% endhint %}

## Common Use Cases

### 1. Offline Contact Form

{% code lineNumbers="true" %}
```twig
<div class="contact-section">
    <h2>Contact Us</h2>

    <div id="status-message"></div>

    <form {{ stimulus_controller('@pwa/backgroundsync-form', {
        redirection: null
    }) }}
          action="/contact/submit"
          method="post"
          id="contact-form">
        <div class="form-row">
            <input type="text" name="name" placeholder="Your Name" required>
        </div>

        <div class="form-row">
            <input type="email" name="email" placeholder="Your Email" required>
        </div>

        <div class="form-row">
            <select name="subject" required>
                <option value="">Select Subject</option>
                <option value="general">General Inquiry</option>
                <option value="support">Technical Support</option>
                <option value="sales">Sales</option>
            </select>
        </div>

        <div class="form-row">
            <textarea name="message" placeholder="Your Message" rows="5" required></textarea>
        </div>

        <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }}
                type="submit"
                class="btn btn-primary">
            Send Message
        </button>
    </form>
</div>

<script type="module">
    const form = document.getElementById('contact-form');
    const status = document.getElementById('status-message');

    form.addEventListener('backgroundsync-form:before:send', () => {
        status.className = 'alert alert-info';
        status.textContent = 'Sending your message...';
    });

    form.addEventListener('backgroundsync-form:after:send', async (e) => {
        if (e.detail.response.ok) {
            status.className = 'alert alert-success';
            status.textContent = 'Thank you! Your message has been sent.';
            form.reset();

            setTimeout(() => {
                status.textContent = '';
            }, 5000);
        } else {
            const error = await e.detail.response.json();
            status.className = 'alert alert-danger';
            status.textContent = error.message || 'Failed to send message. Please try again.';
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        status.className = 'alert alert-warning';
        status.innerHTML = `
            <strong>You're currently offline.</strong><br>
            Don't worry! Your message has been saved and will be sent automatically when you reconnect.
        `;
        form.reset();

        setTimeout(() => {
            status.textContent = '';
        }, 10000);
    });
</script>
```
{% endcode %}

### 2. E-Commerce Order Form

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    headers: {
        'X-CSRF-Token': csrf_token()
    }
}) }}
      action="/order/place"
      method="post"
      id="checkout-form">
    <h3>Shipping Information</h3>
    <input type="text" name="shipping[name]" placeholder="Full Name" required>
    <input type="text" name="shipping[address]" placeholder="Street Address" required>
    <input type="text" name="shipping[city]" placeholder="City" required>
    <input type="text" name="shipping[postal_code]" placeholder="Postal Code" required>
    <input type="tel" name="shipping[phone]" placeholder="Phone Number" required>

    <h3>Payment Information</h3>
    <input type="text" name="payment[card_number]" placeholder="Card Number" required>
    <input type="text" name="payment[expiry]" placeholder="MM/YY" required>
    <input type="text" name="payment[cvv]" placeholder="CVV" required>

    <div class="order-summary">
        <h3>Order Summary</h3>
        <p>Total: ${{ cart.total }}</p>
    </div>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }}
            type="submit"
            id="place-order-btn">
        Place Order
    </button>
</form>

<script type="module">
    const form = document.getElementById('checkout-form');
    const button = document.getElementById('place-order-btn');

    form.addEventListener('backgroundsync-form:before:send', () => {
        button.disabled = true;
        button.textContent = 'Processing Order...';
    });

    form.addEventListener('backgroundsync-form:after:send', async (e) => {
        if (e.detail.response.ok) {
            const order = await e.detail.response.json();
            window.location.href = `/order/confirmation/${order.id}`;
        } else {
            button.disabled = false;
            button.textContent = 'Place Order';
            alert('Order failed. Please check your payment information.');
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        // Order queued for offline processing
        alert('You are offline. Your order has been saved and will be processed when you reconnect.');
        window.location.href = '/order/pending';
    });
</script>
```
{% endcode %}

### 3. Survey/Feedback Form

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/survey/submit"
      method="post"
      enctype="application/json">
    <h2>Customer Satisfaction Survey</h2>

    <div class="question">
        <label>How satisfied are you with our service?</label>
        <div class="rating">
            <input type="radio" name="satisfaction" value="5" required> Very Satisfied
            <input type="radio" name="satisfaction" value="4"> Satisfied
            <input type="radio" name="satisfaction" value="3"> Neutral
            <input type="radio" name="satisfaction" value="2"> Dissatisfied
            <input type="radio" name="satisfaction" value="1"> Very Dissatisfied
        </div>
    </div>

    <div class="question">
        <label>Would you recommend us to others?</label>
        <select name="recommendation" required>
            <option value="">Select...</option>
            <option value="10">Definitely</option>
            <option value="7">Probably</option>
            <option value="5">Maybe</option>
            <option value="3">Probably Not</option>
            <option value="0">Definitely Not</option>
        </select>
    </div>

    <div class="question">
        <label>Additional Comments</label>
        <textarea name="comments" rows="4"></textarea>
    </div>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
        Submit Survey
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');

    form.addEventListener('backgroundsync-form:after:send', (e) => {
        if (e.detail.response.ok) {
            alert('Thank you for your feedback!');
            window.location.href = '/thank-you';
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        alert('Your responses have been saved offline and will be submitted when you reconnect. Thank you!');
        window.location.href = '/thank-you';
    });
</script>
```
{% endcode %}

### 4. Field Service Report

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form') }}
      action="/field/report/submit"
      method="post"
      enctype="multipart/form-data">
    <h2>Service Report</h2>

    <input type="hidden" name="technician_id" value="{{ app.user.id }}">
    <input type="hidden" name="timestamp" value="{{ "now"|date('c') }}">

    <div class="form-group">
        <label>Customer Name</label>
        <input type="text" name="customer_name" required>
    </div>

    <div class="form-group">
        <label>Service Type</label>
        <select name="service_type" required>
            <option value="installation">Installation</option>
            <option value="maintenance">Maintenance</option>
            <option value="repair">Repair</option>
        </select>
    </div>

    <div class="form-group">
        <label>Work Performed</label>
        <textarea name="work_performed" rows="4" required></textarea>
    </div>

    <div class="form-group">
        <label>Photos</label>
        <input type="file" name="photos[]" accept="image/*" multiple>
    </div>

    <div class="form-group">
        <label>Customer Signature</label>
        <canvas id="signature-pad" width="400" height="200"></canvas>
        <input type="hidden" name="signature" id="signature-data">
    </div>

    <button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }}
            type="submit"
            id="submit-report">
        Submit Report
    </button>
</form>

<script type="module">
    const form = document.querySelector('form');

    // Capture signature before submission
    form.addEventListener('backgroundsync-form:before:send', () => {
        const canvas = document.getElementById('signature-pad');
        const signatureData = canvas.toDataURL();
        document.getElementById('signature-data').value = signatureData;
    });

    form.addEventListener('backgroundsync-form:after:send', (e) => {
        if (e.detail.response.ok) {
            alert('Report submitted successfully!');
            window.location.href = '/field/dashboard';
        }
    });

    form.addEventListener('backgroundsync-form:error', () => {
        alert('No internet connection. Report saved offline and will sync automatically.');
        window.location.href = '/field/dashboard';
    });
</script>
```
{% endcode %}

## API Reference

### Values

#### `params`

**Type:** `Object`
**Default:** `{mode: 'cors', cache: 'no-cache', credentials: 'same-origin', redirect: 'follow', referrerPolicy: 'no-referrer'}`

Fetch API parameters to use for the request.

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    params: {
        mode: 'cors',
        cache: 'no-store',
        credentials: 'include'
    }
}) }}>
```
{% endcode %}

#### `headers`

**Type:** `Object`
**Default:** `{}`

Additional HTTP headers to include in the request.

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    headers: {
        'X-API-Key': 'your-api-key',
        'X-Requested-With': 'XMLHttpRequest'
    }
}) }}>
```
{% endcode %}

#### `redirection`

**Type:** `String`
**Default:** `null`

URL to redirect to after form submission (success or queued). If `null`, no redirect occurs.

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    redirection: '/thank-you'
}) }}>
```
{% endcode %}

#### `authenticating`

**Type:** `Boolean`
**Default:** `false`

If `true`, automatically adds JWT authorization header to the request.

{% code lineNumbers="true" %}
```twig
<form {{ stimulus_controller('@pwa/backgroundsync-form', {
    authenticating: true,
    keyIdIndex: 'default'
}) }}>
```
{% endcode %}

#### `keyIdIndex`

**Type:** `String`
**Default:** `'default'`

Key identifier to use for JWT signing when `authenticating` is `true`.

### Actions

#### `send(event)`

Submits the form. Should be attached to the form's submit button.

{% code lineNumbers="true" %}
```twig
<button {{ stimulus_action('@pwa/backgroundsync-form', 'send') }} type="submit">
    Submit
</button>
```
{% endcode %}

### Targets

None

### Events

#### `before:send`

Fired just before the form submission request is sent.

**Event detail:**
- `url` (string): The form action URL
- `params` (object): Complete fetch parameters including headers and body

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:before:send', (e) => {
    console.log('Submitting to:', e.detail.url);
    console.log('With params:', e.detail.params);
});
```
{% endcode %}

#### `after:send`

Fired immediately after receiving a response from the server.

**Event detail:**
- `url` (string): The form action URL
- `params` (object): Fetch parameters used
- `response` (Response): The fetch Response object

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:after:send', async (e) => {
    if (e.detail.response.ok) {
        const data = await e.detail.response.json();
        console.log('Success:', data);
    } else {
        console.error('Server error:', e.detail.response.status);
    }
});
```
{% endcode %}

#### `error`

Fired when the submission fails (typically due to network error/offline status). Request is automatically queued.

**Event detail:**
- `error` (Error): The error object

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:error', (e) => {
    console.log('Form queued for offline sync');
    console.error('Error:', e.detail.error);
});
```
{% endcode %}

#### `invalid-data`

Fired when form validation fails (form.reportValidity() returns false).

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:invalid-data', () => {
    console.log('Please fill out all required fields');
});
```
{% endcode %}

#### `unsupported-enctype`

Fired when the form uses an unsupported encoding type.

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:unsupported-enctype', () => {
    console.error('Form encoding not supported');
});
```
{% endcode %}

#### `auth-missing-key`

Fired when `authenticating` is true but the JWT key is not found.

**Event detail:**
- `keyIdIndex` (string): The missing key identifier

{% code lineNumbers="true" %}
```javascript
form.addEventListener('backgroundsync-form:auth-missing-key', (e) => {
    console.error('Missing auth key:', e.detail.keyIdIndex);
});
```
{% endcode %}

## Related Components

- [BackgroundSync Queue](backgroundsync-queue.md) - Monitor and manage background sync queues
- [Service Worker](service-worker.md) - Service worker configuration and setup
- [Connection Status](connection-status.md) - Display online/offline status to users

## Resources

- [MDN: Background Sync API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [Workbox: Background Sync](https://developer.chrome.com/docs/workbox/modules/workbox-background-sync/)
- [PWA Bundle Service Worker Configuration](../the-service-worker/workbox/backgoundsync.md)

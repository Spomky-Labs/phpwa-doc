# Contact Picker

The Contact Picker component provides access to the Contact Picker API, allowing users to select one or more contacts directly from their device's native address book. This enables seamless integration with the device's contact system while maintaining user privacy through explicit consent for each selection.

This component is particularly useful for:

* Social applications requiring contact selection for invitations
* Communication apps for selecting recipients
* Event management for adding attendees
* Sharing features that integrate with contacts
* Forms requiring contact information input
* CRM applications for importing contacts
* Messaging platforms for contact-based features
* Emergency contact selection in forms
* Group creation and management features

## Browser Support

The Contact Picker API is currently only available on Android devices with Chrome/Edge browsers. It's not supported on desktop or iOS platforms.

**Supported Platforms:**
* Chrome/Edge on Android: Full support
* Desktop browsers: Not supported
* iOS Safari: Not supported
* Firefox: Not supported

{% hint style="warning" %}
Always provide fallback behavior for browsers and platforms that don't support the Contact Picker API. The component will dispatch an `unavailable` event when the API is not available.
{% endhint %}

## Usage

### Basic Contact Selection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <h2>Select a Contact</h2>

    <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click') }}>
        Choose Contact
    </button>

    <div id="contact-display">
        <p>No contact selected yet.</p>
    </div>
</div>

<script>
    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;
        const contact = contacts[0]; // Get first contact

        const displayDiv = document.getElementById('contact-display');
        displayDiv.innerHTML = `
            <h3>Selected Contact:</h3>
            <p><strong>Name:</strong> ${contact.name ? contact.name[0] : 'N/A'}</p>
            <p><strong>Email:</strong> ${contact.email ? contact.email[0] : 'N/A'}</p>
            <p><strong>Phone:</strong> ${contact.tel ? contact.tel[0] : 'N/A'}</p>
        `;
    });

    document.addEventListener('pwa--contact-picker:unavailable', () => {
        document.getElementById('contact-display').innerHTML = `
            <p>Contact Picker is not available on this device.</p>
            <p>Please enter contact information manually.</p>
        `;
    });

    document.addEventListener('pwa--contact-picker:error', (event) => {
        console.log('Contact picker error:', event.detail.exception);
    });
</script>
```
{% endcode %}

### Multiple Contact Selection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <h2>Invite Friends</h2>

    <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
        multiple: true
    }) }}>
        Select Multiple Contacts
    </button>

    <div id="selected-contacts">
        <p>No contacts selected.</p>
    </div>

    <button id="send-invites" style="display: none;">
        Send Invites
    </button>
</div>

<script>
    let selectedContacts = [];

    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;
        selectedContacts = contacts;

        const displayDiv = document.getElementById('selected-contacts');
        const sendBtn = document.getElementById('send-invites');

        if (contacts.length === 0) {
            displayDiv.innerHTML = '<p>No contacts selected.</p>';
            sendBtn.style.display = 'none';
            return;
        }

        const contactsHtml = contacts.map((contact, index) => `
            <div class="contact-card">
                <div class="contact-info">
                    <strong>${contact.name ? contact.name[0] : 'Unknown'}</strong>
                    <div>${contact.email ? contact.email[0] : 'No email'}</div>
                    <div>${contact.tel ? contact.tel[0] : 'No phone'}</div>
                </div>
                <button onclick="removeContact(${index})">Remove</button>
            </div>
        `).join('');

        displayDiv.innerHTML = `
            <h3>${contacts.length} Contact(s) Selected:</h3>
            ${contactsHtml}
        `;

        sendBtn.style.display = 'block';
    });

    function removeContact(index) {
        selectedContacts.splice(index, 1);
        // Trigger re-render
        document.dispatchEvent(new CustomEvent('pwa--contact-picker:selection', {
            detail: { contacts: selectedContacts }
        }));
    }

    document.getElementById('send-invites').addEventListener('click', () => {
        console.log('Sending invites to:', selectedContacts);
        alert(`Invites sent to ${selectedContacts.length} contact(s)!`);
    });
</script>

<style>
    .contact-card {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 15px;
        margin: 10px 0;
        background: #f9fafb;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .contact-info {
        flex: 1;
    }

    .contact-info strong {
        display: block;
        margin-bottom: 5px;
        color: #1f2937;
    }

    .contact-info div {
        font-size: 14px;
        color: #6b7280;
    }

    .contact-card button {
        padding: 6px 12px;
        background: #ef4444;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Form Integration

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <form id="contact-form">
        <h2>Add Emergency Contact</h2>

        <div class="form-group">
            <label for="name">Name</label>
            <div class="input-with-button">
                <input type="text" id="name" name="name" required>
                <button type="button" {{ stimulus_action('@pwa/contact-picker', 'select', 'click') }}>
                    📱 Pick from Contacts
                </button>
            </div>
        </div>

        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" name="email">
        </div>

        <div class="form-group">
            <label for="phone">Phone</label>
            <input type="tel" id="phone" name="phone" required>
        </div>

        <div class="form-group">
            <label for="address">Address</label>
            <textarea id="address" name="address" rows="3"></textarea>
        </div>

        <button type="submit">Save Contact</button>
    </form>
</div>

<script>
    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;
        const contact = contacts[0];

        // Fill form with contact data
        if (contact.name && contact.name.length > 0) {
            document.getElementById('name').value = contact.name[0];
        }

        if (contact.email && contact.email.length > 0) {
            document.getElementById('email').value = contact.email[0];
        }

        if (contact.tel && contact.tel.length > 0) {
            document.getElementById('phone').value = contact.tel[0];
        }

        if (contact.address && contact.address.length > 0) {
            document.getElementById('address').value = contact.address[0];
        }
    });

    document.getElementById('contact-form').addEventListener('submit', (e) => {
        e.preventDefault();

        const formData = new FormData(e.target);
        const data = Object.fromEntries(formData);

        console.log('Saving contact:', data);
        alert('Contact saved successfully!');
    });
</script>

<style>
    .form-group {
        margin-bottom: 20px;
    }

    .form-group label {
        display: block;
        margin-bottom: 5px;
        font-weight: 500;
    }

    .form-group input,
    .form-group textarea {
        width: 100%;
        padding: 10px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 16px;
    }

    .input-with-button {
        display: flex;
        gap: 10px;
    }

    .input-with-button input {
        flex: 1;
    }

    .input-with-button button {
        padding: 10px 15px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        white-space: nowrap;
    }

    form > button[type="submit"] {
        width: 100%;
        padding: 12px;
        background: #10b981;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Email Composer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <div class="email-composer">
        <h2>Compose Email</h2>

        <div class="recipient-section">
            <label>To:</label>
            <div class="recipient-chips" id="recipients"></div>
            <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
                multiple: true
            }) }}>
                + Add Recipients
            </button>
        </div>

        <div class="form-group">
            <label for="subject">Subject:</label>
            <input type="text" id="subject" placeholder="Email subject">
        </div>

        <div class="form-group">
            <label for="message">Message:</label>
            <textarea id="message" rows="10" placeholder="Write your message..."></textarea>
        </div>

        <button id="send-email">Send Email</button>
    </div>
</div>

<script>
    let recipients = [];

    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;

        contacts.forEach(contact => {
            if (contact.email && contact.email.length > 0) {
                const email = contact.email[0];
                const name = contact.name ? contact.name[0] : email;

                // Check if not already added
                if (!recipients.find(r => r.email === email)) {
                    recipients.push({ name, email });
                }
            }
        });

        updateRecipients();
    });

    function updateRecipients() {
        const container = document.getElementById('recipients');

        if (recipients.length === 0) {
            container.innerHTML = '<span class="placeholder">No recipients</span>';
            return;
        }

        const chipsHtml = recipients.map((recipient, index) => `
            <div class="chip">
                <span>${recipient.name}</span>
                <button onclick="removeRecipient(${index})" class="chip-remove">×</button>
            </div>
        `).join('');

        container.innerHTML = chipsHtml;
    }

    function removeRecipient(index) {
        recipients.splice(index, 1);
        updateRecipients();
    }

    document.getElementById('send-email').addEventListener('click', () => {
        const subject = document.getElementById('subject').value;
        const message = document.getElementById('message').value;

        if (recipients.length === 0) {
            alert('Please add at least one recipient');
            return;
        }

        if (!subject || !message) {
            alert('Please fill in subject and message');
            return;
        }

        console.log('Sending email to:', recipients);
        console.log('Subject:', subject);
        console.log('Message:', message);

        alert(`Email sent to ${recipients.length} recipient(s)!`);
    });

    // Initial render
    updateRecipients();
</script>

<style>
    .email-composer {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    .recipient-section {
        margin-bottom: 20px;
        padding: 15px;
        background: #f9fafb;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .recipient-section label {
        display: block;
        margin-bottom: 10px;
        font-weight: 500;
    }

    .recipient-chips {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        min-height: 40px;
        margin-bottom: 10px;
    }

    .placeholder {
        color: #9ca3af;
        font-style: italic;
    }

    .chip {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        padding: 6px 12px;
        background: #3b82f6;
        color: white;
        border-radius: 16px;
        font-size: 14px;
    }

    .chip-remove {
        background: none;
        border: none;
        color: white;
        font-size: 20px;
        cursor: pointer;
        padding: 0;
        width: 20px;
        height: 20px;
        line-height: 1;
    }

    .recipient-section > button {
        padding: 8px 16px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
    }

    .form-group {
        margin-bottom: 20px;
    }

    .form-group label {
        display: block;
        margin-bottom: 5px;
        font-weight: 500;
    }

    .form-group input,
    .form-group textarea {
        width: 100%;
        padding: 10px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 16px;
        font-family: inherit;
    }

    #send-email {
        width: 100%;
        padding: 12px;
        background: #10b981;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Event Attendees Selector

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <div class="event-planner">
        <h2>Create Event</h2>

        <div class="form-group">
            <label for="event-name">Event Name</label>
            <input type="text" id="event-name" placeholder="Birthday Party">
        </div>

        <div class="form-group">
            <label for="event-date">Date & Time</label>
            <input type="datetime-local" id="event-date">
        </div>

        <div class="attendees-section">
            <h3>Attendees</h3>

            <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
                multiple: true
            }) }}>
                📱 Add from Contacts
            </button>

            <div id="attendees-list"></div>

            <div id="attendees-stats" class="stats"></div>
        </div>

        <button id="create-event">Create Event</button>
    </div>
</div>

<script>
    let attendees = [];

    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;

        contacts.forEach(contact => {
            const name = contact.name ? contact.name[0] : 'Unknown';
            const email = contact.email ? contact.email[0] : null;
            const phone = contact.tel ? contact.tel[0] : null;

            // Check if not already added
            const exists = attendees.find(a =>
                (email && a.email === email) ||
                (phone && a.phone === phone)
            );

            if (!exists) {
                attendees.push({
                    name,
                    email,
                    phone,
                    status: 'pending'
                });
            }
        });

        updateAttendeesList();
    });

    function updateAttendeesList() {
        const list = document.getElementById('attendees-list');
        const stats = document.getElementById('attendees-stats');

        if (attendees.length === 0) {
            list.innerHTML = '<p class="empty-state">No attendees added yet</p>';
            stats.innerHTML = '';
            return;
        }

        const attendeesHtml = attendees.map((attendee, index) => `
            <div class="attendee-card">
                <div class="attendee-avatar">${attendee.name.charAt(0).toUpperCase()}</div>
                <div class="attendee-info">
                    <strong>${attendee.name}</strong>
                    <div class="attendee-contact">
                        ${attendee.email ? `📧 ${attendee.email}` : ''}
                        ${attendee.phone ? `📱 ${attendee.phone}` : ''}
                    </div>
                </div>
                <select class="status-select" onchange="updateAttendeeStatus(${index}, this.value)">
                    <option value="pending" ${attendee.status === 'pending' ? 'selected' : ''}>Pending</option>
                    <option value="accepted" ${attendee.status === 'accepted' ? 'selected' : ''}>Accepted</option>
                    <option value="declined" ${attendee.status === 'declined' ? 'selected' : ''}>Declined</option>
                    <option value="tentative" ${attendee.status === 'tentative' ? 'selected' : ''}>Tentative</option>
                </select>
                <button class="remove-btn" onclick="removeAttendee(${index})">×</button>
            </div>
        `).join('');

        list.innerHTML = attendeesHtml;

        // Update stats
        const statuses = {
            pending: attendees.filter(a => a.status === 'pending').length,
            accepted: attendees.filter(a => a.status === 'accepted').length,
            declined: attendees.filter(a => a.status === 'declined').length,
            tentative: attendees.filter(a => a.status === 'tentative').length
        };

        stats.innerHTML = `
            <div class="stat">Total: <strong>${attendees.length}</strong></div>
            <div class="stat accepted">Accepted: <strong>${statuses.accepted}</strong></div>
            <div class="stat pending">Pending: <strong>${statuses.pending}</strong></div>
            <div class="stat tentative">Tentative: <strong>${statuses.tentative}</strong></div>
            <div class="stat declined">Declined: <strong>${statuses.declined}</strong></div>
        `;
    }

    function updateAttendeeStatus(index, status) {
        attendees[index].status = status;
        updateAttendeesList();
    }

    function removeAttendee(index) {
        attendees.splice(index, 1);
        updateAttendeesList();
    }

    document.getElementById('create-event').addEventListener('click', () => {
        const eventName = document.getElementById('event-name').value;
        const eventDate = document.getElementById('event-date').value;

        if (!eventName || !eventDate) {
            alert('Please fill in event name and date');
            return;
        }

        if (attendees.length === 0) {
            alert('Please add at least one attendee');
            return;
        }

        console.log('Creating event:', {
            name: eventName,
            date: eventDate,
            attendees: attendees
        });

        alert(`Event created with ${attendees.length} attendee(s)!`);
    });

    // Initial render
    updateAttendeesList();
</script>

<style>
    .event-planner {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    .form-group {
        margin-bottom: 20px;
    }

    .form-group label {
        display: block;
        margin-bottom: 5px;
        font-weight: 500;
    }

    .form-group input {
        width: 100%;
        padding: 10px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 16px;
    }

    .attendees-section {
        margin: 30px 0;
        padding: 20px;
        background: #f9fafb;
        border-radius: 8px;
    }

    .attendees-section h3 {
        margin: 0 0 15px 0;
    }

    .attendees-section > button {
        padding: 10px 20px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        margin-bottom: 20px;
    }

    .empty-state {
        text-align: center;
        color: #9ca3af;
        padding: 40px;
    }

    .attendee-card {
        display: flex;
        align-items: center;
        gap: 15px;
        padding: 15px;
        margin: 10px 0;
        background: white;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .attendee-avatar {
        width: 50px;
        height: 50px;
        border-radius: 50%;
        background: #3b82f6;
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 20px;
        font-weight: bold;
    }

    .attendee-info {
        flex: 1;
    }

    .attendee-info strong {
        display: block;
        margin-bottom: 5px;
    }

    .attendee-contact {
        font-size: 14px;
        color: #6b7280;
    }

    .status-select {
        padding: 6px 12px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 14px;
    }

    .remove-btn {
        width: 30px;
        height: 30px;
        border: none;
        background: #ef4444;
        color: white;
        border-radius: 4px;
        cursor: pointer;
        font-size: 20px;
        line-height: 1;
    }

    .stats {
        display: flex;
        gap: 15px;
        margin-top: 20px;
        padding-top: 15px;
        border-top: 1px solid #e5e7eb;
    }

    .stat {
        padding: 8px 12px;
        background: white;
        border-radius: 6px;
        font-size: 14px;
    }

    .stat.accepted {
        color: #10b981;
    }

    .stat.pending {
        color: #f59e0b;
    }

    .stat.tentative {
        color: #3b82f6;
    }

    .stat.declined {
        color: #ef4444;
    }

    #create-event {
        width: 100%;
        padding: 12px;
        background: #10b981;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Contact Import for CRM

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <div class="crm-import">
        <h2>Import Contacts to CRM</h2>

        <div class="import-controls">
            <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
                multiple: true
            }) }}>
                📱 Select Contacts to Import
            </button>

            <div class="filters">
                <label>
                    <input type="checkbox" id="filter-email" checked> Only with email
                </label>
                <label>
                    <input type="checkbox" id="filter-phone" checked> Only with phone
                </label>
            </div>
        </div>

        <div id="import-preview" style="display: none;">
            <h3>Import Preview</h3>
            <div id="contacts-preview"></div>

            <div class="import-actions">
                <button id="confirm-import">Import <span id="import-count">0</span> Contact(s)</button>
                <button id="cancel-import">Cancel</button>
            </div>
        </div>

        <div id="import-results" style="display: none;"></div>
    </div>
</div>

<script>
    let importQueue = [];

    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts } = event.detail;
        const filterEmail = document.getElementById('filter-email').checked;
        const filterPhone = document.getElementById('filter-phone').checked;

        // Filter contacts based on criteria
        importQueue = contacts.filter(contact => {
            if (filterEmail && (!contact.email || contact.email.length === 0)) {
                return false;
            }
            if (filterPhone && (!contact.tel || contact.tel.length === 0)) {
                return false;
            }
            return true;
        }).map(contact => ({
            name: contact.name ? contact.name[0] : 'Unknown',
            emails: contact.email || [],
            phones: contact.tel || [],
            addresses: contact.address || []
        }));

        if (importQueue.length === 0) {
            alert('No contacts match the selected filters');
            return;
        }

        showImportPreview();
    });

    function showImportPreview() {
        const preview = document.getElementById('import-preview');
        const contactsPreview = document.getElementById('contacts-preview');

        document.getElementById('import-count').textContent = importQueue.length;

        const previewHtml = importQueue.map((contact, index) => `
            <div class="contact-preview-card">
                <div class="contact-number">${index + 1}</div>
                <div class="contact-details">
                    <strong>${contact.name}</strong>
                    ${contact.emails.length > 0 ? `<div>📧 ${contact.emails.join(', ')}</div>` : ''}
                    ${contact.phones.length > 0 ? `<div>📱 ${contact.phones.join(', ')}</div>` : ''}
                    ${contact.addresses.length > 0 ? `<div>📍 ${contact.addresses.join(', ')}</div>` : ''}
                </div>
            </div>
        `).join('');

        contactsPreview.innerHTML = previewHtml;
        preview.style.display = 'block';
    }

    document.getElementById('confirm-import').addEventListener('click', async () => {
        const results = document.getElementById('import-results');
        results.style.display = 'block';
        results.innerHTML = '<p>Importing contacts...</p>';

        // Simulate import process
        let successful = 0;
        let failed = 0;

        for (let i = 0; i < importQueue.length; i++) {
            // Simulate API call
            await new Promise(resolve => setTimeout(resolve, 100));

            try {
                // In a real app, you would make an API call here
                console.log('Importing contact:', importQueue[i]);
                successful++;
            } catch (error) {
                console.error('Failed to import contact:', error);
                failed++;
            }
        }

        results.innerHTML = `
            <div class="import-success">
                <h3>✅ Import Complete</h3>
                <p><strong>${successful}</strong> contact(s) imported successfully</p>
                ${failed > 0 ? `<p class="error">${failed} contact(s) failed to import</p>` : ''}
            </div>
        `;

        document.getElementById('import-preview').style.display = 'none';
        importQueue = [];
    });

    document.getElementById('cancel-import').addEventListener('click', () => {
        document.getElementById('import-preview').style.display = 'none';
        importQueue = [];
    });
</script>

<style>
    .crm-import {
        max-width: 900px;
        margin: 0 auto;
        padding: 20px;
    }

    .import-controls {
        margin-bottom: 30px;
    }

    .import-controls > button {
        padding: 12px 24px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        cursor: pointer;
        margin-bottom: 15px;
    }

    .filters {
        display: flex;
        gap: 20px;
    }

    .filters label {
        display: flex;
        align-items: center;
        gap: 8px;
        cursor: pointer;
    }

    #import-preview {
        margin: 30px 0;
    }

    #contacts-preview {
        max-height: 400px;
        overflow-y: auto;
        margin: 20px 0;
        padding: 15px;
        background: #f9fafb;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .contact-preview-card {
        display: flex;
        gap: 15px;
        padding: 15px;
        margin: 10px 0;
        background: white;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
    }

    .contact-number {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background: #3b82f6;
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
    }

    .contact-details {
        flex: 1;
    }

    .contact-details strong {
        display: block;
        margin-bottom: 5px;
        font-size: 16px;
    }

    .contact-details div {
        font-size: 14px;
        color: #6b7280;
        margin: 3px 0;
    }

    .import-actions {
        display: flex;
        gap: 10px;
    }

    .import-actions button {
        padding: 12px 24px;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }

    #confirm-import {
        background: #10b981;
        color: white;
    }

    #cancel-import {
        background: #6b7280;
        color: white;
    }

    .import-success {
        padding: 30px;
        background: #f0fdf4;
        border: 2px solid #10b981;
        border-radius: 8px;
        text-align: center;
    }

    .import-success h3 {
        margin: 0 0 15px 0;
        color: #10b981;
    }

    .import-success .error {
        color: #ef4444;
        margin-top: 10px;
    }
</style>
```
{% endcode %}

## Parameters

None - The `select` action accepts parameters to control the selection behavior.

## Actions

### `select`

Opens the native contact picker dialog.

**Options:**
* `multiple` (boolean, optional): If `true`, allows selecting multiple contacts. Default: `false`

```twig
{{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
    multiple: true
}) }}
```

Example for single contact:
```twig
<button {{ stimulus_action('@pwa/contact-picker', 'select', 'click') }}>
    Select One Contact
</button>
```

Example for multiple contacts:
```twig
<button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
    multiple: true
}) }}>
    Select Multiple Contacts
</button>
```

## Targets

None

## Events

### `pwa--contact-picker:selection`

Dispatched when the user successfully selects one or more contacts from the picker.

**Payload**: `{contacts: Array<Contact>}`

Each contact object may include:
* `name` (string[]|null): Array of name strings (may contain multiple names)
* `email` (string[]|null): Array of email addresses
* `tel` (string[]|null): Array of phone numbers
* `address` (string[]|null): Array of physical addresses
* `icon` (Blob|null): Contact's photo/avatar as a Blob (if available and supported)

{% hint style="info" %}
All contact properties are optional and may be null or empty arrays depending on what information is available in the device's contact book.
{% endhint %}

Example:
```javascript
document.addEventListener('pwa--contact-picker:selection', (event) => {
    const { contacts } = event.detail;

    contacts.forEach(contact => {
        console.log('Name:', contact.name ? contact.name[0] : 'N/A');
        console.log('Emails:', contact.email);
        console.log('Phones:', contact.tel);
        console.log('Addresses:', contact.address);

        // Handle contact icon if available
        if (contact.icon) {
            const iconUrl = URL.createObjectURL(contact.icon);
            // Use iconUrl for display
        }
    });
});
```

### `pwa--contact-picker:error`

Dispatched when an error occurs during contact selection or when the user cancels the picker.

**Payload**: `{exception: Error}`

Common causes:
* User cancelled the picker
* Permission denied
* API error

Example:
```javascript
document.addEventListener('pwa--contact-picker:error', (event) => {
    const { exception } = event.detail;

    if (exception.name === 'AbortError') {
        console.log('User cancelled contact selection');
    } else {
        console.error('Contact picker error:', exception.message);
    }
});
```

### `pwa--contact-picker:unavailable`

Dispatched when the Contact Picker API is not available on the current browser/platform.

**No payload**

Example:
```javascript
document.addEventListener('pwa--contact-picker:unavailable', () => {
    console.log('Contact Picker not available on this device');

    // Show manual input form
    document.getElementById('manual-input-form').style.display = 'block';
    document.getElementById('contact-picker-button').style.display = 'none';
});
```

## Best Practices

1. **Always provide fallback**: Not all platforms support Contact Picker - provide manual input
2. **Handle empty data**: Contact properties can be null or empty - always check before using
3. **Request only what you need**: The API provides basic contact info, not full contact details
4. **Respect user privacy**: Only use selected contact data for its intended purpose
5. **Clear selection state**: Allow users to modify or remove selected contacts
6. **Validate input**: Don't assume contact data is valid (emails might be malformed, etc.)
7. **Handle cancellation gracefully**: Users might cancel the picker - don't treat as error
8. **Test on real devices**: Contact Picker behavior varies between devices
9. **Use progressive enhancement**: Build core functionality without Contact Picker first
10. **Inform users**: Explain why you need contact access and what you'll do with it

## Handling Contact Data

### Extracting Primary Contact Info

```javascript
function getPrimaryContactInfo(contact) {
    return {
        name: contact.name && contact.name.length > 0 ? contact.name[0] : null,
        email: contact.email && contact.email.length > 0 ? contact.email[0] : null,
        phone: contact.tel && contact.tel.length > 0 ? contact.tel[0] : null,
        address: contact.address && contact.address.length > 0 ? contact.address[0] : null
    };
}
```

### Validating Contact Data

```javascript
function validateContact(contact) {
    const errors = [];

    if (!contact.name || contact.name.length === 0) {
        errors.push('Contact must have a name');
    }

    if (!contact.email || contact.email.length === 0) {
        errors.push('Contact must have at least one email');
    }

    if (contact.email) {
        contact.email.forEach(email => {
            if (!isValidEmail(email)) {
                errors.push(`Invalid email: ${email}`);
            }
        });
    }

    return {
        isValid: errors.length === 0,
        errors
    };
}

function isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### Deduplicating Contacts

```javascript
function deduplicateContacts(contacts) {
    const seen = new Set();
    const unique = [];

    contacts.forEach(contact => {
        // Create unique key based on email or phone
        const email = contact.email && contact.email.length > 0 ? contact.email[0] : null;
        const phone = contact.tel && contact.tel.length > 0 ? contact.tel[0] : null;

        const key = email || phone;

        if (key && !seen.has(key)) {
            seen.add(key);
            unique.push(contact);
        }
    });

    return unique;
}
```

## Complete Example: Contact Management App

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/contact-picker') }}>
    <div class="contact-manager">
        <header class="app-header">
            <h1>Contact Manager</h1>
            <button {{ stimulus_action('@pwa/contact-picker', 'select', 'click', {
                multiple: true
            }) }}>
                📱 Import from Device
            </button>
        </header>

        <div class="search-bar">
            <input type="text" id="search" placeholder="Search contacts...">
        </div>

        <div class="contacts-grid" id="contacts-grid">
            <p class="empty-state">No contacts yet. Import from your device to get started.</p>
        </div>

        <div id="contact-modal" class="modal" style="display: none;">
            <div class="modal-content">
                <span class="close" onclick="closeModal()">&times;</span>
                <h2 id="modal-title">Contact Details</h2>
                <div id="modal-body"></div>
            </div>
        </div>
    </div>
</div>

<script>
    let contacts = JSON.parse(localStorage.getItem('contacts') || '[]');

    document.addEventListener('pwa--contact-picker:selection', (event) => {
        const { contacts: newContacts } = event.detail;

        newContacts.forEach(contact => {
            const formattedContact = {
                id: Date.now() + Math.random(),
                name: contact.name ? contact.name[0] : 'Unknown',
                emails: contact.email || [],
                phones: contact.tel || [],
                addresses: contact.address || [],
                importedAt: new Date().toISOString()
            };

            // Check for duplicates
            const isDuplicate = contacts.some(c =>
                (formattedContact.emails.length > 0 && c.emails.includes(formattedContact.emails[0])) ||
                (formattedContact.phones.length > 0 && c.phones.includes(formattedContact.phones[0]))
            );

            if (!isDuplicate) {
                contacts.push(formattedContact);
            }
        });

        saveContacts();
        renderContacts();
    });

    document.addEventListener('pwa--contact-picker:unavailable', () => {
        alert('Contact Picker is not available on this device. You can still add contacts manually.');
    });

    document.getElementById('search').addEventListener('input', (e) => {
        const query = e.target.value.toLowerCase();
        renderContacts(query);
    });

    function renderContacts(searchQuery = '') {
        const grid = document.getElementById('contacts-grid');

        let filtered = contacts;

        if (searchQuery) {
            filtered = contacts.filter(contact =>
                contact.name.toLowerCase().includes(searchQuery) ||
                contact.emails.some(email => email.toLowerCase().includes(searchQuery)) ||
                contact.phones.some(phone => phone.includes(searchQuery))
            );
        }

        if (filtered.length === 0) {
            grid.innerHTML = '<p class="empty-state">No contacts found.</p>';
            return;
        }

        const cardsHtml = filtered.map(contact => `
            <div class="contact-card" onclick="showContactDetails('${contact.id}')">
                <div class="contact-avatar">${contact.name.charAt(0).toUpperCase()}</div>
                <div class="contact-info">
                    <h3>${contact.name}</h3>
                    ${contact.emails.length > 0 ? `<p>📧 ${contact.emails[0]}</p>` : ''}
                    ${contact.phones.length > 0 ? `<p>📱 ${contact.phones[0]}</p>` : ''}
                </div>
                <button class="delete-btn" onclick="event.stopPropagation(); deleteContact('${contact.id}')">
                    🗑️
                </button>
            </div>
        `).join('');

        grid.innerHTML = cardsHtml;
    }

    function showContactDetails(contactId) {
        const contact = contacts.find(c => c.id == contactId);
        if (!contact) return;

        const modal = document.getElementById('contact-modal');
        const modalTitle = document.getElementById('modal-title');
        const modalBody = document.getElementById('modal-body');

        modalTitle.textContent = contact.name;

        modalBody.innerHTML = `
            <div class="detail-section">
                <h3>Emails</h3>
                ${contact.emails.length > 0 ?
                    contact.emails.map(email => `<p><a href="mailto:${email}">${email}</a></p>`).join('') :
                    '<p class="no-data">No emails</p>'
                }
            </div>

            <div class="detail-section">
                <h3>Phone Numbers</h3>
                ${contact.phones.length > 0 ?
                    contact.phones.map(phone => `<p><a href="tel:${phone}">${phone}</a></p>`).join('') :
                    '<p class="no-data">No phone numbers</p>'
                }
            </div>

            <div class="detail-section">
                <h3>Addresses</h3>
                ${contact.addresses.length > 0 ?
                    contact.addresses.map(addr => `<p>${addr}</p>`).join('') :
                    '<p class="no-data">No addresses</p>'
                }
            </div>

            <div class="detail-section">
                <h3>Import Info</h3>
                <p>Imported: ${new Date(contact.importedAt).toLocaleString()}</p>
            </div>
        `;

        modal.style.display = 'block';
    }

    function closeModal() {
        document.getElementById('contact-modal').style.display = 'none';
    }

    function deleteContact(contactId) {
        if (!confirm('Are you sure you want to delete this contact?')) {
            return;
        }

        contacts = contacts.filter(c => c.id != contactId);
        saveContacts();
        renderContacts();
    }

    function saveContacts() {
        localStorage.setItem('contacts', JSON.stringify(contacts));
    }

    // Close modal when clicking outside
    window.onclick = function(event) {
        const modal = document.getElementById('contact-modal');
        if (event.target == modal) {
            closeModal();
        }
    }

    // Initial render
    renderContacts();
</script>

<style>
    .contact-manager {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
    }

    .app-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 30px;
    }

    .app-header button {
        padding: 12px 24px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        cursor: pointer;
    }

    .search-bar {
        margin-bottom: 30px;
    }

    .search-bar input {
        width: 100%;
        padding: 12px 20px;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        font-size: 16px;
    }

    .contacts-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 20px;
    }

    .empty-state {
        grid-column: 1 / -1;
        text-align: center;
        padding: 60px 20px;
        color: #9ca3af;
        font-size: 18px;
    }

    .contact-card {
        display: flex;
        align-items: center;
        gap: 15px;
        padding: 20px;
        background: white;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.2s;
    }

    .contact-card:hover {
        border-color: #3b82f6;
        transform: translateY(-2px);
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }

    .contact-avatar {
        width: 60px;
        height: 60px;
        border-radius: 50%;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 24px;
        font-weight: bold;
    }

    .contact-info {
        flex: 1;
    }

    .contact-info h3 {
        margin: 0 0 5px 0;
        font-size: 18px;
    }

    .contact-info p {
        margin: 3px 0;
        font-size: 14px;
        color: #6b7280;
    }

    .delete-btn {
        padding: 8px;
        background: #fee2e2;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 18px;
        transition: background 0.2s;
    }

    .delete-btn:hover {
        background: #ef4444;
    }

    .modal {
        display: none;
        position: fixed;
        z-index: 1000;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
    }

    .modal-content {
        position: relative;
        background-color: white;
        margin: 5% auto;
        padding: 30px;
        border-radius: 12px;
        width: 90%;
        max-width: 600px;
        max-height: 80vh;
        overflow-y: auto;
    }

    .close {
        position: absolute;
        right: 20px;
        top: 20px;
        font-size: 28px;
        font-weight: bold;
        color: #9ca3af;
        cursor: pointer;
    }

    .close:hover {
        color: #1f2937;
    }

    .detail-section {
        margin: 25px 0;
        padding-bottom: 20px;
        border-bottom: 1px solid #e5e7eb;
    }

    .detail-section:last-child {
        border-bottom: none;
    }

    .detail-section h3 {
        margin: 0 0 15px 0;
        color: #6b7280;
        font-size: 14px;
        text-transform: uppercase;
    }

    .detail-section p {
        margin: 8px 0;
        font-size: 16px;
    }

    .detail-section a {
        color: #3b82f6;
        text-decoration: none;
    }

    .detail-section a:hover {
        text-decoration: underline;
    }

    .no-data {
        color: #9ca3af;
        font-style: italic;
    }
</style>
```
{% endcode %}

## Troubleshooting

### API not available

**Issue**: Contact Picker not working

**Platform limitations**:
- Only works on Android with Chrome/Edge
- Not available on iOS, desktop, or other browsers

**Solution**: Always provide manual input fallback:
```javascript
document.addEventListener('pwa--contact-picker:unavailable', () => {
    showManualInputForm();
});
```

### User cancels picker

**Issue**: Error event fires when user cancels

**Cause**: Cancellation is treated as an `AbortError`

**Solution**: Handle gracefully:
```javascript
document.addEventListener('pwa--contact-picker:error', (event) => {
    if (event.detail.exception.name === 'AbortError') {
        // User cancelled - this is normal behavior
        return;
    }
    // Handle actual errors
});
```

### Empty contact data

**Issue**: Selected contacts have no email/phone

**Cause**: Not all contacts in device address book have complete information

**Solution**: Validate and filter:
```javascript
const validContacts = contacts.filter(contact =>
    (contact.email && contact.email.length > 0) ||
    (contact.tel && contact.tel.length > 0)
);
```

### Icon/photo not loading

**Issue**: Contact icon property is not available

**Cause**: Limited support for contact photos

**Solution**: Use fallback avatars with initials:
```javascript
if (contact.icon) {
    const iconUrl = URL.createObjectURL(contact.icon);
    // Use icon
} else {
    // Show initials avatar
    const initial = contact.name[0].charAt(0);
}
```

## Platform Compatibility

| Platform | Support |
|----------|---------|
| Chrome/Edge (Android) | ✓ Full support |
| Chrome/Edge (Desktop) | ✗ Not supported |
| Safari (iOS) | ✗ Not supported |
| Safari (Desktop) | ✗ Not supported |
| Firefox | ✗ Not supported |

{% hint style="danger" %}
Due to extremely limited platform support, the Contact Picker API should be treated as a progressive enhancement. Always provide alternative ways to input contact information.
{% endhint %}

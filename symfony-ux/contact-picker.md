# Contact Picker

This controller uses the Contacts Picker API\
to let users select one or more contacts directly from their device’s native address book.\
It emits events for availability, contact selection, and errors.

**Usage**

```twig
<div {{ stimulus_controller('@pwa/contact') }}>
  <button data-action="@pwa/contact#select">Select contact</button>
  <pre id="contacts-output">No contacts selected yet.</pre>
</div>

<script type="module">
  const el = document.querySelector('[data-controller="pwa__contacts"]');
  const out = document.getElementById('contacts-output');

  el.addEventListener('contacts:unavailable', () => {
    out.textContent = 'The Contacts Picker API is not available on this device.';
  });

  el.addEventListener('contacts:selection', (e) => {
    const { contacts } = e.detail;
    out.textContent = JSON.stringify(contacts, null, 2);
  });

  el.addEventListener('contacts:error', (e) => {
    out.textContent = `Error: ${e.detail.exception.message}`;
  });
</script>
```

### Parameters

None

### Actions

`select`: Opens the native contact picker. This action accepts the parameter `multiple` (boolean) to allow multiple contact selection.

### Targets

None

### Events

`unavailable`: The API is not supported on the current browser or platform.

`error`: The user cancels the picker or an exception occurs.

`selection`: The user successfully selects one or more contacts. Contains a list of contact objects.&#x20;

Each contact object may include:

* `name`  (string\[]|null)
* `email`  (string\[]|null)
* `tel`  (string\[]|null)
* `address`  (string\[]|null)
* `icon` (a blob if supported)

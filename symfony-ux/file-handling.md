# File Handling

This controller handles files or links passed to your Progressive Web App when it is launched via the Launch Queue API.\
It allows your app to receive files opened by the user through the system (for example, “Open with…” a PWA).

You application must declare the appropriate [file handlers](../the-manifest/file-handlers.md).

**Usage**

```twig
<div {{ stimulus_controller('@pwa/file-handling') }}>
  <p>Waiting for files to open…</p>
</div

<script type="module">
  const el = document.querySelector('[data-controller="launch-queue"]');

  el.addEventListener('pwa__file-handling:selected', (e) => {
    const { data } = e.detail;
    console.log('File received via Launch Queue:', data);

    // Example: preview in an <img>
    const img = document.createElement('img');
    img.src = data;
    document.body.appendChild(img);
  });
</script>
```

### Parameters

None

### Actions

None

### Targets

None

### Events

`selected`: The app is launched with one or more files (triggered for each file).

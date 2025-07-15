---
description: Display a badge on the app icon
---

# Badge

Applications can show badges on their icons to notify users of new content. This content may include unread emails, notifications, or new blog posts.

In the following example, the badge is set when the page is loaded. Feel free to dynamically enable this feature.

```twig
<body
    data-controller="pwa--badge"
    data-action="load@window->pwa--badge#update"
    data-pwa--badge-counter-param="42"
>
    ...
</body>
```

### Parameters

None

### Actions

`update`: this action sets a new value `counter` that is passed as argument

`clear`: removes the badge

### Targets

None

### Events

On status change, the event `pwa--badge:updated` is dispatched. The payload property contains the following data:

* `counter`: the value set to the badge.

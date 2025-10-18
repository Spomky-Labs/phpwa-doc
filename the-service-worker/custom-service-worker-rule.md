# Custom Service Worker Rule

If needed, you can define custom sections in the service worker appended by your own services and depending on your application configuration or requirements.

To do so, you can create a service that implements `SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRule`.

{% hint style="info" %}
The method process shall return valid JS as a string. This script will be executed by browsers.
{% endhint %}

```php
<?php

declare(strict_types=1);

namespace Acme;

use SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRule;

final readonly class MyCustomRule implements ServiceWorkerRule
{
    private Workbox $workbox;

    public function __construct(
        ServiceWorker $serviceWorker,
    ) {
        $this->workbox = $serviceWorker->workbox;
    }

    public function process(bool $debug = false): string
    {
        return <<<HELLO_WORLD
// This will be added to the Service Worker
console.log('FOO-BAR from the Service Worker!');
HELLO_WORLD;
    }
}
```

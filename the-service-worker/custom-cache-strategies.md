# Custom Cache Strategies

The bundle provides a large set of cache strategies. However, you can define your own strategies if you need a specific behavior that is not already supported.

With a service that implement `SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface` you will be able to return cache strategy objects.

{% hint style="info" %}
The method process shall return valid JS as a string. This script will be executed by browsers.
{% endhint %}

{% code title="src/MyCustomStrategies.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace Acme;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface;

final readonly class MyCustomStrategies implements HasCacheStrategiesInterface
{
    /**
     * @return array<CacheStrategyInterface>
     */
    public function getCacheStrategies(): array
    {
        return [
            ... // You cache strategies here
        ];
    }
}
```
{% endcode %}

If the service autoconfiguration is enable, the tag will be added automatically. Otherwise, you need to add the tag `spomky_labs_pwa.cache_strategy`.

Cache Strategies are objects that implement `CacheStrategyInterface`. The most important function is the rendered JS that will be injected to the Service Worker.

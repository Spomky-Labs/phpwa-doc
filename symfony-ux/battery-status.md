# Battery Status

The battery status indicates the current battery level. This status is updated when the component is connected and whenever there are changes.

In the following example, a Twig component will display the information received from the Stimulus Controller.

{% code title="src/Twig/Component/Battery.php" lineNumbers="true" %}
```php
<?php

namespace App\Twig\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveArg;
use Symfony\UX\LiveComponent\Attribute\LiveListener;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentToolsTrait;

#[AsLiveComponent('Battery')]
class Battery
{
    use DefaultActionTrait;
    use ComponentToolsTrait;

    #[LiveProp]
    public string $charging = '';
    #[LiveProp]
    public string $level = '';
    #[LiveProp]
    public string $chargingTime = '';
    #[LiveProp]
    public string $dischargingTime = '';

    #[LiveListener('updated')]
    public function onUpdate(
        #[LiveArg] bool $charging,
        #[LiveArg] float $level,
        #[LiveArg] ?int $chargingTime,
        #[LiveArg] ?int $dischargingTime,
    ): void {
        $this->charging = $charging ? 'Yes' : 'No';
        $this->level = ($level* 100) . '%';
        $this->chargingTime = $this->formatSeconds($chargingTime);
        $this->dischargingTime = $this->formatSeconds($dischargingTime);
    }

    private function formatSeconds(?float $s): string
    {
        if ($s === null ||!is_finite($s)) {
            return '∞';
        }

        if ($s === 0.0) {
            return 'done';
        }

        $h = str_pad((string) floor($s / 3600), 2, '0', STR_PAD_LEFT);
        $m = str_pad((string) floor(($s % 3600) / 60), 2, '0', STR_PAD_LEFT);
        $sec = str_pad((string) floor($s % 60), 2, '0', STR_PAD_LEFT);

        return "$h:$m:$sec";
    }
}
```
{% endcode %}

{% code title="templates/components/Battery.html.twig" lineNumbers="true" %}
```twig
<div data-controller="pwa--battery live"{{ attributes }}>
    <dl class="divide-y divide-gray-100">
        <div  class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Charging</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{{ this.charging }}</dd>
        </div>
        <div  class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Level</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{{ this.level }}</dd>
        </div>
        <div  class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Discharging Time</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{{ this.dischargingTime }}</dd>
        </div>
        <div  class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Charging Time</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{{ this.chargingTime }}</dd>
        </div>
    </dl>
</div>

```
{% endcode %}

Next, simply place the live component where needed in your templates.

```twig
<twig:Battery />
```

### Parameters

None

### Actions

None

### Targets

None

### Events

Upon status change, the event `pwa--battery:updated` is triggered. The payload includes the following data:

* `charging`: indicates whether the battery is charging or not.
* `level`: the current battery level.
* `chargingTime`: the time needed to recharge the battery from empty to full.
* `dischargingTime`: the time the battery can last before being completely discharged.

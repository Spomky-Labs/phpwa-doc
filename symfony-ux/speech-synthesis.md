# Speech Synthesis

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Speech Synthesis component provides a comprehensive interface to the Web Speech API's text-to-speech functionality, allowing your Progressive Web App to convert text into spoken words. This enables accessible, hands-free experiences and enhanced user interactions.

This component is particularly useful for:

* Accessibility features for users with visual impairments or reading difficulties
* Educational applications (language learning, pronunciation guides)
* Navigation and turn-by-turn directions
* Content reading (news articles, books, messages)
* Voice-enabled interfaces and assistive technologies
* Multi-language support with native voice selection

## Browser Support

The Speech Synthesis API is widely supported across modern browsers:

* **Chrome/Edge**: Full support with extensive voice libraries
* **Safari**: Full support with high-quality voices
* **Firefox**: Full support with system voices
* **Mobile browsers**: Excellent support on both iOS and Android

{% hint style="info" %}
Voice availability varies by platform and language. Desktop browsers typically offer more voices than mobile browsers. The controller automatically handles voice detection and selection.
{% endhint %}

## Usage

### Basic Text-to-Speech

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <p>Enter text to be spoken:</p>
    <textarea id="speech-text">Hello! Welcome to our Progressive Web App.</textarea>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'Hello! Welcome to our Progressive Web App.'
    }) }}>
        Speak Text
    </button>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'status') }}></div>
</div>

<script>
    document.addEventListener('pwa:speech-synthesis:start', (event) => {
        console.log('Speaking:', event.detail.text);
    });

    document.addEventListener('pwa:speech-synthesis:end', () => {
        console.log('Finished speaking');
    });
</script>
```
{% endcode %}

### Speaking HTML Elements

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <h2>Article Reader</h2>

    <!-- Individual paragraphs with item target -->
    <article>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
           {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
           style="cursor: pointer;">
            Click this paragraph to hear it read aloud. The Speech Synthesis API
            makes it easy to add text-to-speech to any web application.
        </p>

        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
           {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
           style="cursor: pointer;">
            You can customize the voice, speed, pitch, and volume for each piece
            of content to match your application's needs.
        </p>
    </article>

    <!-- Read all items button -->
    <button {{ stimulus_action('@pwa/speech-synthesis', 'enqueueItems', 'click') }}>
        Read Entire Article
    </button>
</div>
```
{% endcode %}

### Voice Selection with Custom Parameters

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    locale: 'en-US',
    rate: 1,
    pitch: 1,
    volume: 1
}) }}>
    <h3>Voice Settings</h3>

    <!-- Voice selector (automatically populated) -->
    <label>Voice:</label>
    <select {{ stimulus_target('@pwa/speech-synthesis', 'voiceSelect') }}
            {{ stimulus_action('@pwa/speech-synthesis', 'changeVoiceFromSelect', 'change') }}>
    </select>

    <!-- Speech rate control -->
    <label>Speed: <span id="rate-display">1</span>x</label>
    <input type="range" min="0.5" max="2" step="0.1" value="1"
           {{ stimulus_action('@pwa/speech-synthesis', 'setRate', 'input', {
               rate: '@$el.valueAsNumber'
           }) }}
           oninput="document.getElementById('rate-display').textContent = this.value">

    <!-- Pitch control -->
    <label>Pitch: <span id="pitch-display">1</span></label>
    <input type="range" min="0" max="2" step="0.1" value="1"
           {{ stimulus_action('@pwa/speech-synthesis', 'setPitch', 'input', {
               pitch: '@$el.valueAsNumber'
           }) }}
           oninput="document.getElementById('pitch-display').textContent = this.value">

    <!-- Volume control -->
    <label>Volume: <span id="volume-display">100</span>%</label>
    <input type="range" min="0" max="1" step="0.1" value="1"
           {{ stimulus_action('@pwa/speech-synthesis', 'setVolume', 'input', {
               volume: '@$el.valueAsNumber'
           }) }}
           oninput="document.getElementById('volume-display').textContent = Math.round(this.value * 100)">

    <textarea id="custom-text">This text will be spoken with custom settings.</textarea>
    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: '@#custom-text.value'
    }) }}>
        Speak with Custom Settings
    </button>
</div>
```
{% endcode %}

### Multi-Language Content

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <h3>Multi-Language Reader</h3>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="en-US"
         data-speech-text="Hello, how are you?"
         {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
         style="cursor: pointer; padding: 10px; border: 1px solid #ccc; margin: 5px;">
        🇺🇸 English: "Hello, how are you?"
    </div>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="fr-FR"
         data-speech-text="Bonjour, comment allez-vous ?"
         {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
         style="cursor: pointer; padding: 10px; border: 1px solid #ccc; margin: 5px;">
        🇫🇷 French: "Bonjour, comment allez-vous ?"
    </div>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="es-ES"
         data-speech-text="Hola, ¿cómo estás?"
         {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
         style="cursor: pointer; padding: 10px; border: 1px solid #ccc; margin: 5px;">
        🇪🇸 Spanish: "Hola, ¿cómo estás?"
    </div>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="de-DE"
         data-speech-text="Hallo, wie geht es dir?"
         {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
         style="cursor: pointer; padding: 10px; border: 1px solid #ccc; margin: 5px;">
        🇩🇪 German: "Hallo, wie geht es dir?"
    </div>
</div>
```
{% endcode %}

### Playback Controls

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: true
}) }}>
    <h3>Audio Book Player</h3>

    <!-- Content to be read -->
    <div id="book-content">
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>
            Chapter One: The Beginning. It was a dark and stormy night when our story begins.
        </p>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>
            Chapter Two: The Journey. The protagonist set out on an adventure that would change everything.
        </p>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>
            Chapter Three: The Discovery. What they found would alter their understanding of the world.
        </p>
    </div>

    <!-- Playback controls -->
    <div style="margin-top: 20px;">
        <button {{ stimulus_action('@pwa/speech-synthesis', 'enqueueItems', 'click') }}>
            ▶️ Play All
        </button>

        <button {{ stimulus_action('@pwa/speech-synthesis', 'pause', 'click') }}>
            ⏸️ Pause
        </button>

        <button {{ stimulus_action('@pwa/speech-synthesis', 'resume', 'click') }}>
            ⏯️ Resume
        </button>

        <button {{ stimulus_action('@pwa/speech-synthesis', 'cancel', 'click') }}>
            ⏹️ Stop
        </button>
    </div>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'status') }}
         style="margin-top: 10px; font-weight: bold;"></div>
</div>

<script>
    document.addEventListener('pwa:speech-synthesis:queued', (event) => {
        console.log('Queue size:', event.detail.size);
    });

    document.addEventListener('pwa:speech-synthesis:dequeue', (event) => {
        console.log('Remaining items:', event.detail.remaining);
    });
</script>
```
{% endcode %}

### Custom Internationalization

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    i18n: {
        loading: 'Chargement des voix…',
        ready: 'Prêt',
        unsupported: 'La synthèse vocale n\'est pas supportée.',
        playing: 'Lecture en cours',
        paused: 'En pause',
        canceled: 'Annulé',
        finished: 'Terminé'
    }
}) }}>
    <h3>Lecteur Français</h3>

    <div {{ stimulus_target('@pwa/speech-synthesis', 'status') }}></div>

    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-locale="fr-FR"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
        Cliquez pour écouter ce texte en français.
    </p>
</div>
```
{% endcode %}

### Per-Item Custom Parameters

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <h3>Dynamic Speech Parameters</h3>

    <!-- Normal speech -->
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-text="This is normal speed speech."
       data-speech-rate="1"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
       style="cursor: pointer;">
        Normal Speed (1x)
    </p>

    <!-- Fast speech -->
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-text="This is fast speech for quick reading."
       data-speech-rate="1.5"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
       style="cursor: pointer;">
        Fast Speed (1.5x)
    </p>

    <!-- Slow speech -->
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-text="This is slow speech for careful listening."
       data-speech-rate="0.75"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
       style="cursor: pointer;">
        Slow Speed (0.75x)
    </p>

    <!-- High pitch -->
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-text="This text has a higher pitch."
       data-speech-pitch="1.5"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
       style="cursor: pointer;">
        High Pitch
    </p>

    <!-- Low volume -->
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
       data-speech-text="This text is quieter."
       data-speech-volume="0.5"
       {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}
       style="cursor: pointer;">
        Low Volume (50%)
    </p>
</div>
```
{% endcode %}

### Immediate vs Queue Mode

{% code lineNumbers="true" %}
```twig
<!-- Queue mode (default): Utterances are queued and played sequentially -->
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: true
}) }}>
    <h3>Queue Mode</h3>
    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'First message'
    }) }}>Speak First</button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'Second message'
    }) }}>Speak Second</button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'Third message'
    }) }}>Speak Third</button>

    <p><small>Click multiple buttons - all will be spoken in order</small></p>
</div>

<!-- Immediate mode: New speech cancels previous -->
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: false
}) }}>
    <h3>Immediate Mode</h3>
    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'First message'
    }) }}>Speak First</button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'Second message'
    }) }}>Speak Second</button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
        text: 'Third message'
    }) }}>Speak Third</button>

    <p><small>Click multiple buttons - only the latest will be spoken</small></p>
</div>
```
{% endcode %}

## Common Use Cases

### 1. Accessible News Reader

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    rate: 1.2,
    enqueue: true
}) }}>
    <article>
        <h1 {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
            data-speech-rate="0.9">
            Breaking News: Important Announcement
        </h1>

        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>
            Today's top story covers significant developments in technology...
        </p>

        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>
            Experts believe this will change the industry forever...
        </p>
    </article>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'enqueueItems', 'click') }}>
        🔊 Read Article Aloud
    </button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'cancel', 'click') }}>
        Stop Reading
    </button>
</div>
```
{% endcode %}

### 2. Language Learning App

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <h2>Pronunciation Practice</h2>

    <div class="word-card" {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="fr-FR"
         data-speech-rate="0.8"
         data-speech-text="Bonjour">
        <h3>Bonjour</h3>
        <p>French greeting - "Hello"</p>
        <button {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
            🔊 Slow (0.8x)
        </button>
    </div>

    <div class="word-card" {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="fr-FR"
         data-speech-rate="1.0"
         data-speech-text="Bonjour">
        <button {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
            🔊 Normal (1x)
        </button>
    </div>
</div>

<script>
    document.addEventListener('pwa:speech-synthesis:boundary', (event) => {
        // Highlight word being spoken
        console.log('Word boundary at character:', event.detail.charIndex);
    });
</script>
```
{% endcode %}

### 3. Form Validation Feedback

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: false
}) }}>
    <form id="user-form">
        <label>Email:</label>
        <input type="email" id="email" required>

        <button type="submit">Submit</button>
    </form>
</div>

<script>
    document.getElementById('user-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const email = document.getElementById('email').value;

        if (!email.includes('@')) {
            // Trigger speech for error
            const event = new CustomEvent('speak-error', {
                detail: { text: 'Please enter a valid email address' }
            });

            // Use Stimulus action programmatically
            const controller = document.querySelector('[data-controller="@pwa/speech-synthesis"]');
            controller.dispatchEvent(new CustomEvent('click', {
                detail: { params: { text: 'Please enter a valid email address' } }
            }));
        }
    });
</script>
```
{% endcode %}

### 4. Navigation Assistant

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: false,
    rate: 1.1
}) }} data-speech-controller>
    <div id="map">
        <!-- Map interface -->
    </div>

    <button onclick="announceDirection('Turn left in 500 meters')">
        Simulate Navigation
    </button>
</div>

<script>
    function announceDirection(instruction) {
        const controller = document.querySelector('[data-speech-controller]');
        controller.dispatchEvent(new CustomEvent('speak', {
            detail: {
                params: { text: instruction }
            }
        }));
    }

    document.addEventListener('pwa:speech-synthesis:start', () => {
        // Could lower background music volume here
        console.log('Navigation instruction started');
    });
</script>
```
{% endcode %}

### 5. Interactive Tutorial System

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: true,
    rate: 0.9
}) }}>
    <div class="tutorial">
        <div class="step" {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
             data-speech-text="Step one: Click the menu button in the top right corner">
            <h3>Step 1</h3>
            <p>Click the menu button in the top right corner</p>
            <button {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
                🔊 Listen
            </button>
        </div>

        <div class="step" {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
             data-speech-text="Step two: Select your preferred language from the dropdown">
            <h3>Step 2</h3>
            <p>Select your preferred language from the dropdown</p>
            <button {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
                🔊 Listen
            </button>
        </div>
    </div>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'enqueueItems', 'click') }}>
        🔊 Play Full Tutorial
    </button>
</div>
```
{% endcode %}

## API Reference

### Values

The controller accepts the following configuration values:

**`localeValue`** (String, default: `"en-US"`)
- Default language/locale for speech synthesis
- Examples: `"en-US"`, `"fr-FR"`, `"es-ES"`, `"de-DE"`

**`rateValue`** (Number, default: `1`)
- Speech speed/rate
- Range: 0.1 to 10 (typical: 0.5 to 2)
- `1` = normal speed, `< 1` = slower, `> 1` = faster

**`pitchValue`** (Number, default: `1`)
- Speech pitch
- Range: 0 to 2
- `1` = normal pitch, `< 1` = lower, `> 1` = higher

**`volumeValue`** (Number, default: `1`)
- Speech volume
- Range: 0 to 1
- `0` = silent, `1` = maximum volume

**`voiceValue`** (String, default: `undefined`)
- Specific voice name to use
- If not set, automatically selects best voice for locale
- Example: `"Google US English"`, `"Microsoft David Desktop"`

**`enqueueValue`** (Boolean, default: `true`)
- Queue mode behavior
- `true`: Utterances are queued and played sequentially
- `false`: New utterance cancels previous (immediate mode)

**`i18nValue`** (Object, default: `{}`)
- Internationalization strings for status messages
- Keys: `loading`, `ready`, `unsupported`, `playing`, `paused`, `canceled`, `finished`

**Example:**
{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    locale: 'fr-FR',
    rate: 1.2,
    pitch: 1,
    volume: 0.8,
    voice: 'Google français',
    enqueue: true,
    i18n: {
        ready: 'Prêt',
        playing: 'Lecture'
    }
}) }}>
</div>
```
{% endcode %}

### Targets

**`item`**
- Marks HTML elements as speakable items
- Can be clicked individually or queued together
- Supports custom data attributes for per-item configuration

**`voiceSelect`**
- A `<select>` element that will be automatically populated with available voices
- Automatically formatted as: `"Voice Name (locale) • local"` or `"Voice Name (locale)"`

**`status`**
- Element where status messages are displayed
- Shows: "Loading voices…", "Ready", "Playing", "Paused", etc.
- Content controlled by `i18nValue` or defaults

**Example:**
{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <select {{ stimulus_target('@pwa/speech-synthesis', 'voiceSelect') }}></select>
    <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>Speakable text</p>
    <div {{ stimulus_target('@pwa/speech-synthesis', 'status') }}></div>
</div>
```
{% endcode %}

### Actions

**`speak({ params })`**
- Speaks the provided text
- Parameters:
  - `text` (String, required): Text to speak
  - `locale` (String, optional): Override default locale
  - `voice` (String, optional): Override default voice
  - `rate` (Number, optional): Override default rate
  - `pitch` (Number, optional): Override default pitch
  - `volume` (Number, optional): Override default volume

**`speakItem(event)`**
- Speaks the clicked item target element
- Uses `data-speech-*` attributes or element's `textContent`
- Supports per-item parameters via data attributes

**`enqueueItems({ params })`**
- Queues all item targets for sequential playback
- Parameters can override defaults for all items
- Automatically starts playback

**`pause()`**
- Pauses current speech
- Can be resumed with `resume()`

**`resume()`**
- Resumes paused speech
- No effect if not paused

**`cancel()`**
- Stops current speech and clears queue
- Cannot be resumed

**`changeVoiceFromSelect()`**
- Updates voice from the voiceSelect target value
- Automatically bound to voiceSelect change event

**`setRate({ params })`**
- Updates the speech rate
- Parameters: `rate` (Number)

**`setPitch({ params })`**
- Updates the speech pitch
- Parameters: `pitch` (Number)

**`setVolume({ params })`**
- Updates the speech volume
- Parameters: `volume` (Number)

**Example:**
{% code lineNumbers="true" %}
```twig
<button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click', {
    text: 'Hello world',
    rate: 1.5
}) }}>Speak Fast</button>

<button {{ stimulus_action('@pwa/speech-synthesis', 'pause', 'click') }}>Pause</button>
<button {{ stimulus_action('@pwa/speech-synthesis', 'resume', 'click') }}>Resume</button>
<button {{ stimulus_action('@pwa/speech-synthesis', 'cancel', 'click') }}>Cancel</button>
```
{% endcode %}

### Item Data Attributes

When using `item` targets, you can customize speech parameters per element:

**`data-speech-text`**
- Text to speak (overrides element's `textContent`)

**`data-speech-locale`**
- Language/locale for this item

**`data-speech-voice`**
- Voice name for this item

**`data-speech-rate`**
- Speech rate for this item (Number)

**`data-speech-pitch`**
- Speech pitch for this item (Number)

**`data-speech-volume`**
- Speech volume for this item (Number)

**Example:**
{% code lineNumbers="true" %}
```twig
<p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
   data-speech-text="Custom text to speak"
   data-speech-locale="fr-FR"
   data-speech-rate="1.2"
   data-speech-pitch="1.1"
   data-speech-volume="0.9"
   {{ stimulus_action('@pwa/speech-synthesis', 'speakItem', 'click') }}>
    Click to hear custom speech
</p>
```
{% endcode %}

### Events

All events are prefixed with `pwa:speech-synthesis:` and dispatched on the controller element.

**`pwa:speech-synthesis:ready`**
- Dispatched when the controller is initialized and ready
- No event detail

**`pwa:speech-synthesis:unsupported`**
- Dispatched if Speech Synthesis API is not supported
- No event detail

**`pwa:speech-synthesis:voicesloaded`**
- Dispatched when voices are loaded and available
- Event detail:
  - `voices` (Array): List of available voices with `name`, `lang`, `local` properties

**`pwa:speech-synthesis:start`**
- Dispatched when speech starts
- Event detail:
  - `text` (String): The text being spoken
  - `lang` (String): The language used
  - `sourceEl` (Element|undefined): The source element if speaking an item

**`pwa:speech-synthesis:end`**
- Dispatched when speech finishes
- Event detail:
  - `text` (String): The text that was spoken
  - `lang` (String): The language used
  - `sourceEl` (Element|undefined): The source element if speaking an item

**`pwa:speech-synthesis:pause`**
- Dispatched when speech is paused
- Event detail:
  - `text` (String): The text being paused
  - `sourceEl` (Element|undefined): The source element

**`pwa:speech-synthesis:resume`**
- Dispatched when speech is resumed
- Event detail:
  - `text` (String): The text being resumed
  - `sourceEl` (Element|undefined): The source element

**`pwa:speech-synthesis:error`**
- Dispatched when a speech error occurs
- Event detail:
  - `error` (String): Error message or type
  - `sourceEl` (Element|undefined): The source element

**`pwa:speech-synthesis:boundary`**
- Dispatched at word or sentence boundaries during speech
- Event detail:
  - `name` (String): Boundary type ("word" or "sentence")
  - `charIndex` (Number): Character index in the text
  - `charLength` (Number): Length of the current word/sentence
  - `elapsedTime` (Number): Elapsed time in milliseconds
  - `sourceEl` (Element|undefined): The source element

**`pwa:speech-synthesis:queued`**
- Dispatched when an utterance is added to the queue
- Event detail:
  - `size` (Number): Current queue size

**`pwa:speech-synthesis:dequeue`**
- Dispatched when an utterance is removed from the queue for playback
- Event detail:
  - `remaining` (Number): Number of items remaining in queue

**`pwa:speech-synthesis:cancel`**
- Dispatched when speech is canceled
- No event detail

**`pwa:speech-synthesis:voicechange`**
- Dispatched when the voice is changed
- Event detail:
  - `name` (String): New voice name

**`pwa:speech-synthesis:ratechange`**
- Dispatched when the rate is changed
- Event detail:
  - `rate` (Number): New rate value

**`pwa:speech-synthesis:pitchchange`**
- Dispatched when the pitch is changed
- Event detail:
  - `pitch` (Number): New pitch value

**`pwa:speech-synthesis:volumechange`**
- Dispatched when the volume is changed
- Event detail:
  - `volume` (Number): New volume value

**Example:**
{% code lineNumbers="true" %}
```javascript
const controller = document.querySelector('[data-controller="@pwa/speech-synthesis"]');

controller.addEventListener('pwa:speech-synthesis:start', (event) => {
    console.log('Started speaking:', event.detail.text);
    console.log('Language:', event.detail.lang);
});

controller.addEventListener('pwa:speech-synthesis:boundary', (event) => {
    console.log('Boundary:', event.detail.name, 'at', event.detail.charIndex);
    // Use for text highlighting, karaoke-style reading, etc.
});

controller.addEventListener('pwa:speech-synthesis:voicesloaded', (event) => {
    console.log('Available voices:', event.detail.voices.length);
    event.detail.voices.forEach(v => {
        console.log(`${v.name} (${v.lang}) ${v.local ? '- Local' : ''}`);
    });
});
```
{% endcode %}

### Properties

The controller exposes the following read-only properties:

**`voices`**
- Returns array of available `SpeechSynthesisVoice` objects
- Updated when voices are loaded

**`locales`**
- Returns array of unique locale codes from available voices
- Example: `["en-US", "fr-FR", "es-ES"]`

**`isSpeaking`**
- Returns `true` if currently speaking, `false` otherwise
- Boolean property

**Example:**
{% code lineNumbers="true" %}
```javascript
const controller = application.getControllerForElementAndIdentifier(
    document.querySelector('[data-controller="@pwa/speech-synthesis"]'),
    '@pwa/speech-synthesis'
);

console.log('Available voices:', controller.voices);
console.log('Supported locales:', controller.locales);
console.log('Is speaking:', controller.isSpeaking);
```
{% endcode %}

## Best Practices

### 1. Always Check Browser Support

{% code lineNumbers="true" %}
```javascript
document.addEventListener('pwa:speech-synthesis:unsupported', () => {
    // Show alternative UI or fallback
    document.getElementById('speech-controls').style.display = 'none';
    document.getElementById('text-only-mode').style.display = 'block';
});

document.addEventListener('pwa:speech-synthesis:ready', () => {
    // Enable speech features
    document.getElementById('speech-controls').style.display = 'block';
});
```
{% endcode %}

### 2. Provide Visual Feedback

Always show the current state of speech synthesis:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <div {{ stimulus_target('@pwa/speech-synthesis', 'status') }}
         class="speech-status"></div>
</div>

<style>
.speech-status {
    padding: 10px;
    border-radius: 4px;
    margin: 10px 0;
}
</style>

<script>
    const statusEl = document.querySelector('.speech-status');

    document.addEventListener('pwa:speech-synthesis:start', () => {
        statusEl.style.backgroundColor = '#4caf50';
        statusEl.style.color = 'white';
    });

    document.addEventListener('pwa:speech-synthesis:end', () => {
        statusEl.style.backgroundColor = '#e0e0e0';
        statusEl.style.color = 'black';
    });
</script>
```
{% endcode %}

### 3. Handle Long Content Appropriately

For long articles, break content into manageable chunks:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    enqueue: true
}) }}>
    <!-- Break long content into paragraphs -->
    <article>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>First paragraph...</p>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>Second paragraph...</p>
        <p {{ stimulus_target('@pwa/speech-synthesis', 'item') }}>Third paragraph...</p>
    </article>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'enqueueItems', 'click') }}>
        Read Article
    </button>

    <!-- Allow users to stop long readings -->
    <button {{ stimulus_action('@pwa/speech-synthesis', 'cancel', 'click') }}>
        Stop Reading
    </button>
</div>
```
{% endcode %}

### 4. Use Appropriate Speech Rates

Match speech rate to content type:

- **0.5-0.8**: Learning content, complex information, pronunciation guides
- **0.9-1.0**: Standard reading, news articles, general content
- **1.1-1.5**: Quick scanning, familiar content, user preference
- **1.6-2.0**: Rapid reading for advanced users

### 5. Respect User Preferences

{% code lineNumbers="true" %}
```javascript
// Save user preferences
document.addEventListener('pwa:speech-synthesis:ratechange', (event) => {
    localStorage.setItem('speech-rate', event.detail.rate);
});

document.addEventListener('pwa:speech-synthesis:voicechange', (event) => {
    localStorage.setItem('speech-voice', event.detail.name);
});

// Load preferences on page load
const savedRate = localStorage.getItem('speech-rate');
const savedVoice = localStorage.getItem('speech-voice');

if (savedRate || savedVoice) {
    // Apply saved preferences to controller values
}
```
{% endcode %}

### 6. Optimize for Multi-Language Apps

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <!-- Automatically use correct locale per item -->
    <div {{ stimulus_target('@pwa/speech-synthesis', 'item') }}
         data-speech-locale="{{ app.request.locale }}"
         data-speech-text="{{ 'greeting.text'|trans }}">
        {{ 'greeting.text'|trans }}
    </div>
</div>
```
{% endcode %}

### 7. Use Boundary Events for Highlighting

{% code lineNumbers="true" %}
```javascript
let currentText = '';
let highlightedEl = null;

document.addEventListener('pwa:speech-synthesis:start', (event) => {
    currentText = event.detail.text;
    highlightedEl = event.detail.sourceEl;
});

document.addEventListener('pwa:speech-synthesis:boundary', (event) => {
    if (event.detail.name === 'word' && highlightedEl) {
        // Highlight current word
        const start = event.detail.charIndex;
        const end = start + event.detail.charLength;
        const word = currentText.substring(start, end);

        // Update UI to highlight current word
        console.log('Current word:', word);
    }
});
```
{% endcode %}

### 8. Provide Keyboard Controls

Make speech controls accessible via keyboard:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click') }}
            accesskey="s"
            title="Press Alt+S to speak">
        Speak (Alt+S)
    </button>

    <button {{ stimulus_action('@pwa/speech-synthesis', 'pause', 'click') }}
            accesskey="p"
            title="Press Alt+P to pause">
        Pause (Alt+P)
    </button>
</div>
```
{% endcode %}

## Troubleshooting

### No Voices Available

**Problem:** Voice list is empty or voices don't load

**Solutions:**
1. Wait for `voicesloaded` event before using voices
2. Some browsers load voices asynchronously - the controller handles this automatically
3. On some systems, text-to-speech voices may need to be installed separately

{% code lineNumbers="true" %}
```javascript
document.addEventListener('pwa:speech-synthesis:voicesloaded', (event) => {
    if (event.detail.voices.length === 0) {
        console.warn('No voices available on this system');
        // Show message to user about installing TTS voices
    }
});
```
{% endcode %}

### Speech Not Working on iOS

**Problem:** Speech synthesis doesn't work or requires user interaction on iOS Safari

**Solutions:**
1. iOS requires user interaction before speech can play - ensure speech is triggered by user action (click, tap)
2. Don't auto-play speech on page load
3. Test on actual iOS devices, not just simulators

### Voice Selection Not Working

**Problem:** Selected voice is not being used

**Solutions:**
1. Ensure voice name matches exactly (case-sensitive)
2. Voice must be available in the voices list
3. Some voices are locale-specific - check voice's `lang` property matches utterance locale

{% code lineNumbers="true" %}
```javascript
document.addEventListener('pwa:speech-synthesis:voicesloaded', (event) => {
    console.log('Available voices:');
    event.detail.voices.forEach(v => {
        console.log(`Name: "${v.name}", Lang: ${v.lang}`);
    });
});
```
{% endcode %}

### Speech Cuts Off or Stops

**Problem:** Speech stops unexpectedly or gets interrupted

**Solutions:**
1. Check for multiple speech synthesis controllers on the same page
2. Ensure `enqueue` mode is set correctly
3. Verify no JavaScript errors in console
4. Some browsers limit queue size - break long content into smaller chunks

### Rate/Pitch/Volume Not Applied

**Problem:** Speech parameters don't seem to work

**Solutions:**
1. Some voices ignore certain parameters (especially pitch)
2. Valid ranges: rate (0.1-10), pitch (0-2), volume (0-1)
3. Not all voices support all parameters - try different voices

### Language/Accent Mismatch

**Problem:** Wrong accent or language is used

**Solutions:**
1. Set correct `locale` value (e.g., "en-US" vs "en-GB")
2. Explicitly select a voice with desired accent using `voiceValue`
3. Use `data-speech-locale` per item for multi-language content

{% code lineNumbers="true" %}
```twig
<!-- Explicit locale and voice for best results -->
<div {{ stimulus_controller('@pwa/speech-synthesis', {
    locale: 'en-GB',
    voice: 'Google UK English Female'
}) }}>
```
{% endcode %}

### Memory Issues with Long Content

**Problem:** Browser becomes slow or unresponsive with very long content

**Solutions:**
1. Break content into smaller chunks (use `item` targets)
2. Limit queue size - don't enqueue hundreds of items at once
3. Cancel and clear queue when user navigates away

{% code lineNumbers="true" %}
```javascript
// Clear queue on page unload
window.addEventListener('beforeunload', () => {
    const controller = document.querySelector('[data-controller="@pwa/speech-synthesis"]');
    controller?.dispatchEvent(new CustomEvent('cancel'));
});
```
{% endcode %}

## Accessibility Considerations

### 1. ARIA Attributes

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <button {{ stimulus_action('@pwa/speech-synthesis', 'speak', 'click') }}
            aria-label="Read this content aloud"
            role="button">
        🔊 Listen
    </button>
</div>
```
{% endcode %}

### 2. Screen Reader Compatibility

The component works alongside screen readers. Provide options to disable speech synthesis if user prefers their own assistive technology:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/speech-synthesis') }}>
    <label>
        <input type="checkbox" id="enable-tts" checked>
        Enable text-to-speech
    </label>
</div>

<script>
    document.getElementById('enable-tts').addEventListener('change', (e) => {
        const controller = document.querySelector('[data-controller="@pwa/speech-synthesis"]');
        if (!e.target.checked) {
            controller.dispatchEvent(new CustomEvent('cancel'));
        }
    });
</script>
```
{% endcode %}

### 3. Visual Indicators

Always provide visual feedback that complements audio:

{% code lineNumbers="true" %}
```css
.speaking {
    background-color: #fff9c4;
    border-left: 4px solid #fbc02d;
    padding-left: 10px;
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.7; }
}
```
{% endcode %}

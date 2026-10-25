# Direction and language

The `dir` and `lang` properties define the text direction and primary language of your PWA, ensuring proper display, accessibility, and localization for international audiences.

## Purpose

Language and direction settings enable:
- **Proper text rendering**: Correct text flow for different writing systems
- **Accessibility**: Screen readers use correct pronunciation and voice
- **Search optimization**: Search engines understand content language
- **User experience**: Interface elements adapt to language/direction
- **Localization**: Foundation for multi-language support

## Configuration

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        dir: "ltr"       # Text direction
        lang: "en"       # Primary language (ISO 639-1)
```

!!! info

    **Important**: Manifest `dir` and `lang` should match your HTML `<html>` tag attributes for consistency.

    ```html
    <html dir="ltr" lang="en">
    ```

## The `dir` parameter

The `dir` property specifies how text flows in your application.

### Available values

#### ltr (left-to-right)

Default for most languages. Text flows from left to right.

```yaml
dir: "ltr"
```

**Used for**:
- English, Spanish, French, German
- Most European languages
- Asian languages (Chinese, Japanese, Korean)
- Most world languages

**Examples**: "Hello World" displays as:
```
→ Hello World
```

#### rtl (right-to-left)

Text flows from right to left.

```yaml
dir: "rtl"
```

**Used for**:
- Arabic (العربية)
- Hebrew (עברית)
- Persian/Farsi (فارسی)
- Urdu (اردو)

**Examples**: "مرحبا بالعالم" (Hello World) displays as:
```
مرحبا بالعالم ←
```

#### auto (automatic)

Determines direction based on content. Rarely used in manifests.

```yaml
dir: "auto"
```

**Used for**:
- Mixed-direction content
- User-generated content with multiple languages
- Dynamic content where direction varies

!!! warning

    Using `auto` in manifests is uncommon. Most apps should explicitly set `ltr` or `rtl`.

## The `lang` parameter

The `lang` property declares the primary language using ISO 639-1 language codes.

### Language code format

```yaml
lang: "en"              # Language only
lang: "en-US"           # Language + Region
lang: "en-GB"           # Language + Specific region
lang: "zh-Hans"         # Language + Script
```

### Common language codes

#### European languages

```yaml
lang: "en"     # English
lang: "en-US"  # English (United States)
lang: "en-GB"  # English (United Kingdom)
lang: "fr"     # French
lang: "fr-FR"  # French (France)
lang: "fr-CA"  # French (Canada)
lang: "de"     # German
lang: "es"     # Spanish
lang: "es-ES"  # Spanish (Spain)
lang: "es-MX"  # Spanish (Mexico)
lang: "it"     # Italian
lang: "pt"     # Portuguese
lang: "pt-BR"  # Portuguese (Brazil)
lang: "pt-PT"  # Portuguese (Portugal)
lang: "nl"     # Dutch
lang: "pl"     # Polish
lang: "ru"     # Russian
lang: "sv"     # Swedish
lang: "no"     # Norwegian
lang: "da"     # Danish
lang: "fi"     # Finnish
```

#### Asian languages

```yaml
lang: "zh"        # Chinese
lang: "zh-Hans"   # Chinese (Simplified)
lang: "zh-Hant"   # Chinese (Traditional)
lang: "zh-CN"     # Chinese (China)
lang: "zh-TW"     # Chinese (Taiwan)
lang: "ja"        # Japanese
lang: "ko"        # Korean
lang: "th"        # Thai
lang: "vi"        # Vietnamese
lang: "hi"        # Hindi
lang: "bn"        # Bengali
lang: "id"        # Indonesian
lang: "ms"        # Malay
lang: "tl"        # Tagalog
```

#### Right-to-left languages

```yaml
lang: "ar"     # Arabic
lang: "ar-SA"  # Arabic (Saudi Arabia)
lang: "ar-EG"  # Arabic (Egypt)
lang: "he"     # Hebrew
lang: "fa"     # Persian/Farsi
lang: "ur"     # Urdu
```

#### Other languages

```yaml
lang: "tr"     # Turkish
lang: "el"     # Greek
lang: "cs"     # Czech
lang: "sk"     # Slovak
lang: "hu"     # Hungarian
lang: "ro"     # Romanian
lang: "uk"     # Ukrainian
```

## Practical examples

### English (united states)

```yaml
pwa:
    manifest:
        name: "TaskManager"
        description: "Organize your tasks efficiently."
        dir: "ltr"
        lang: "en-US"
```

### Arabic application

```yaml
pwa:
    manifest:
        name: "مدير المهام"
        description: "نظم مهامك بكفاءة"
        dir: "rtl"
        lang: "ar"
```

### French (france)

```yaml
pwa:
    manifest:
        name: "Gestionnaire de Tâches"
        description: "Organisez vos tâches efficacement."
        dir: "ltr"
        lang: "fr-FR"
```

### Chinese (simplified)

```yaml
pwa:
    manifest:
        name: "任务管理器"
        description: "高效管理您的任务"
        dir: "ltr"
        lang: "zh-Hans"
```

### Hebrew application

```yaml
pwa:
    manifest:
        name: "מנהל משימות"
        description: "ארגן את המשימות שלך ביעילות"
        dir: "rtl"
        lang: "he"
```

### Spanish (mexico)

```yaml
pwa:
    manifest:
        name: "Gestor de Tareas"
        description: "Organiza tus tareas eficientemente."
        dir: "ltr"
        lang: "es-MX"
```

## Multi-language support

### Translatable manifest properties

The PWA Bundle supports translatable manifest properties:

```yaml
pwa:
    manifest:
        name:
            translatable: true
            translations:
                en: "Task Manager"
                fr: "Gestionnaire de Tâches"
                es: "Gestor de Tareas"
                de: "Aufgabenverwaltung"
                ar: "مدير المهام"

        description:
            translatable: true
            translations:
                en: "Organize your tasks efficiently."
                fr: "Organisez vos tâches efficacement."
                es: "Organiza tus tareas eficientemente."
                de: "Organisieren Sie Ihre Aufgaben effizient."
                ar: "نظم مهامك بكفاءة"

        # Direction and language change per locale
        dir: "ltr"  # Default, but can be dynamic
        lang: "en"  # Default, but serves correct locale
```

### Serving different manifests

You can serve different manifests based on user locale:

```php
// In your Symfony controller
#[Route('/manifest.json', name: 'app_manifest')]
public function manifest(Request $request): Response
{
    $locale = $request->getLocale();

    // Adjust dir based on locale
    $dir = in_array($locale, ['ar', 'he', 'fa', 'ur']) ? 'rtl' : 'ltr';

    return $this->json([
        'name' => $translator->trans('app.name'),
        'description' => $translator->trans('app.description'),
        'dir' => $dir,
        'lang' => $locale,
        // ... other properties
    ]);
}
```

## HTML integration

### Matching HTML and manifest

Your HTML should match manifest settings:

```html
<!-- For English -->
<html dir="ltr" lang="en">
    <head>
        <link rel="manifest" href="/manifest.json">
    </head>
</html>

<!-- For Arabic -->
<html dir="rtl" lang="ar">
    <head>
        <link rel="manifest" href="/manifest.json">
    </head>
</html>
```

### Symfony Twig integration

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html dir="{{ app.request.locale in ['ar', 'he', 'fa', 'ur'] ? 'rtl' : 'ltr' }}"
      lang="{{ app.request.locale }}">
<head>
    {{ pwa() }}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>
```

## CSS considerations

### RTL-specific styles

When supporting RTL, adjust your CSS:

```css
/* Use logical properties */
.container {
    padding-inline-start: 20px;  /* Automatically adjusts for RTL */
    margin-inline-end: 10px;
}

/* Or use direction-specific rules */
[dir="ltr"] .container {
    padding-left: 20px;
    margin-right: 10px;
}

[dir="rtl"] .container {
    padding-right: 20px;
    margin-left: 10px;
}

/* Mirror images/icons in RTL */
[dir="rtl"] .arrow-icon {
    transform: scaleX(-1);
}
```

### Logical CSS properties

Prefer logical properties for better RTL support:

```css
/* ✓ Logical - automatically adapts to direction */
margin-inline-start: 10px;
padding-inline-end: 20px;
border-inline-start: 1px solid;

/* ✗ Physical - requires manual RTL adjustments */
margin-left: 10px;
padding-right: 20px;
border-left: 1px solid;
```

## Accessibility impact

### Screen readers

Correct `lang` attribute helps screen readers:
- Use proper pronunciation
- Select correct voice
- Apply language-specific rules

```yaml
# Screen reader will use English voice
lang: "en"

# Screen reader will use French voice
lang: "fr"
```

### Text-to-speech

Browsers use `lang` to determine:
- Voice selection
- Pronunciation rules
- Speech synthesis parameters

## SEO considerations

### Language targeting

Search engines use `lang` for:
- Geo-targeting search results
- Language-specific indexing
- Serving content to appropriate audiences

```yaml
# Targets English speakers
lang: "en"

# Targets French speakers in France
lang: "fr-FR"
```

### `hreflang` relationship

Coordinate with HTML `hreflang` tags:

```html
<link rel="alternate" hreflang="en" href="https://example.com/en" />
<link rel="alternate" hreflang="fr" href="https://example.com/fr" />
<link rel="alternate" hreflang="es" href="https://example.com/es" />
```

## Platform behavior

### iOS/Safari

- Respects `dir` for interface layout
- Uses `lang` for system integration
- May override with user language preferences

### Android/Chrome

- Strong `dir` support
- Automatic UI mirroring for RTL
- System language integration

### Desktop browsers

- Full `dir` and `lang` support
- CSS `:dir()` pseudo-class support
- Proper text rendering

## Common mistakes

### 1. Mismatched direction and language

```yaml
# ✗ Wrong - Arabic is RTL, not LTR
dir: "ltr"
lang: "ar"
```

**Fix**:
```yaml
# ✓ Correct
dir: "rtl"
lang: "ar"
```

### 2. Missing language region

```yaml
# ✗ Ambiguous for Chinese
lang: "zh"  # Which Chinese? Simplified or Traditional?
```

**Fix**:
```yaml
# ✓ Specific
lang: "zh-Hans"  # Simplified Chinese
lang: "zh-Hant"  # Traditional Chinese
```

### 3. Invalid language codes

```yaml
# ✗ Wrong codes
lang: "english"
lang: "EN"
lang: "usa"
```

**Fix**:
```yaml
# ✓ ISO 639-1 codes
lang: "en"
lang: "en-US"
```

### 4. HTML/manifest mismatch

```html
<!-- HTML says LTR English -->
<html dir="ltr" lang="en">
```

```yaml
# Manifest says RTL Arabic (inconsistent!)
dir: "rtl"
lang: "ar"
```

**Fix**: Ensure both match or dynamically generate manifest.

### 5. Forgetting RTL CSS

```yaml
# Set RTL in manifest
dir: "rtl"
lang: "ar"
```

```css
/* But forgot to handle RTL in CSS */
.button {
    margin-left: 10px;  /* Won't work correctly in RTL */
}
```

**Fix**: Use logical properties or RTL-specific rules.

## Testing

### 1. Test in DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "dir" and "lang" values
4. Verify they match HTML tag
```

### 2. Test with screen readers

- NVDA (Windows)
- JAWS (Windows)
- VoiceOver (Mac/iOS)
- TalkBack (Android)

Verify correct language voice is used.

### 3. Visual testing

For RTL languages:
- UI should mirror (buttons, menus on opposite side)
- Text should flow right-to-left
- Scrollbars should appear on left

### 4. Test language detection

```javascript
// Check manifest language
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        console.log('Direction:', manifest.dir);
        console.log('Language:', manifest.lang);

        // Check HTML matches
        console.log('HTML dir:', document.documentElement.dir);
        console.log('HTML lang:', document.documentElement.lang);
    });
```

## Language code reference

### ISO 639-1 standard

Language codes follow ISO 639-1 (two letters):
- https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes

### Region codes

Region codes follow ISO 3166-1 alpha-2:
- https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2

### Complete format

```
lang: "{language}-{Script}-{Region}"

Examples:
- en-US (English, United States)
- zh-Hans-CN (Chinese, Simplified script, China)
- pt-BR (Portuguese, Brazil)
```

## Related properties

- [Translations](../translations.md) - Multi-language manifest support
- [Description](description.md) - Translatable descriptions
- [Name](README.md) - Translatable app names

## Summary

Direction and language best practices:
- ✓ Set `lang` using ISO 639-1 codes
- ✓ Use `ltr` for most languages, `rtl` for Arabic/Hebrew/Persian
- ✓ Match HTML `dir` and `lang` attributes
- ✓ Include region codes when relevant (e.g., `en-US` vs `en-GB`)
- ✓ Support RTL with appropriate CSS
- ✓ Test with screen readers
- ✓ Use translatable manifest properties for multi-language apps
- ✗ Don't use invalid language codes
- ✗ Don't mismatch direction and language
- ✗ Don't forget to test RTL layouts
- ✗ Don't ignore HTML/manifest consistency

# Localisation & Translations

The bundle leverages the Symfony Translation component to provide full internationalization (i18n) support for your Progressive Web App. This allows you to create multilingual PWAs that automatically adapt to the user's language preferences.

## Overview

The PWA Bundle automatically translates manifest content based on the user's locale. When you provide text values in your configuration (names, descriptions, labels, etc.), they are treated as **translation keys** that are resolved using Symfony's translation system.

**Key benefits**:
- Automatic manifest localization per user locale
- Support for all Symfony translation formats (YAML, XLIFF, PHP, JSON)
- Separate manifest files per locale (e.g., `site.en.webmanifest`, `site.fr.webmanifest`)
- Seamless integration with Symfony's translation workflow
- Full Unicode support for all languages
- RTL (Right-to-Left) language support

**Translation domain**: All PWA translations use the `pwa` domain by default.

## How Translation Works

When you configure your PWA manifest:

1. **Text values become translation keys**: Values like `"app.name"` are translation keys, not literal text
2. **Bundle checks for translations**: The bundle looks for translations in the `pwa` domain
3. **Locale-specific manifests generated**: A separate manifest is created for each enabled locale
4. **Automatic locale detection**: The `pwa()` Twig function automatically serves the correct manifest based on `app.request.locale`

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "app.name"              # Translation key, not literal text!
        short_name: "app.short_name"  # Translation key
        description: "app.description" # Translation key
```
{% endcode %}

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "My Awesome App"
    short_name: "AwesomeApp"
    description: "The best productivity app"
```
{% endcode %}

{% code title="/translations/pwa.fr.yaml" lineNumbers="true" %}
```yaml
app:
    name: "Mon Application Géniale"
    short_name: "AppGéniale"
    description: "La meilleure application de productivité"
```
{% endcode %}

## Translatable Properties

The following properties are automatically translated by the bundle:

| Component | Translatable Properties |
|-----------|------------------------|
| **Manifest** | `name`, `short_name`, `description` |
| **Screenshot** | `label` |
| **Share Target** | `title`, `text` |
| **Shortcut** | `name`, `short_name`, `description` |
| **Widget** | `name`, `short_name`, `description` |

{% hint style="warning" %}
**Not Translatable**: The `categories` property should **not** be translated. This capability is deprecated and will be removed in version 2.0.0.
{% endhint %}

{% hint style="danger" %}
**Breaking Change**: The `start_url` property cannot be translated since version 1.3.0. See the [migration guide](../upgrades/from-1.2.x-to-1.3.0.md#start-url) for details.
{% endhint %}

## Basic Configuration

### Step 1: Enable Locales in Symfony

First, configure the locales your app supports:

{% code title="/config/packages/translation.yaml" lineNumbers="true" %}
```yaml
framework:
    enabled_locales: ['en', 'fr', 'de', 'es']
    default_locale: 'en'
    translator:
        default_path: '%kernel.project_dir%/translations'
        fallbacks: ['en']
```
{% endcode %}

See the [Symfony Translation documentation](https://symfony.com/doc/current/translation.html) for more information about configuring locales.

### Step 2: Configure Locale Placeholder

Add the `{locale}` placeholder to your manifest public URL:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/site.{locale}.webmanifest"
        name: "app.name"
        short_name: "app.short_name"
        description: "app.description"
```
{% endcode %}

{% hint style="info" %}
**Important**: When using the `{locale}` placeholder, do **not** manually set the `lang` parameter. The bundle automatically sets it based on the current locale.
{% endhint %}

### Step 3: Use the pwa() Twig Function

The `pwa()` Twig function automatically serves the correct localized manifest:

{% code title="/templates/base.html.twig" lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="{{ app.request.locale }}">
<head>
    {{ pwa() }}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>
```
{% endcode %}

{% hint style="success" %}
**Automatic Detection**: Since version 1.3.0, you don't need to pass the locale to `pwa()`. It's automatically detected from `app.request.locale`.
{% endhint %}

### Step 4: Create Translation Files

Create translation files for each locale in the `pwa` domain:

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "TaskMaster Pro"
    short_name: "TaskMaster"
    description: "Professional task management for teams"
```
{% endcode %}

{% code title="/translations/pwa.fr.yaml" lineNumbers="true" %}
```yaml
app:
    name: "TaskMaster Pro"
    short_name: "TaskMaster"
    description: "Gestion professionnelle des tâches pour équipes"
```
{% endcode %}

{% code title="/translations/pwa.de.yaml" lineNumbers="true" %}
```yaml
app:
    name: "TaskMaster Pro"
    short_name: "TaskMaster"
    description: "Professionelles Aufgabenmanagement für Teams"
```
{% endcode %}

## Translation File Formats

The PWA Bundle supports all Symfony translation formats.

### YAML Format (Recommended)

YAML is human-readable and easy to edit:

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "My App"
    short_name: "MyApp"
    description: "A great application"

shortcuts:
    dashboard:
        name: "Dashboard"
        description: "View your dashboard"
    settings:
        name: "Settings"
        description: "Manage settings"

screenshots:
    home: "Home screen with features"
    dashboard: "Analytics dashboard"
```
{% endcode %}

### XLIFF Format

XLIFF is ideal for professional translation workflows:

{% code title="/translations/pwa+intl-icu.en.xlf" lineNumbers="true" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<xliff xmlns="urn:oasis:names:tc:xliff:document:1.2" version="1.2">
  <file source-language="en" target-language="en" datatype="plaintext" original="file.ext">
    <header>
      <tool tool-id="symfony" tool-name="Symfony"/>
    </header>
    <body>
      <trans-unit id="app.name" resname="app.name">
        <source>app.name</source>
        <target>TaskMaster Pro</target>
      </trans-unit>
      <trans-unit id="app.short_name" resname="app.short_name">
        <source>app.short_name</source>
        <target>TaskMaster</target>
      </trans-unit>
      <trans-unit id="app.description" resname="app.description">
        <source>app.description</source>
        <target>Professional task management</target>
      </trans-unit>
    </body>
  </file>
</xliff>
```
{% endcode %}

### PHP Format

PHP format allows for dynamic translations:

{% code title="/translations/pwa.en.php" lineNumbers="true" %}
```php
<?php

return [
    'app' => [
        'name' => 'TaskMaster Pro',
        'short_name' => 'TaskMaster',
        'description' => 'Professional task management',
    ],
    'shortcuts' => [
        'dashboard' => [
            'name' => 'Dashboard',
            'description' => 'View your dashboard',
        ],
    ],
];
```
{% endcode %}

### JSON Format

JSON is compatible with JavaScript tooling:

{% code title="/translations/pwa.en.json" lineNumbers="true" %}
```json
{
    "app.name": "TaskMaster Pro",
    "app.short_name": "TaskMaster",
    "app.description": "Professional task management",
    "shortcuts.dashboard.name": "Dashboard",
    "shortcuts.dashboard.description": "View your dashboard"
}
```
{% endcode %}

## Complete Use Cases

### Use Case 1: Multilingual E-Commerce App

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/manifest.{locale}.webmanifest"
        name: "shop.name"
        short_name: "shop.short_name"
        description: "shop.description"

        screenshots:
            - src: "screenshots/home-en.png"
              label: "shop.screenshots.home"
            - src: "screenshots/products-en.png"
              label: "shop.screenshots.products"
            - src: "screenshots/cart-en.png"
              label: "shop.screenshots.cart"

        shortcuts:
            - name: "shop.shortcuts.cart.name"
              short_name: "shop.shortcuts.cart.short"
              description: "shop.shortcuts.cart.description"
              url: "/cart"
              icons:
                - src: "icons/cart.png"
                  sizes: [96]

            - name: "shop.shortcuts.orders.name"
              short_name: "shop.shortcuts.orders.short"
              description: "shop.shortcuts.orders.description"
              url: "/orders"
              icons:
                - src: "icons/orders.png"
                  sizes: [96]
```
{% endcode %}

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
shop:
    name: "ShopMaster"
    short_name: "Shop"
    description: "Your favorite online shopping destination"

    screenshots:
        home: "Browse thousands of products"
        products: "Detailed product information"
        cart: "Easy checkout process"

    shortcuts:
        cart:
            name: "Shopping Cart"
            short: "Cart"
            description: "View your cart"
        orders:
            name: "My Orders"
            short: "Orders"
            description: "Track your orders"
```
{% endcode %}

{% code title="/translations/pwa.fr.yaml" lineNumbers="true" %}
```yaml
shop:
    name: "ShopMaster"
    short_name: "Shop"
    description: "Votre destination shopping en ligne préférée"

    screenshots:
        home: "Parcourez des milliers de produits"
        products: "Informations détaillées sur les produits"
        cart: "Processus de paiement facile"

    shortcuts:
        cart:
            name: "Panier"
            short: "Panier"
            description: "Voir votre panier"
        orders:
            name: "Mes commandes"
            short: "Commandes"
            description: "Suivez vos commandes"
```
{% endcode %}

{% code title="/translations/pwa.de.yaml" lineNumbers="true" %}
```yaml
shop:
    name: "ShopMaster"
    short_name: "Shop"
    description: "Ihr bevorzugtes Online-Shopping-Ziel"

    screenshots:
        home: "Durchsuchen Sie Tausende von Produkten"
        products: "Detaillierte Produktinformationen"
        cart: "Einfacher Checkout-Prozess"

    shortcuts:
        cart:
            name: "Warenkorb"
            short: "Korb"
            description: "Warenkorb anzeigen"
        orders:
            name: "Meine Bestellungen"
            short: "Bestellungen"
            description: "Bestellungen verfolgen"
```
{% endcode %}

### Use Case 2: Global News App with RTL Support

{% code title="/config/packages/translation.yaml" lineNumbers="true" %}
```yaml
framework:
    enabled_locales: ['en', 'ar', 'he', 'fr', 'es']
    default_locale: 'en'
```
{% endcode %}

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/news.{locale}.webmanifest"
        name: "news.app.name"
        short_name: "news.app.short"
        description: "news.app.description"

        shortcuts:
            - name: "news.shortcuts.breaking.name"
              url: "/breaking"
              icons:
                - src: "icons/breaking.png"
                  sizes: [96]
```
{% endcode %}

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
news:
    app:
        name: "Global News"
        short: "News"
        description: "Breaking news from around the world"
    shortcuts:
        breaking:
            name: "Breaking News"
```
{% endcode %}

{% code title="/translations/pwa.ar.yaml" lineNumbers="true" %}
```yaml
news:
    app:
        name: "الأخبار العالمية"
        short: "أخبار"
        description: "آخر الأخبار من جميع أنحاء العالم"
    shortcuts:
        breaking:
            name: "الأخبار العاجلة"
```
{% endcode %}

{% code title="/translations/pwa.he.yaml" lineNumbers="true" %}
```yaml
news:
    app:
        name: "חדשות עולמיות"
        short: "חדשות"
        description: "חדשות מהירות מכל העולם"
    shortcuts:
        breaking:
            name: "חדשות מהירות"
```
{% endcode %}

{% hint style="success" %}
**RTL Support**: The bundle automatically sets the `dir` property based on the locale. For RTL languages like Arabic (`ar`) and Hebrew (`he`), the direction is set to `"rtl"` automatically.
{% endhint %}

### Use Case 3: SaaS Platform with Region-Specific Content

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/app.{locale}.webmanifest"
        name: "saas.name"
        short_name: "saas.short"
        description: "saas.description"

        screenshots:
            - src: "screenshots/dashboard-{locale}.png"
              label: "saas.screenshots.dashboard"
            - src: "screenshots/analytics-{locale}.png"
              label: "saas.screenshots.analytics"
```
{% endcode %}

Note: The `{locale}` placeholder in screenshot paths allows for locale-specific screenshots.

## Generated Manifest Files

When you configure translations with the `{locale}` placeholder, the bundle generates separate manifest files for each enabled locale.

**Example with locales `['en', 'fr', 'de']`**:

```
public/
├── site.en.webmanifest
├── site.fr.webmanifest
└── site.de.webmanifest
```

**site.en.webmanifest**:
```json
{
  "name": "TaskMaster Pro",
  "short_name": "TaskMaster",
  "description": "Professional task management",
  "lang": "en",
  "dir": "ltr",
  ...
}
```

**site.fr.webmanifest**:
```json
{
  "name": "TaskMaster Pro",
  "short_name": "TaskMaster",
  "description": "Gestion professionnelle des tâches",
  "lang": "fr",
  "dir": "ltr",
  ...
}
```

**site.de.webmanifest**:
```json
{
  "name": "TaskMaster Pro",
  "short_name": "TaskMaster",
  "description": "Professionelles Aufgabenmanagement",
  "lang": "de",
  "dir": "ltr",
  ...
}
```

## Translation Workflow

### 1. Extract Translation Keys

Use Symfony's translation extraction command to find untranslated keys:

```bash
# Extract all translation keys to YAML files
php bin/console translation:extract --force en --domain=pwa

# Extract to XLIFF format
php bin/console translation:extract --force --format=xlf en --domain=pwa

# Extract for multiple locales
php bin/console translation:extract --force en --domain=pwa
php bin/console translation:extract --force fr --domain=pwa
php bin/console translation:extract --force de --domain=pwa
```

### 2. Translate Content

**Option A: Manual Translation**
- Edit YAML/XLIFF files directly
- Use your preferred text editor

**Option B: Professional Translation**
- Export XLIFF files
- Send to translation service
- Import translated XLIFF files

**Option C: Translation Management Platform**
- Use platforms like Crowdin, Lokalise, or Phrase
- Import/export via API
- Collaborative translation workflow

### 3. Validate Translations

Check for missing translations:

```bash
# List missing translations
php bin/console translation:extract --dump-messages en --domain=pwa

# Check all locales
php bin/console debug:translation --domain=pwa
```

### 4. Clear Cache

After updating translations, clear the cache:

```bash
# Clear cache
php bin/console cache:clear

# Warm up cache
php bin/console cache:warmup
```

## Testing Translations

### 1. Test Manifest Generation

Verify that manifests are generated for each locale:

```bash
# Check for generated manifests
ls -la public/*.webmanifest

# Expected:
# site.en.webmanifest
# site.fr.webmanifest
# site.de.webmanifest
```

### 2. Validate Manifest Content

Check that translations are applied correctly:

```bash
# View English manifest
cat public/site.en.webmanifest | jq '.name, .description'

# View French manifest
cat public/site.fr.webmanifest | jq '.name, .description'

# Compare manifests
diff <(jq -S . public/site.en.webmanifest) <(jq -S . public/site.fr.webmanifest)
```

### 3. Test in Browser

**Manual Testing**:
```bash
1. Change browser language to target locale
2. Clear browser cache and service worker
3. Visit your app
4. Check DevTools → Application → Manifest
5. Verify translated content appears
```

**Automated Testing with Puppeteer**:

{% code title="tests/pwa-i18n.test.js" lineNumbers="true" %}
```javascript
const puppeteer = require('puppeteer');

async function testManifestTranslations() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    // Test English
    await page.setExtraHTTPHeaders({
        'Accept-Language': 'en-US,en;q=0.9'
    });
    await page.goto('https://your-app.com/');

    const manifestUrlEn = await page.evaluate(() => {
        const link = document.querySelector('link[rel="manifest"]');
        return link ? link.href : null;
    });

    console.log('English manifest URL:', manifestUrlEn);
    // Expected: https://your-app.com/site.en.webmanifest

    // Test French
    await page.setExtraHTTPHeaders({
        'Accept-Language': 'fr-FR,fr;q=0.9'
    });
    await page.goto('https://your-app.com/');

    const manifestUrlFr = await page.evaluate(() => {
        const link = document.querySelector('link[rel="manifest"]');
        return link ? link.href : null;
    });

    console.log('French manifest URL:', manifestUrlFr);
    // Expected: https://your-app.com/site.fr.webmanifest

    await browser.close();
}

testManifestTranslations();
```
{% endcode %}

### 4. Lighthouse Audit

Run Lighthouse with different locales:

```bash
# Audit with English locale
LANG=en npx lighthouse https://your-app.com --view

# Audit with French locale
LANG=fr npx lighthouse https://your-app.com --view
```

## Best Practices

### 1. Use Meaningful Translation Keys

```yaml
# ✅ Good - Clear, hierarchical keys
app:
    name: "TaskMaster Pro"

shortcuts:
    dashboard:
        name: "Dashboard"
        description: "View your dashboard"

# ❌ Avoid - Unclear, flat keys
name: "TaskMaster Pro"
shortcut1: "Dashboard"
desc1: "View your dashboard"
```

### 2. Keep Brand Names Consistent

```yaml
# ✅ Good - Preserve brand name across locales
# pwa.en.yaml
app:
    name: "TaskMaster Pro"  # Brand name unchanged
    description: "Professional task management"

# pwa.fr.yaml
app:
    name: "TaskMaster Pro"  # Same brand name
    description: "Gestion professionnelle des tâches"  # Translated description
```

### 3. Provide Fallbacks

{% code title="/config/packages/translation.yaml" lineNumbers="true" %}
```yaml
framework:
    translator:
        fallbacks: ['en']  # Fallback to English if translation missing
```
{% endcode %}

### 4. Use Translation Parameters for Dynamic Content

While the PWA manifest doesn't support dynamic parameters, you can use them in your translation files:

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "MyApp"
    description: "Available in 20 countries"  # Static for manifest

# For dynamic content in your app (not manifest):
messages:
    welcome: "Welcome, {username}!"  # Parameters work in regular translations
```
{% endcode %}

### 5. Keep Translations Synchronized

Use a consistent structure across all locales:

```yaml
# All locales should have the same structure

# ✅ Good - Same keys in all files
# pwa.en.yaml
app:
    name: "..."
    short_name: "..."
    description: "..."

# pwa.fr.yaml
app:
    name: "..."
    short_name: "..."
    description: "..."

# ❌ Avoid - Missing keys in some locales
# pwa.fr.yaml (missing short_name)
app:
    name: "..."
    description: "..."
```

### 6. Test All Locales

Before deploying, verify all locale-specific manifests:

```bash
# Check all generated manifests exist
for locale in en fr de es; do
    if [ -f "public/site.${locale}.webmanifest" ]; then
        echo "✓ $locale manifest exists"
    else
        echo "✗ $locale manifest missing!"
    fi
done
```

## Common Issues

### 1. Translations Not Applied

**Problem**: Manifest shows translation keys instead of translated text

**Solutions**:

```bash
# ✅ Check 1: Verify translation files exist
ls -la translations/pwa.*

# ✅ Check 2: Ensure correct domain
# Translation files must use 'pwa' domain: pwa.en.yaml, pwa.fr.yaml, etc.

# ✅ Check 3: Clear cache
php bin/console cache:clear

# ✅ Check 4: Check enabled locales
php bin/console debug:config framework enabled_locales
```

### 2. Wrong Locale Manifest Served

**Problem**: English manifest served when French is expected

**Solutions**:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
# ✅ Ensure {locale} placeholder is present
pwa:
    manifest:
        public_url: "/site.{locale}.webmanifest"  # Must include {locale}
```
{% endcode %}

```bash
# ✅ Check request locale
# In your Twig template:
{{ dump(app.request.locale) }}

# ✅ Verify locale detection
# Check your LocaleSubscriber or locale configuration
```

### 3. Missing Translations for Specific Locale

**Problem**: Some keys not translated in certain locales

**Solutions**:

```bash
# ✅ Check for missing translations
php bin/console debug:translation --domain=pwa fr

# ✅ Extract missing keys
php bin/console translation:extract --force fr --domain=pwa

# ✅ Compare locales
diff translations/pwa.en.yaml translations/pwa.fr.yaml
```

### 4. Special Characters Not Displaying

**Problem**: Accented characters or non-Latin scripts display incorrectly

**Solutions**:

{% code title="/translations/pwa.fr.yaml" lineNumbers="true" %}
```yaml
# ✅ Ensure UTF-8 encoding
# Save file as UTF-8 without BOM

app:
    name: "Application Géniale"  # é should display correctly
    description: "Optimisée pour vous"  # é should display correctly
```
{% endcode %}

```bash
# ✅ Check file encoding
file -i translations/pwa.fr.yaml
# Expected: charset=utf-8
```

### 5. RTL Languages Not Working

**Problem**: Arabic/Hebrew text displays left-to-right

**Solutions**:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
# ✅ Don't manually set 'dir' or 'lang'
pwa:
    manifest:
        # dir: "ltr"   ← Remove this! Let bundle auto-detect
        # lang: "en"   ← Remove this! Let bundle auto-detect
        public_url: "/site.{locale}.webmanifest"
```
{% endcode %}

The bundle automatically sets `dir: "rtl"` for RTL locales (ar, he, fa, ur).

## Migration Guide

### Migrating from Non-Translated Setup

**Before (Hard-coded text)**:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/manifest.json"
        name: "My App"  # Hard-coded
        short_name: "MyApp"  # Hard-coded
        description: "A great app"  # Hard-coded
```
{% endcode %}

**After (Translation keys)**:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/site.{locale}.webmanifest"  # Add {locale} placeholder
        name: "app.name"  # Translation key
        short_name: "app.short_name"  # Translation key
        description: "app.description"  # Translation key
```
{% endcode %}

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "My App"
    short_name: "MyApp"
    description: "A great app"
```
{% endcode %}

{% code title="/translations/pwa.fr.yaml" lineNumbers="true" %}
```yaml
app:
    name: "Mon Application"
    short_name: "MonApp"
    description: "Une excellente application"
```
{% endcode %}

### Migration Steps

1. **Add `{locale}` placeholder** to `public_url`
2. **Remove manual `lang` setting** (if present)
3. **Replace hard-coded text with translation keys** in config
4. **Create translation files** for each locale
5. **Configure enabled locales** in `framework.yaml`
6. **Clear cache** and test

```bash
# After migration, clear cache
php bin/console cache:clear

# Extract translation keys
php bin/console translation:extract --force en --domain=pwa

# Test manifest generation
php bin/console cache:warmup
ls -la public/*.webmanifest
```

## Troubleshooting Checklist

Before deploying multilingual PWA:

- [ ] `{locale}` placeholder added to `public_url`
- [ ] Enabled locales configured in `framework.yaml`
- [ ] Translation files created for all locales (pwa.*.yaml)
- [ ] All translation keys defined in all locale files
- [ ] Cache cleared after translation changes
- [ ] Manifest files generated for each locale
- [ ] Correct locale served based on `app.request.locale`
- [ ] Translations appear correctly in browser
- [ ] RTL languages display correctly (if applicable)
- [ ] No translation keys visible in manifest
- [ ] Tested installation flow in each language
- [ ] Lighthouse audit passes for all locales

## Related Documentation

- [Direction and Language](application-information/direction-and-language.md) - Language and RTL configuration
- [Screenshots](screenshots.md) - Translating screenshot labels
- [Shortcuts](shortcuts.md) - Translating shortcut names
- [Symfony Translation](https://symfony.com/doc/current/translation.html) - Official Symfony Translation docs

## Resources

- **Symfony Translation**: https://symfony.com/doc/current/translation.html
- **Translation Component**: https://symfony.com/doc/current/components/translation.html
- **Enabled Locales**: https://symfony.com/doc/current/reference/configuration/framework.html#enabled-locales
- **ICU Message Format**: https://unicode-org.github.io/icu/userguide/format_parse/messages/
- **XLIFF Format**: https://docs.oasis-open.org/xliff/xliff-core/v2.0/xliff-core-v2.0.html
- **Crowdin**: https://crowdin.com/ - Translation management platform
- **Lokalise**: https://lokalise.com/ - Localization platform
- **Phrase**: https://phrase.com/ - Translation platform

# Description

The `description` member provides a human-readable explanation of what your web application does. This text appears in app stores, installation prompts, and is used by assistive technologies for accessibility.

## Purpose

The description serves multiple purposes:

- **Installation prompts**: Shown when users install your PWA
- **App stores**: Displayed in app listings and search results
- **Accessibility**: Read by screen readers to describe the application
- **SEO**: Helps search engines understand your application's purpose

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        description: "Track your fitness goals, log workouts, and monitor progress with detailed analytics and personalized recommendations."
```
{% endcode %}

## Best Practices

### Write Clear and Concise Descriptions

**Good examples**:
```yaml
description: "Manage your tasks efficiently with reminders, priorities, and team collaboration features."
description: "Browse and purchase eco-friendly products from local sustainable brands."
description: "Learn languages through interactive lessons, games, and conversation practice."
```

**Poor examples**:
```yaml
description: "App"  # Too vague
description: "The best, most amazing, revolutionary application you've ever used!"  # Too promotional
description: "This is an application that helps users to manage their daily tasks and organize their work efficiently while providing advanced features for productivity enhancement and team collaboration capabilities."  # Too long
```

### Length Guidelines

- **Minimum**: 20-30 characters (meaningful description)
- **Recommended**: 80-200 characters (sweet spot for most contexts)
- **Maximum**: 300 characters (avoid longer descriptions)

**Why these limits?**:
- Short descriptions may not provide enough context
- Very long descriptions get truncated in UI
- 80-200 characters displays well across all platforms

### Focus on Value

Describe **what** the app does and **why** users should care:

```yaml
# ✓ Good - explains value
description: "Find and book local services instantly, with secure payments and verified reviews."

# ✗ Poor - generic
description: "A platform for services."
```

### Use Action-Oriented Language

Start with verbs that describe user actions:

```yaml
description: "Track expenses, create budgets, and achieve your financial goals."
description: "Browse recipes, save favorites, and generate shopping lists automatically."
description: "Monitor your home security, control devices, and receive instant alerts."
```

### Avoid Technical Jargon

Use language your target audience understands:

```yaml
# ✓ Good - accessible
description: "Share photos with friends and family in private groups with easy organization."

# ✗ Poor - jargon-heavy
description: "Leverage our distributed cloud-based photo storage infrastructure with advanced metadata indexing."
```

## Practical Examples

### E-commerce Application

```yaml
pwa:
    manifest:
        description: "Shop for handcrafted items from independent artisans with fast shipping and buyer protection."
```

### Productivity Tool

```yaml
pwa:
    manifest:
        description: "Organize projects, assign tasks, track deadlines, and collaborate with your team in real-time."
```

### Health & Fitness

```yaml
pwa:
    manifest:
        description: "Log meals, count calories, track nutrients, and reach your health goals with personalized meal plans."
```

### News/Content

```yaml
pwa:
    manifest:
        description: "Stay informed with breaking news, in-depth articles, and personalized content from trusted sources."
```

### Social Platform

```yaml
pwa:
    manifest:
        description: "Connect with people who share your interests, join communities, and discover engaging content daily."
```

## Multilingual Descriptions

For applications supporting multiple languages, use translatable objects:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        description:
            translatable: true
            translations:
                en: "Manage your finances with budgeting tools and expense tracking."
                fr: "Gérez vos finances avec des outils de budgétisation et de suivi des dépenses."
                es: "Administra tus finanzas con herramientas de presupuesto y seguimiento de gastos."
                de: "Verwalten Sie Ihre Finanzen mit Budgetierungs- und Ausgabenverfolgungstools."
```
{% endcode %}

See [Direction and Language](direction-and-language.md) for more details on translations.

## Where Descriptions Appear

### Installation Dialog

When users add your PWA to their home screen:

```
Add "My App" to Home Screen?

[Your description appears here]

[Cancel] [Add]
```

### App Info Screen

After installation, in the app details view:

```
My App
[Icon]
Description: [Your description]
Version: 1.0
Size: 2.5 MB
```

### Search Results

When users search for PWAs in browsers or app stores.

## Testing Your Description

### 1. Check Length

Count characters including spaces:

```javascript
const description = "Track expenses, create budgets, and achieve goals.";
console.log(description.length); // Should be 80-200
```

### 2. Read Aloud

Your description should sound natural when spoken:
- Use a screen reader
- Read it to someone unfamiliar with your app
- Ask if they understand what the app does

### 3. Test on Different Devices

View your PWA's install prompt on:
- Mobile devices (small screens truncate text)
- Tablets (medium-length descriptions)
- Desktops (full description visible)

### 4. Validate Accessibility

Ensure your description is meaningful for screen reader users:
- Avoid starting with "This app..." (redundant)
- Don't use special characters unnecessarily
- Be specific about functionality

## Common Mistakes

### 1. Missing Description

```yaml
pwa:
    manifest:
        # description not set - uses defaults or appears blank
```

**Impact**: Users may not understand what your app does

### 2. Using Company Name Only

```yaml
description: "Acme Corporation"  # ✗ Doesn't describe functionality
```

**Instead**:
```yaml
description: "Acme Corporation's task management platform for remote teams."  # ✓ Descriptive
```

### 3. Duplicating the App Name

```yaml
name: "TaskMaster Pro"
description: "TaskMaster Pro - The Task Management Application"  # ✗ Redundant
```

**Instead**:
```yaml
name: "TaskMaster Pro"
description: "Organize tasks, set priorities, and collaborate with your team."  # ✓ Adds value
```

### 4. Marketing Speak

```yaml
description: "Revolutionary, game-changing, next-generation productivity suite!"  # ✗ Hyperbole
```

**Instead**:
```yaml
description: "Manage tasks, track time, and collaborate with team members."  # ✓ Factual
```

## SEO Considerations

While the manifest description isn't directly used for web search ranking, it can impact:

- **App discovery**: In PWA directories and app stores
- **User intent matching**: Helps users find relevant apps
- **Click-through rates**: Compelling descriptions increase installations

## Related Properties

- [Name and Short Name](README.md#name-and-short-name) - How your app is titled
- [Direction and Language](direction-and-language.md) - Localization
- [Categories](categories.md) - App classification for discovery

## Summary

A well-crafted description:
- ✓ Clearly explains what the app does (80-200 characters)
- ✓ Uses action-oriented, accessible language
- ✓ Focuses on user benefits
- ✓ Avoids marketing jargon and hyperbole
- ✓ Reads naturally when spoken aloud
- ✓ Provides value beyond the app name

# Categories

The `categories` property helps app stores, browsers, and platforms classify your PWA, making it easier for users to discover your application. It's a list of keywords that describe what your application does.

## Purpose

Categories help with:
- **App discovery**: Users browsing by category find your app
- **Search relevance**: Improves searchability in app stores
- **Store organization**: Platforms group similar apps together
- **User expectations**: Sets correct expectations about functionality
- **Recommendations**: Powers "similar apps" suggestions

## Configuration

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        categories: ['productivity', 'business']
```

!!! info

    **Best practices**:
    - Use lowercase for consistency
    - Choose 1-3 most relevant categories
    - Start with the primary category
    - Categories are hints - platforms may use their own classification

## Standard categories

### Business & productivity

**business**
- CRM systems
- Project management
- Business analytics
- Invoice management

**productivity**
- Task managers
- Note-taking apps
- Time tracking
- Calendar applications

**finance**
- Banking apps
- Expense trackers
- Investment platforms
- Cryptocurrency wallets

**developer** / **developer tools**
- Code editors
- API clients
- DevOps dashboards
- Documentation browsers

### Media & entertainment

**music**
- Streaming services
- Music players
- Podcast apps
- Audio editors

**video**
- Video streaming
- Video editors
- YouTube clients
- Video conferencing

**photo** / **photo & video**
- Photo editors
- Image galleries
- Camera apps
- Photo sharing platforms

**games**
- All types of games
- Game launchers
- Gaming platforms

**entertainment**
- Streaming services
- Media centers
- Fun utilities
- Novelty apps

### Social & communication

**social** / **social networking**
- Social media platforms
- Community forums
- Messaging apps
- Social discovery

**dating**
- Dating apps
- Matchmaking platforms
- Relationship services

**network** / **networking**
- Professional networking
- Business connections
- Conference apps

### Lifestyle & health

**health** / **health & fitness**
- Fitness trackers
- Workout apps
- Health monitoring
- Wellness platforms

**fitness**
- Gym workout apps
- Running trackers
- Yoga and meditation
- Exercise planners

**food**
- Recipe apps
- Meal planners
- Nutrition trackers
- Restaurant finders

**lifestyle**
- General lifestyle apps
- Habit trackers
- Personal development
- Life organization

**medical**
- Medical records
- Symptom checkers
- Telemedicine
- Medical reference

### Content & learning

**books** / **books & reference**
- E-book readers
- Library apps
- Literary platforms
- Reading trackers

**education**
- Learning platforms
- Online courses
- Educational games
- Study tools

**news**
- News aggregators
- Newspapers
- RSS readers
- Breaking news alerts

**magazines**
- Magazine readers
- Periodical apps
- Article collections

**reference**
- Dictionaries
- Encyclopedias
- Lookup tools
- Information databases

### Specialized

**travel**
- Trip planners
- Hotel booking
- Flight tracking
- Travel guides

**navigation**
- Maps
- GPS navigation
- Transit apps
- Location services

**shopping**
- E-commerce
- Price comparison
- Shopping lists
- Marketplace apps

**sports**
- Sports news
- Live scores
- Fantasy sports
- Team management

**weather**
- Weather forecasts
- Weather alerts
- Climate data
- Weather radar

**utilities**
- General utilities
- Calculators
- Converters
- System tools

### Creative & design

**design**
- Design tools
- Mockup creators
- UI/UX tools

**graphics** / **graphics & design**
- Graphic design
- Vector editors
- Image manipulation

**multimedia** / **multimedia design**
- Multi-media creation
- Audio/video editing
- Content production

### Specialized audiences

**kids**
- Child-appropriate content
- Educational games for children
- Parental controls included

**parenting**
- Parenting tips
- Baby trackers
- Family organization
- Child development tools

**pets**
- Pet care
- Vet finders
- Pet health trackers
- Animal adoption

### Other

**beauty**
- Makeup tutorials
- Beauty tips
- Product reviews
- Style guides

**cars** / **transportation**
- Car maintenance
- Vehicle tracking
- Ride sharing
- Public transit

**fashion**
- Style guides
- Fashion shopping
- Outfit planning
- Trend tracking

**events**
- Event planning
- Ticket booking
- Event discovery
- Calendar integration

**fundraising**
- Donation platforms
- Crowdfunding
- Charity apps

**government**
- Government services
- Civic engagement
- Public records
- Official portals

**personalization**
- Customization tools
- Themes
- Wallpapers
- Launchers

**politics**
- Political news
- Campaign information
- Voting resources
- Political engagement

**security**
- Password managers
- VPN clients
- Security tools
- Privacy apps

## Choosing the right categories

### Single primary category

For highly focused apps:

```yaml
categories: ['fitness']  # Workout tracking app
categories: ['music']    # Music streaming service
categories: ['games']    # Gaming platform
```

### Multiple related categories

For apps spanning related areas:

```yaml
# Health and fitness tracker
categories: ['health & fitness', 'lifestyle']

# Business collaboration tool
categories: ['productivity', 'business']

# Recipe and meal planner
categories: ['food', 'lifestyle']

# Photo editor and social sharing
categories: ['photo & video', 'social']
```

### Avoid over-categorization

```yaml
# ✗ Too many - dilutes focus
categories: ['productivity', 'business', 'utilities', 'lifestyle', 'social']

# ✓ Focused - clear purpose
categories: ['productivity', 'business']
```

## Practical examples

### E-commerce app

```yaml
pwa:
    manifest:
        name: "ShopEasy"
        description: "Shop online with great deals."
        categories: ['shopping']
```

### Fitness tracker

```yaml
pwa:
    manifest:
        name: "FitTrack"
        description: "Track workouts and reach fitness goals."
        categories: ['health & fitness', 'sports']
```

### Task manager

```yaml
pwa:
    manifest:
        name: "TaskMaster"
        description: "Organize tasks and boost productivity."
        categories: ['productivity', 'business']
```

### News reader

```yaml
pwa:
    manifest:
        name: "NewsHub"
        description: "Stay updated with breaking news."
        categories: ['news']
```

### Recipe app

```yaml
pwa:
    manifest:
        name: "CookBook"
        description: "Discover and save delicious recipes."
        categories: ['food', 'lifestyle']
```

### Social network

```yaml
pwa:
    manifest:
        name: "ConnectApp"
        description: "Connect with friends and family."
        categories: ['social networking', 'social']
```

### Music streaming

```yaml
pwa:
    manifest:
        name: "MusicStream"
        description: "Stream millions of songs."
        categories: ['music', 'entertainment']
```

### Weather app

```yaml
pwa:
    manifest:
        name: "WeatherNow"
        description: "Accurate weather forecasts."
        categories: ['weather', 'utilities']
```

### Developer tools

```yaml
pwa:
    manifest:
        name: "APITester"
        description: "Test and debug APIs easily."
        categories: ['developer tools', 'productivity']
```

### Educational platform

```yaml
pwa:
    manifest:
        name: "LearnHub"
        description: "Interactive courses and tutorials."
        categories: ['education', 'books & reference']
```

## Platform-specific behavior

### Google play store

- Uses categories as hints
- May map to Play Store categories
- Helps with search and discovery
- Not guaranteed to match exactly

### Microsoft store

- Supports PWA categories
- Maps to Windows Store categories
- Influences app placement
- Used in search algorithms

### App stores in browsers

- Chrome Web Store considers categories
- Edge Add-ons uses categories
- Other browser stores may vary

### General web

- May be ignored by some platforms
- Still useful for:
  - PWA directories
  - Third-party catalogs
  - Search engine optimization
  - Documentation/metadata

## Best practices

### 1. Be specific

```yaml
# ✗ Too generic
categories: ['utilities', 'productivity']

# ✓ Specific
categories: ['task management', 'productivity']
```

### 2. Primary category first

```yaml
# ✓ Most important category first
categories: ['health & fitness', 'lifestyle']
```

### 3. Limit to 2-3 categories

```yaml
# ✓ Focused
categories: ['finance', 'business']

# ✗ Too many
categories: ['finance', 'business', 'productivity', 'utilities', 'social']
```

### 4. Use standard categories

```yaml
# ✓ Standard category
categories: ['productivity']

# ✗ Non-standard (may not be recognized)
categories: ['task-management-tools']
```

### 5. Match your app's purpose

Ensure categories align with your app's actual functionality:

```yaml
# For a photo editing app
categories: ['photo & video', 'graphics']  # ✓ Accurate

# Not appropriate if your app doesn't do this
categories: ['games']  # ✗ Misleading
```

## Common mistakes

### 1. Using too many categories

```yaml
# ✗ Excessive
pwa:
    manifest:
        categories: ['productivity', 'business', 'utilities', 'tools', 'lifestyle', 'organization']
```

**Impact**: Dilutes app's focus, may be ignored by platforms

**Fix**:
```yaml
# ✓ Focused
pwa:
    manifest:
        categories: ['productivity', 'business']
```

### 2. Mixing specific and general

```yaml
# ✗ Inconsistent specificity
categories: ['health & fitness', 'apps', 'mobile']
```

**Fix**:
```yaml
# ✓ Consistent level
categories: ['health & fitness', 'lifestyle']
```

### 3. Using non-standard names

```yaml
# ✗ Custom categories not recognized
categories: ['super-productivity-tools', 'amazing-apps']
```

**Fix**:
```yaml
# ✓ Standard categories
categories: ['productivity', 'business']
```

### 4. Wrong capitalization

```yaml
# ✗ Mixed case (inconsistent)
categories: ['Productivity', 'BUSINESS', 'health & Fitness']
```

**Fix**:
```yaml
# ✓ Lowercase (standard)
categories: ['productivity', 'business', 'health & fitness']
```

### 5. Mismatched categories

```yaml
# ✗ Categories don't match app purpose
pwa:
    manifest:
        name: "MusicPlayer"
        description: "Stream your favorite music"
        categories: ['games', 'sports']  # Wrong!
```

**Fix**:
```yaml
# ✓ Appropriate categories
pwa:
    manifest:
        name: "MusicPlayer"
        description: "Stream your favorite music"
        categories: ['music', 'entertainment']
```

## Testing categories

### 1. Validate in manifest

```javascript
// Check categories in your manifest
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        console.log('Categories:', manifest.categories);

        // Validate
        if (!manifest.categories || manifest.categories.length === 0) {
            console.warn('No categories specified');
        } else if (manifest.categories.length > 3) {
            console.warn('Too many categories - consider reducing to 2-3');
        }
    });
```

### 2. Check in DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "categories" array
4. Verify categories are lowercase
```

### 3. Test discovery

- Search for your app in stores by category
- Verify it appears in expected sections
- Check if similar apps are in same categories
- Test on multiple platforms (Play Store, Microsoft Store)

## Category selection checklist

Before finalizing categories:

- [ ] Primary category accurately describes app
- [ ] Using 1-3 categories maximum
- [ ] All categories are lowercase
- [ ] Categories are from standard list
- [ ] No overly generic categories (e.g., just "apps")
- [ ] Categories match app description
- [ ] Tested in app stores/platforms
- [ ] Similar to competitor categorization
- [ ] Makes sense to target users

## SEO and discovery impact

Categories affect:

1. **Store Search**: Apps appear in category searches
2. **Browse Discovery**: Users browsing categories find your app
3. **Similar Apps**: Powers "you might also like" features
4. **Filtering**: Enables category-based filtering
5. **Rankings**: May influence category-specific rankings

## Related properties

- [Description](description.md) - Describes app functionality
- [IARC Rating](iarc-rating-id.md) - Age and content rating
- [Name](README.md) - App identification

## Summary

Category selection best practices:
- ✓ Choose 1-3 most relevant standard categories
- ✓ Put primary category first
- ✓ Use lowercase for consistency
- ✓ Match categories to actual functionality
- ✓ Use specific categories when possible
- ✓ Check against standard category list
- ✗ Don't use more than 3 categories
- ✗ Don't make up custom categories
- ✗ Don't use misleading categories
- ✗ Don't forget to test on actual platforms

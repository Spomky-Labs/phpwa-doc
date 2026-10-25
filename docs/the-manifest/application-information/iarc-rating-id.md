# IARC Rating ID

The `iarc_rating_id` property allows you to specify an International Age Rating Coalition (IARC) certificate for your PWA, which provides globally recognized age and content ratings for digital applications.

## What is IARC?

IARC (International Age Rating Coalition) is a globally recognized system that provides age and content ratings for digital applications and games. It consolidates multiple rating systems into a single certification process:

- **ESRB** (Entertainment Software Rating Board) - North America
- **PEGI** (Pan European Game Information) - Europe
- **USK** (Unterhaltungssoftware Selbstkontrolle) - Germany
- **ClassInd** - Brazil
- **ACB** (Australian Classification Board) - Australia
- **GCAM** (Generic) - Global

## Purpose

IARC ratings help:
- **App stores**: Display appropriate age ratings
- **Parents**: Make informed decisions about content
- **Compliance**: Meet legal requirements in various regions
- **Distribution**: Qualify for certain app stores and platforms
- **Trust**: Build credibility with users

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        iarc_rating_id: "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7"
```
{% endcode %}

{% hint style="info" %}
The `iarc_rating_id` is optional. Only include it if you have obtained an official IARC certificate for your application.
{% endhint %}

## When You Need an IARC Rating

### Required For

- **Games**: All games distributed through major platforms
- **Social platforms**: Apps with user-generated content
- **Dating apps**: Applications with mature content
- **App stores**: Some stores require ratings for listing
- **Regional compliance**: Certain countries mandate ratings

### Not Required For

- **Internal business apps**: Enterprise or private applications
- **Productivity tools**: Office, note-taking, basic utilities
- **Information websites**: News, documentation, basic content
- **Simple utilities**: Calculators, weather apps, timers

## How to Obtain an IARC Rating

### Step 1: Determine Eligibility

IARC ratings are available through participating storefronts:
- Google Play Store
- Microsoft Store
- Oculus Store
- Nintendo eShop
- Other participating platforms

### Step 2: Complete the Questionnaire

Access the IARC questionnaire through your app store's developer console:

1. **Log in** to your developer account
2. **Navigate** to app ratings section
3. **Complete** the content questionnaire
4. **Review** the generated ratings
5. **Accept** and receive your certificate ID

### Step 3: Use the Certificate ID

After completing the questionnaire, you'll receive a unique certificate ID:

```yaml
iarc_rating_id: "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7"
```

## IARC Questionnaire Categories

The questionnaire evaluates your app across several categories:

### 1. Violence

- Physical conflict
- Blood and gore
- Weapons
- Injury depictions

### 2. Sexual Content

- Nudity
- Sexual references
- Sexual activity
- Romantic content

### 3. Language

- Profanity
- Crude humor
- Insults
- Slang

### 4. Controlled Substances

- Alcohol references
- Drug use
- Tobacco references

### 5. User Interaction

- User-to-user communication
- Sharing location
- Unrestricted internet access
- User-generated content

### 6. In-App Purchases

- Digital goods purchases
- Real money transactions
- Subscription features

## Rating Age Groups

Based on your responses, you'll receive ratings for different systems:

### ESRB (North America)

- **E** (Everyone) - Content suitable for all ages
- **E10+** (Everyone 10+) - Suitable for ages 10 and up
- **T** (Teen) - Content for ages 13 and up
- **M** (Mature) - Content for ages 17 and up
- **AO** (Adults Only) - Content for ages 18 and up

### PEGI (Europe)

- **PEGI 3** - Suitable for all ages
- **PEGI 7** - Suitable for 7 and older
- **PEGI 12** - Suitable for 12 and older
- **PEGI 16** - Suitable for 16 and older
- **PEGI 18** - Suitable for adults only

### Other Regional Ratings

IARC automatically generates equivalent ratings for all participating systems.

## Practical Examples

### Social Media App

```yaml
pwa:
    manifest:
        name: "SocialConnect"
        description: "Connect with friends and share moments."
        iarc_rating_id: "a1b2c3d4-e5f6-7890-1234-567890abcdef"
        # Likely rated: Teen/PEGI 12+ (user-generated content)
```

**Questionnaire responses might include**:
- User-to-user communication: Yes
- User-generated content: Yes
- Location sharing: Optional
- Mature content: Moderated

### Gaming App

```yaml
pwa:
    manifest:
        name: "SpaceRacer"
        description: "Fast-paced racing game with power-ups."
        iarc_rating_id: "b2c3d4e5-f6g7-8901-2345-678901bcdefg"
        # Likely rated: Everyone/PEGI 3 (mild cartoon violence)
```

**Questionnaire responses might include**:
- Violence: Cartoon/fantasy
- Blood: None
- Language: None
- In-app purchases: Yes (cosmetics)

### Productivity App (No Rating Needed)

```yaml
pwa:
    manifest:
        name: "TaskManager"
        description: "Organize your tasks efficiently."
        # No iarc_rating_id - not required for productivity tools
```

### News App

```yaml
pwa:
    manifest:
        name: "NewsDaily"
        description: "Stay informed with breaking news."
        iarc_rating_id: "c3d4e5f6-g7h8-9012-3456-789012cdefgh"
        # Likely rated: Teen/PEGI 12+ (news content)
```

## Certificate Management

### Certificate ID Format

IARC certificate IDs are UUID format:

```
e84b072d-71b3-4d3e-86ae-31a8ce4e53b7
│         │    │    │    │
└─ 8 hex  └─ 4 └─ 4 └─ 4 └─ 12 hex characters
   chars    hex  hex  hex
```

### Updating Your Rating

If your app's content changes significantly:

1. **Re-evaluate** content against original questionnaire
2. **Update** responses in developer console
3. **Obtain** new certificate ID (may be the same if rating unchanged)
4. **Update** manifest if ID changed

```yaml
# Before content update
iarc_rating_id: "old-certificate-id"

# After re-evaluation (if needed)
iarc_rating_id: "new-certificate-id"
```

### Certificate Expiration

IARC certificates generally don't expire, but you must:
- Update if content changes significantly
- Re-certify if app is republished after removal
- Comply with platform-specific renewal policies

## Platform Support

### Google Play Store

- Full IARC integration
- Automatic rating display
- Required for games
- Certificate obtained through Play Console

### Microsoft Store

- Supports IARC ratings
- Displays regional equivalents
- Certificate via Partner Center
- Required for certain categories

### Apple App Store

- Uses separate rating system (not IARC)
- IARC not directly applicable
- Use App Store Connect ratings

### Web App Stores

- Some support IARC in PWA manifests
- Display may vary by platform
- Check specific store requirements

## Common Mistakes

### 1. Using Placeholder or Fake IDs

```yaml
# ✗ Wrong - invalid ID
iarc_rating_id: "example-certificate"
iarc_rating_id: "12345"
iarc_rating_id: "pending"
```

**Impact**: Rating won't be recognized, may cause validation errors

**Fix**: Only use valid IARC certificate IDs obtained through official process

### 2. Including Rating When Not Needed

```yaml
# ✗ Unnecessary for simple productivity app
pwa:
    manifest:
        name: "CalculatorApp"
        iarc_rating_id: "..."  # Not needed
```

**Fix**: Omit `iarc_rating_id` if not required

### 3. Not Updating After Content Changes

```yaml
# ✗ Outdated rating after adding chat feature
iarc_rating_id: "old-id"  # Rated Everyone, but now has user communication
```

**Fix**: Re-evaluate and update when adding features like:
- User communication
- User-generated content
- In-app purchases
- Mature content

### 4. Confusing IARC with App Store Ratings

IARC is separate from:
- App Store star ratings (user reviews)
- Google Play store ratings
- Platform-specific content ratings

## Testing Your Rating

### 1. Validate Certificate ID

Check that your certificate is active:
- Log into the platform where you obtained it
- Verify certificate status
- Ensure app details match

### 2. Check Manifest

```javascript
// Verify IARC ID in manifest
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        if (manifest.iarc_rating_id) {
            console.log('IARC Rating ID:', manifest.iarc_rating_id);
        } else {
            console.log('No IARC rating specified');
        }
    });
```

### 3. View in DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "iarc_rating_id" field
```

## Legal and Compliance

### Regional Requirements

Some regions require age ratings:
- **Germany**: USK ratings mandatory for games
- **Brazil**: ClassInd ratings required
- **Australia**: ACB ratings for certain content
- **South Korea**: GRB ratings for games

### Penalties for Non-Compliance

- App removal from stores
- Fines in certain jurisdictions
- Loss of distribution rights
- Legal liability for inappropriate content access

### Recommended Practice

Even if not required:
- Consider obtaining rating for transparency
- Helps with parental controls
- Builds user trust
- Facilitates international distribution

## FAQs

**Q: Is IARC rating free?**
A: Yes, obtaining an IARC certificate is free through participating platforms.

**Q: How long does it take?**
A: Instant - you receive the certificate immediately after completing the questionnaire.

**Q: Can I use one IARC ID for multiple apps?**
A: No, each app requires its own certificate based on its specific content.

**Q: What if my app is for adults only?**
A: You'll receive appropriate mature ratings (M/AO/PEGI 18), which may limit distribution.

**Q: Do I need separate ratings for iOS?**
A: Yes, Apple uses its own rating system via App Store Connect.

**Q: Can I appeal a rating?**
A: Yes, contact the IARC support team or the platform where you obtained the rating.

## Resources

- **IARC Official Website**: https://www.globalratings.com/
- **Google Play IARC**: Available in Play Console
- **Microsoft Store IARC**: Available in Partner Center
- **IARC FAQs**: https://www.globalratings.com/faqs.aspx

## Related Properties

- [Description](description.md) - Describe your app's content
- [Categories](categories.md) - App classification
- [Name](README.md) - App identification

## Summary

IARC rating best practices:
- ✓ Obtain official certificate through participating platforms
- ✓ Only include if your app needs age rating
- ✓ Answer questionnaire honestly and accurately
- ✓ Update rating when content changes significantly
- ✓ Use correct certificate ID format (UUID)
- ✓ Comply with regional requirements
- ✗ Don't use fake or placeholder IDs
- ✗ Don't skip rating for games or social apps
- ✗ Don't forget to update after major content changes

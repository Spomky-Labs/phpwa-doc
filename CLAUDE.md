# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the documentation repository for the **PWA Bundle** (spomky-labs/pwa-bundle), a Symfony bundle that enables Progressive Web App (PWA) functionality in Symfony applications. The repository is a GitBook-based documentation site organized as markdown files.

## Repository Structure

The documentation is organized into major sections:

- **Root level**: Main guides (installation, how-tos, deployment)
- **the-manifest/**: PWA manifest configuration documentation
  - `application-information/`: App metadata (scope, ID, orientation, language, categories, etc.)
  - Icon, screenshot, shortcut, protocol/file handler, and share target configuration
- **the-service-worker/**: Service worker configuration and features
  - `workbox/`: Workbox library integration (caching strategies, offline fallbacks, background sync)
  - Push notifications, CSP, custom service worker rules
- **symfony-ux/**: Symfony UX components (connection status, prefetch, sync broadcast, background sync forms)
- **image-management/**: Icon and screenshot management guides
- **experimental-features/**: Non-standard PWA features (launch handler, display override, widgets, translations)

## Content Structure

- `SUMMARY.md`: GitBook table of contents - **always update this when adding/removing/moving pages**
- `.gitbook/assets/`: Images and assets referenced in documentation
- Each markdown file uses GitBook-flavored markdown with special syntax:
  - YAML frontmatter for metadata
  - `{% code %}` blocks with titles and line numbers
  - `{% hint %}` blocks for callouts (info, warning, success, danger)

## Documentation Standards

When editing documentation:

1. **Preserve GitBook syntax**: Keep `{% code %}`, `{% hint %}`, and other GitBook-specific tags intact
2. **Update SUMMARY.md**: When adding new pages, add entries to SUMMARY.md in the appropriate section
3. **Use relative links**: Link to other pages using relative paths (e.g., `the-manifest/icons.md`)
4. **Code examples**: Include practical Symfony/PHP configuration examples in YAML, Twig, or PHP as appropriate
5. **Bundle context**: This documents the `spomky-labs/pwa-bundle` - examples should show bundle configuration via `/config/packages/pwa.yaml` and Twig functions like `{{ pwa() }}`

## Key Technical Context

- **Installation**: Bundle installed via Composer (`composer require spomky-labs/pwa-bundle`)
- **Integration**: Bundle integrated using `{{ pwa() }}` Twig function in HTML `<head>` tag
- **Configuration**: Bundle configured in `/config/packages/pwa.yaml`
- **PWA Components**: Documentation covers both the Web App Manifest and Service Worker aspects of PWAs
- **Workbox**: The bundle uses Google's Workbox library for service worker functionality

## Branch Information

- The main branch corresponds to the next version to be released (`1.3`, `1.4` or `2.0` depending on the progress of work).
- Create PRs against the last version when possible.

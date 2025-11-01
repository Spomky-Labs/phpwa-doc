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

## Source Code Repository

The source code for the PWA Bundle is available at: https://github.com/Spomky-Labs/pwa-bundle

When documenting features or creating examples, you can reference the source code to understand implementation details.

## External Documentation Resources

When writing documentation for this bundle, you have access to specialized resources:

- **Symfony Documentation**: Use the `symfony-docs-expert` subagent via the Task tool to search and retrieve information from the official Symfony documentation at https://symfony.com/doc
- **Symfony UX**: Symfony UX components documentation is also available through the symfony-docs-expert subagent via Context7
- **Best practices**: Always reference official Symfony documentation for standard Symfony features (routing, configuration, Twig, etc.) to ensure accuracy and consistency

Example usage: When documenting how the bundle integrates with Symfony's routing system or how to configure services, use the symfony-docs-expert subagent to verify the correct Symfony approach.

## Branch Information

- **Documentation branches**: This repository uses version branches without the `.x` suffix (e.g., `1.3`, `1.4`, `2.0`)
- **Source code branches**: The source code repository uses version branches with the `.x` suffix (e.g., `1.3.x`, `1.4.x`, `2.0.x`)
- **Branch mapping**: Documentation branch `1.3` corresponds to source code branch `1.3.x`
- The main branch corresponds to the next version to be released (`1.3`, `1.4` or `2.0` depending on the progress of work)
- Create PRs against the last version when possible

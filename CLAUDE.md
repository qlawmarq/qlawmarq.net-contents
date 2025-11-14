# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a content repository for blog posts and static pages, managed separately from the main Next.js application. Content is organized by locale (en/ja) and synchronized with the parent application via `manifest.json`.

## Content Structure

```
contents/
  blogs/[locale]/*.md    # Blog posts with frontmatter metadata
  pages/[locale]/*.md    # Static pages (e.g., About)
  manifest.json          # Auto-generated index of all content
```

### Supported Locales

- `en` - English content
- `ja` - Japanese content

## Content File Format

All Markdown files must include YAML frontmatter with these required fields:

```yaml
---
title: "Article Title"
description: "Brief description for SEO and previews"
tags: ["Tag1", "Tag2"]
publishedAt: 2025-01-01T15:00:00.000Z
updatedAt: 2025-01-01T15:00:00.000Z
---
```

### Field Requirements

- **title**: Article title (both locales should convey the same meaning)
- **description**: SEO-friendly summary (150-160 characters recommended)
- **tags**: Array of topic tags for categorization
- **publishedAt**: ISO 8601 timestamp of initial publication
- **updatedAt**: ISO 8601 timestamp of last modification

## Manifest Generation

The `manifest.json` file is auto-generated from the parent application using:

```bash
pnpm generate:manifest
```

This command:
1. Scans all `.md` files in `blogs/` and `pages/` directories
2. Parses frontmatter metadata from each file
3. Generates slugs from filenames (removes `.md` extension)
4. Creates a JSON index organized by content type and locale

**Important**: Do not manually edit `manifest.json` - it will be overwritten by the generation script.

## Content Management Workflow

### Adding New Content

1. Create a new `.md` file in the appropriate directory:
   - Blog posts: `blogs/[locale]/your-slug.md`
   - Static pages: `pages/[locale]/your-slug.md`

2. Add required frontmatter metadata

3. Write content in Markdown format

4. Create corresponding file in other locale(s) for bilingual support

5. Regenerate manifest from parent application:
   ```bash
   cd .. && pnpm generate:manifest
   ```

### Editing Existing Content

1. Update the Markdown file directly

2. Update the `updatedAt` timestamp in frontmatter

3. Regenerate manifest if frontmatter metadata changed

### Content Synchronization

Content is managed through Google Cloud Storage in production:

- **Upload**: `pnpm upload:gcs` (from parent application)
- **Download**: `pnpm download:gcs` (from parent application)

## Obsidian Integration

This repository includes `.obsidian/` configuration for content management via Obsidian:

- **Obsidian Git plugin**: Configured for automatic git commits
- Use Obsidian as a content editor with live preview and graph view
- Git operations are automated through the plugin

## Git Workflow

This is a separate Git repository from the parent application. Commits should:

- Focus on content changes only (blog posts, pages, metadata)
- Use descriptive commit messages referencing the content being added/modified
- Keep manifest.json in sync with content changes

### Typical Commit Messages

- `docs: new blog post about [topic]`
- `docs: update [article-name] with [changes]`
- `chore: update manifest.json`
- `chore: add updatedAt to [article-name]`

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**IMPORTANT**: `AGENTS.md` in the project root contains comprehensive technical documentation. Read it for deep dives on any topic.

## Project Overview

Personal blog built on **Astro Modular** — an Astro 6 blog theme for Obsidian users. `src/content/` is an Obsidian vault; the theme publishes it as a performant blog. Deployed to Cloudflare Workers via `wrangler`.

## Commands

All commands use `pnpm`:

```bash
pnpm run dev          # Dev server on port 5000 (runs setup scripts first)
pnpm run build        # Production build
pnpm run preview      # Build then preview
pnpm run check-images # Find missing image references
pnpm run update       # Run update script
```

No test suite (`test` script exits with error).

`dev` and `build` run these pre-steps in order:
1. `scripts/sync-images.js` — copies media from content folders to `public/`
2. `scripts/process-aliases.js` — resolves Obsidian wikilink aliases/redirects
3. `scripts/generate-deployment-config.js` — writes platform config (`wrangler.toml`, etc.)
4. `scripts/generate-graph-data.js` — builds knowledge graph JSON at `/graph/graph-data.json`

## Architecture

### Content Collections (`src/content.config.ts`)

Five collections using Astro's `glob` loader:
- `posts` → `src/content/posts/` → `/posts/<id>`
- `pages` → `src/content/pages/` → `/<id>`
- `projects` → `src/content/projects/` → `/projects/<id>`
- `docs` → `src/content/docs/` → `/docs/<id>`
- `special` → `src/content/special/` → fixed URLs (home, 404, posts index, etc.)

All collections support both single-file (`post.md`) and folder-based (`post/index.md`) organization. Folder name becomes the URL slug.

### Configuration (`src/config.ts`)

Single source of truth for all site settings. Key sections: `theme`, `fonts`, `deployment.platform`, `commandPalette`, `homeOptions`, `postOptions`, `navigation`, `social`.

**Config Marker System**: `src/config.ts` contains `// [CONFIG:KEY]` comments used by the Obsidian plugin to update settings. **Never remove or modify these markers.** When adding new configurable values, add a matching `// [CONFIG:NEW_KEY]` comment on the line before.

### Theming (`src/themes/`)

- Theme definitions in `src/themes/index.ts`
- Exposed as `primary-*` and `highlight-*` Tailwind color scales
- CSS variables stored as space-separated RGB: `"255 255 255"`
- Theme saved to `localStorage` key `selectedTheme`; falls back to `siteConfig.theme`
- **Always use Tailwind theme variables** — never hardcode colors like `#fff` or `white`
- Always include dark mode: `text-primary-900 dark:text-primary-50`

### Routing (`src/pages/`)

- `index.astro` — Homepage
- `[...slug].astro` — Catch-all for pages and special content
- `posts/`, `projects/`, `docs/` — Dynamic routes per collection
- `api/` — JSON endpoints for command palette search (use `entry.id`, not `entry.slug`)
- `feed.xml.ts`, `rss.xml.ts`, `sitemap.xml.ts`, `robots.txt.ts`, `llms.txt.ts`

### Layouts (`src/layouts/`)

- `BaseLayout.astro` — Shell with Swup transitions, theme system, global JS hooks
- `PostLayout.astro`, `PageLayout.astro`, `ProjectLayout.astro`, `DocumentationLayout.astro`

### Markdown Processing (`src/config/`)

Remark/rehype plugin pipeline (order matters — plugins run sequentially and share AST state):

1. `remarkInternalLinks` — wikilinks (posts only) + standard markdown links (all types)
2. `remarkBreaks`
3. `remarkFolderImages` — ⚠️ MUST skip non-image extensions (audio/video/PDF)
4. `remarkObsidianEmbeds` — audio, video, YouTube, PDF, Twitter/X
5. `remarkBases`
6. `remarkImageCaptions`
7. `remarkMath`
8. `remarkCallouts` — `> [!note]` Obsidian callouts
9. `remarkImageGrids`
10. `remarkMermaid`
11. `remarkReadingTime`
12. `remarkToc`

Rehype: `rehypeKatex` (first), `rehypeMark`, `rehypeSlug`, `rehypeAutolinkHeadings`

### Key Utilities

- `src/config/dev.ts` — Logger and dev-mode settings (use this, not `console.log()`)
- `src/utils/internallinks.ts` — Wikilink + standard link processing, URL mapping
- `src/utils/graph-theme-colors.ts` — Shared D3 graph color system
- `src/utils/images.ts` — Image path resolution

## Critical Rules

### Content
1. **Never edit `src/content/` markdown files** without explicit user request — this is an Obsidian vault managed by the user. Content is sacred.
2. **Never add H1 to markdown content** — both `PostLayout` and `PageLayout` hardcode the `<h1>` from frontmatter. Content starts at H2.

### Astro v6 API
3. **Use `entry.id` not `entry.slug`** — `slug` is removed in Astro v6. In folder-based posts, `id` is just `"folder-name"`, NOT `"folder-name/index"`.

### Swup Page Transitions
4. **JavaScript must be re-initialized after every Swup transition** — Swup replaces DOM without firing `DOMContentLoaded`. Pattern:
   ```javascript
   function initMyComponent() { /* init code */ }
   window.initMyComponent = initMyComponent;
   document.addEventListener('DOMContentLoaded', initMyComponent);
   // In BaseLayout.astro swup hooks (both page:view AND visit:end):
   if (window.initMyComponent) window.initMyComponent();
   ```
5. **Never call `handleInitialHashScroll()` in Swup `visit:end` hook** — causes broken back/forward scroll behavior. Let the browser handle scroll restoration.

### Math Rendering
6. **KaTeX CSS**: Hide `.katex-html`, show `.katex-mathml` — NOT the reverse. Reversing causes math to render twice.

### Plugins
7. **Plugin order matters** — `remarkFolderImages` runs before `remarkObsidianEmbeds`. It must skip `.mp3`, `.wav`, `.ogg`, `.mp4`, `.webm`, `.mov`, `.pdf`, etc., or embeds get broken by WebP conversion.

### Styling
8. **Never use hardcoded colors** — use `primary-*`/`highlight-*` Tailwind classes with `dark:` variants.
9. **Never disable `vite.server.fs.strict`** — security requirement.
10. **Never disable Astro dev toolbar** — keep `devToolbar.enabled: true`. pnpm console errors from it are cosmetic/harmless.

### Linking
11. **Wikilinks `[[...]]` only resolve for posts** — for cross-collection linking (pages, projects, docs), use standard markdown links `[text](url)`.
12. **URL mapping is rendering-only** — linked mentions and graph view remain posts-only regardless of URL mapping.

### Logging
13. **Never use raw `console.log()` in production code** — use `src/config/dev.ts` logger.

### Deployment
14. **Set `deployment.platform` in `src/config.ts`** (not env vars) — build auto-generates the correct config file (`wrangler.toml`, `netlify.toml`, etc.).

### Favicons
15. **Favicon follows system theme only** — not the site theme toggle. Use `window.matchMedia('(prefers-color-scheme: dark)')`, not `localStorage`.

### Images
16. **Two separate image systems**:
    - Post *card* images: controlled by `siteConfig.postOptions.showPostCardCoverImages` config
    - Post *content* cover image: controlled by `hideCoverImage` frontmatter field
    - These are completely independent — don't mix them up.

### TOC
17. **Posts TOC**: controlled by global `postOptions.tableOfContents` setting (override per-post with `hideTOC: true`)
18. **Pages/projects/docs TOC**: independent of posts setting, defaults to showing; use `hideTOC: true` to hide.

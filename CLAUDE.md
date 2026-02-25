# mskalski.dev

Personal blog at mskalski.dev — Astro 5.x + TypeScript + TailwindCSS 4.x, based on AstroPaper.

## Project Structure

```
src/
  assets/           - Static images/media
  components/       - Astro UI components (Header, Footer, Card, Datetime, etc.)
  data/blog/        - Blog posts as Markdown files
  layouts/          - Page layout templates
  pages/            - Routes (index, about, posts, tags, search, archives, og.png.ts)
  scripts/          - Client-side scripts
  styles/           - Global CSS
  utils/            - Helpers (OG image gen, slugify, post sorting, transformers)
  config.ts         - SITE constants (author, URL, title, pagination, etc.)
  constants.ts      - Theme and other constants
  content.config.ts - Blog collection Zod schema
astro.config.ts     - Astro config (Shiki themes, remark plugins, TailwindCSS via Vite)
public/             - Served as-is (pagefind search index lives here post-build)
dist/               - Build output (gitignored)
```

## Tech Stack

**Language:** TypeScript 5.9
**Framework:** Astro 5.x
**CSS:** TailwindCSS 4.x (loaded via `@tailwindcss/vite` plugin)
**Syntax Highlighting:** Shiki — themes: `min-light` (light) / `nord` (dark)
**Search:** Pagefind (static, runs post-build)
**Linting:** ESLint + Prettier
**Dependency Mgmt:** pnpm
**Node:** 24 (via mise)

## Development Workflows

### Writing a Blog Post

1. Create `src/data/blog/your-post-slug.md`
   - Use subdirectory for organization: `src/data/blog/2025/post.md` → `/posts/2025/post`
   - Prefix dir with `_` to hide it from URL: `src/data/blog/_drafts/post.md` → `/posts/post`
2. Add required frontmatter: `title`, `description`, `pubDatetime`, `tags`
3. Write content in Markdown
4. Add `## Table of contents` heading to trigger auto-generated TOC
5. `pnpm run dev` → preview at localhost:4321
6. Set `draft: false` (or omit) when ready to publish

**Required frontmatter:**
```yaml
---
title: Your post title
description: Short description shown in post cards and SEO
pubDatetime: 2025-01-01T00:00:00Z
tags:
  - tag-name
---
```

**Optional frontmatter:** `featured`, `draft`, `ogImage`, `modDatetime`, `hideEditPost`, `timezone`, `canonicalURL`

### Styling / Theming Changes

1. Global styles → `src/styles/`
2. Theme constants (colors, etc.) → `src/constants.ts`
3. Tailwind customization → `astro.config.ts` (Vite plugins section)
4. Syntax highlight themes → `astro.config.ts` `shikiConfig.themes`
5. Site metadata (title, desc, author) → `src/config.ts` SITE object
6. `pnpm run dev` → verify at localhost:4321, test both light and dark mode
7. `pnpm run lint && pnpm run format:check` before committing

### Adding / Modifying a Component

1. Create or edit `.astro` file in `src/components/`
2. Use TailwindCSS utility classes — no inline styles
3. Use `client:load` / `client:idle` only when client-side JS is required — Astro is SSG by default
4. Import in the relevant page or layout
5. `pnpm run dev` to verify, then run quality gates before commit

## Build & Deploy

### Development

```bash
pnpm run dev      # Dev server at localhost:4321
mise run dev      # Same via mise task
```

### Production Build

```bash
pnpm run build    # astro check + astro build + pagefind --site dist + cp dist/pagefind public/
pnpm run preview  # Preview the built site locally
```

**Build steps (sequential):**
1. `astro check` — TypeScript type errors
2. `astro build` — Static site to `dist/`
3. `pagefind --site dist` — Build search index
4. `cp -r dist/pagefind public/` — Expose search index to dev server

### Deployment

GitHub Actions CI/CD — pushes to `main` trigger deployment automatically.

**Environment variables:**
- `PUBLIC_GOOGLE_SITE_VERIFICATION` — Google Search Console (optional, defined in `astro.config.ts`)

## Site Configuration

All site constants live in `src/config.ts`. Always reference these — never hardcode values:

```typescript
SITE.website    // "https://mskalski.dev/"
SITE.author     // "Marcin Skalski"
SITE.title      // Page/meta title
SITE.timezone   // "Europe/Warsaw"
SITE.postPerPage
SITE.showArchives
```

## Blog Post Schema

Defined in `src/content.config.ts` (Zod). Missing required fields break the build.

| Field | Required | Notes |
|-------|----------|-------|
| `title` | ✅ | |
| `description` | ✅ | |
| `pubDatetime` | ✅ | ISO date |
| `tags` | ✅ | defaults to `["others"]` |
| `author` | — | defaults to `SITE.author` |
| `draft` | — | `true` hides post from index |
| `featured` | — | shows on home page |
| `ogImage` | — | custom OG image |
| `modDatetime` | — | shows "updated" date |

## Quality Gates

Before committing:

- [ ] ESLint: `pnpm run lint`
- [ ] Prettier: `pnpm run format:check`
- [ ] TypeScript + Astro check + build: `pnpm run build`

```bash
pnpm run lint
pnpm run format:check
pnpm run build
```

## Common Commands

```bash
# Dev server
pnpm run dev

# Production build
pnpm run build

# Preview build
pnpm run preview

# Lint
pnpm run lint

# Format all files
pnpm run format

# Check formatting (non-destructive, for CI)
pnpm run format:check

# Sync Astro content types
pnpm run sync
```

## Anti-Patterns

**AVOID:**

- ❌ Hardcoding site URL, author, or title — use `SITE.*` from `src/config.ts`
- ❌ Missing required frontmatter (`title`, `description`, `pubDatetime`, `tags`) — build fails
- ❌ `client:load` on components that don't need client interactivity — Astro is SSG-first, keep hydration minimal
- ❌ Inline styles in `.astro` components — use Tailwind utility classes
- ❌ Editing AstroPaper example posts in `src/data/blog/` — they're theme docs, not real content
- ❌ Skipping `astro check` — TypeScript errors only surface at check time, not in dev server

## Extensibility

Add sections as blog evolves:
- Custom Shiki transformers → `src/utils/transformers/`
- OG image template variants → `src/utils/og-templates/`
- New remark/rehype plugins → `astro.config.ts` `remarkPlugins`
- Comment system (giscus) → `src/components/` + post layout

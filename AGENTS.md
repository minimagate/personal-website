# AGENTS.md — Michelangelo Gubinelli Personal Website

A comprehensive reference for AI agents and developers working on this project.

---

## Project Overview

- **Site**: [minimagate.github.io/personal-website](https://minimagate.github.io/personal-website)
- **Author**: Michelangelo Gubinelli
- **Stack**: [Zola](https://www.getzola.org/) static site generator (v0.19+), [Tera](https://keats.github.io/tera/) templates, vanilla CSS
- **Fonts**: Poppins + Geist Mono (Latin subsets as woff2, local only)
- **Licenses**: Poppins under [POPPINS.txt](LICENSES/POPPINS.txt), Geist Mono under [GEIST.txt](LICENSES/GEIST.txt)

The site is a minimalist personal portfolio with a blog, project showcase, and contact page. All pages are static, server-rendered at build time by Zola.

---

## Directory Structure

```
.
├── zola.toml                  # Zola configuration (base_url, title, markdown opts, extra vars)
├── content/                   # Markdown content (pages, sections, blog posts, projects)
│   ├── _index.md              # Home page (empty body, layout lives in templates/index.html)
│   ├── contacts.md            # Standalone page → templates/contacts.html
│   ├── blog/
│   │   ├── _index.md          # Blog section → templates/section.html → page.html
│   │   └── *.md               # Individual blog posts → templates/page.html
│   └── projects/
│       ├── _index.md          # Projects section → templates/section.html → project.html
│       └── *.md               # Individual project pages → templates/project.html
├── templates/                 # Tera templates
│   ├── base.html              # Root layout: <html>, <head>, <meta>, <body>, nav+footer
│   ├── index.html             # Home page: extends base, lists projects + blog posts
│   ├── page.html              # Blog post detail: extends base, LD+JSON, article
│   ├── section.html           # Section listing: extends base, post list
│   ├── project.html           # Project detail: extends base, article only
│   ├── contacts.html          # Contact page: extends base, article only
│   ├── 404.html               # 404 page: extends base, static message
│   └── partials/
│       ├── nav.html           # Primary navigation (home, blog, contacts)
│       ├── footer.html        # Footer with GitHub/Logven links + copyright
│       ├── post-list.html     # Reusable post listing (date + title links)
│       ├── projects-list.html # Project listing (category badge + title links)
│       └── arrow.html         # SVG diagonal arrow icon for external links
├── static/                    # Static assets served as-is
│   ├── styles/site.css        # All styles, single file
│   └── fonts/
│       ├── poppins-regular.woff2
│       ├── poppins-medium.woff2
│       ├── poppins-semibold.woff2
│       └── geist-mono.woff2
└── LICENSES/
    ├── GEIST.txt              # Geist Mono license
    └── POPPINS.txt            # Poppins license
```

**Note**: `public/` is the Zola build output directory. It is gitignored and should never be committed or edited directly.

---

## Configuration (`zola.toml`)

```toml
base_url = "https://minimagate.github.io/personal-website"
title = "Michelangelo Gubinelli"
description = "Designer, developer, founder at Logven, and mathematics student."
default_language = "en"
compile_sass = false
minify_html = true

[markdown]
smart_punctuation = true

[extra]
github = "https://github.com/minimagate"
logven = "https://logven.com"
```

- `compile_sass = false` — CSS is plain, no preprocessing
- `minify_html = true` — production builds are minified
- `[extra]` — arbitrary key/value store accessible as `config.extra.*` in templates
- `[markdown]` — `smart_punctuation = true` converts straight quotes to curly, `--` to en-dash, `---` to em-dash

---

## Content Authoring

### Frontmatter Conventions

Every content file must have TOML frontmatter wrapped in `+++`. Required fields by content type:

#### Blog Posts (`content/blog/*.md`)
```toml
+++
title = "Post Title"
description = "SEO description / lede"
date = 2026-08-03
authors = ["Author Name"]     # typically ["Michelangelo Gubinelli"]
+++
```
- Template: `page.html` (implicit — no `template` key needed)
- `date` is required for sorting and display
- `authors` is displayed in structured data but not rendered on-page

#### Projects (`content/projects/*.md`)
```toml
+++
title = "Project Name"
description = "Short project description"
slug = "project-slug"
date = 2026-08-03
template = "project.html"

[extra]
category = "Product"
+++
```
- `slug` must match the filename (without `.md`)
- `template = "project.html"` is required on each project page
- `[extra].category` is displayed in the project list on the home page
- `date` controls sort order in the projects section

#### Sections (`content/*/_index.md`)
```toml
+++
title = "Section Title"
description = "Section description for SEO"
sort_by = "date"
template = "section.html"
page_template = "page.html"    # or "project.html" for projects
+++
```
- `sort_by = "date"` — pages within the section are ordered by descending date
- `page_template` — the template used for child pages (default = `page.html`)
- `template` — the template used for the section listing itself
- Section body can be empty

#### Standalone Pages (`content/contacts.md`)
```toml
+++
title = "Page Title"
description = "SEO description"
date = 2026-01-01
template = "contacts.html"
+++
```
- Always set a `date` for consistency, even if not displayed
- `template` points to the specific template for that page

### Adding Content

**New blog post:**
```sh
# Create content/blog/my-new-post.md with frontmatter above
zola check   # validate
```

**New project:**
```sh
# Create content/projects/my-project.md
# Both the filename and the slug in frontmatter must match
zola check
```

**Adding to the nav:** Edit `templates/partials/nav.html`. Links use Zola's `get_url` with `@/` path prefix:
```html
<a href="{{ get_url(path='@/blog/_index.md') }}">blog</a>
```

---

## Template System

### Inheritance Chain

```
base.html
├── index.html          (home page)
├── page.html           (blog posts)
├── section.html        (blog listing, project listing)
├── project.html       (individual project detail)
├── contacts.html      (contact page)
└── 404.html           (not found)
```

All templates extend `base.html` via `{% extends "base.html" %}`.

### Block System

`base.html` defines these blocks that child templates override:

| Block | Purpose | Default |
|---|---|---|
| `title` | `<title>` tag | `config.title` |
| `description` | `<meta name="description">` | `config.description` |
| `social` | Open Graph + Twitter Card meta tags | site-wide defaults |
| `canonical` | `<link rel="canonical">` | `config.base_url` |
| `content` | Main body content | (empty) |

### Partials (Includes)

Partials are included via `{% include "partials/name.html" %}` and inherit the parent template's context.

| Partial | Used by | Variables it reads |
|---|---|---|
| `nav.html` | `base.html` | `config` (via `get_url`) |
| `footer.html` | `base.html` | `config.extra.github`, `config.extra.logven` |
| `post-list.html` | `section.html` | `section` (the current Zola section object) |
| `projects-list.html` | `index.html` | `projects` (set via `get_section`) |
| `arrow.html` | `footer.html` | None (pure SVG) |

### Key Template Details

- **`index.html`**: Fetches projects via `{% set projects = get_section(path="projects/_index.md") %}` and blog posts via `{% set all_blog = get_section(path="blog/_index.md") %}`. Blog posts on the home page filter out any post that has `extra.project` defined (a hook for project-linked posts if ever needed).

- **`page.html`**: Includes a `<script type="application/ld+json">` block with `BlogPosting` structured data. This is appropriate for blog posts. Uses `page.updated` if present, otherwise falls back to `page.date` for `dateModified`.

- **`section.html`**: Generic section listing. Renders the section title plus a `post-list.html` partial iterating over `section.pages`.

- **`project.html`**: Renders project title + prose content from markdown body.

- **`contacts.html`**: Renders contact page title + prose content.

---

## CSS Architecture

All styles live in a single file: `static/styles/site.css`.

### Design Tokens
- **Background**: `#fff`
- **Text**: Primary `#171717`, Secondary `#262626`, Muted `#525252`, Subtle `#737373`
- **Accent**: Selection highlight `#47a3f3`
- **Font family**: Poppins (sans-serif) + Geist Mono (code)
- **Max width**: `36rem` (≈576px) for body content
- **Spacing**: Based on `rem` units; margins use `1rem` / `2rem` / `4rem`
- **Breakpoints**: `768px` (tablet, row layouts), `1024px` (desktop centering)

### CSS Organization
1. `@font-face` declarations
2. Reset / box-sizing
3. `::selection` + `:root` variables
4. `html`, `body` base styles
5. `main`, `a`, typography base (`p`, headings)
6. `.nav-shell` / `#nav` / `.nav-items` — navigation
7. `.page-heading`, `.post-title`, `.intro` — page headers
8. `.post-list-wrap`, `.post-link`, `.post-row` — list items
9. `.post-meta` — blog post metadata row
10. `.prose` — article content (paragraphs, links, headings, lists, code, tables)
11. `footer` — footer layout and links
12. `@media` queries — responsive overrides

### Rules for Styling
- Keep all CSS in `site.css` — no inline styles, no additional stylesheets
- Use the existing class names for new elements
- Link colors follow the `color: inherit; text-decoration: none` pattern
- Links in prose get `text-decoration: underline` with a muted `#a3a3a3` color
- Font sizes use `rem`; line-heights are unitless
- CSS is loaded via `get_url(path='styles/site.css')` in `base.html` (Zola cache-busts automatically)

---

## Build & Development

### Local Development
```sh
zola serve          # Serves at http://127.0.0.1:1111, live reload
```

### Validation
```sh
zola check          # Validates templates, internal links, and structure
```

### Production Build
```sh
zola check && zola build   # Outputs to public/
```

Before deploying, update `base_url` in `zola.toml` to the production domain.

### Deployment
The site is deployed to GitHub Pages at `minimagate.github.io/personal-website`. The `public/` directory is the deployable artifact.

---

## Conventions & Rules

### General
- **No JavaScript**. The site has zero client-side JS.
- **No external dependencies** beyond Zola itself. No npm, no CDN fonts, no analytics.
- **All assets are local**: Fonts are self-hosted as woff2, CSS is a single local file.
- **Dates use ISO 8601 format** (`YYYY-MM-DD`) in frontmatter, displayed as human-readable (`%B %-d, %Y`).
- **File naming**: Blog posts and projects use kebab-case slugs matching their frontmatter `slug` (for projects) or derived from `title` (for blog posts).
- **Commit messages**: Follow [Conventional Commits](https://www.conventionalcommits.org/).

### Frontmatter
- Every content file must have `title` and `description`.
- Every content file should have a `date` (even standalone pages).
- `authors` is a TOML array of strings: `authors = ["Name"]`.
- Never use `authors` as a plain string — always wrap in `[]`.

### Templates
- Template filenames are singular (`page.html`, `project.html`, `contacts.html`).
- Content files reference templates via the `template` key.
- Partials go in `templates/partials/` and use kebab-case.
- Variables passed to partials through includes use the parent template's context — no explicit parameter passing.
- Zola provides `config`, `page`, `section`, and `current_url` as built-in context variables.

### Content
- Blog posts use `templates/page.html` (with LD+JSON structured data).
- Project pages use `templates/project.html`.
- Section listings use `templates/section.html`.
- Body content is standard Markdown with `smart_punctuation = true`.

### CSS
- Single file: `static/styles/site.css`.
- No CSS preprocessing (Sass is disabled in zola.toml).
- No CSS framework or utility library.
- Prefer existing utility classes (`.prose`, `.post-row`, `.post-link`, `.intro`) over creating new ones.
- Use Poppins for all proportional text; Geist Mono for code.

---

## Template Variable Reference

### Zola Built-ins (always available)

| Variable | Description |
|---|---|
| `config` | Parsed `zola.toml` (e.g., `config.title`, `config.extra.github`) |
| `page` | Current content page (title, date, content, permalink, description, slug, extra, updated) |
| `section` | Current section (title, pages, permalink, description) |
| `current_url` | Full URL of the current page |
| `get_url(path='...')` | Resolves `@/` aliases to actual URLs |
| `get_section(path='...')` | Loads a section by its `_index.md` path |

### Custom Extra Fields (in `[extra]`)

| Field | Used In | Purpose |
|---|---|---|
| `extra.github` | `zola.toml`, footer | GitHub profile link |
| `extra.logven` | `zola.toml`, footer | Logven website link |
| `extra.category` | Project pages | Category badge in project listing |

---

## Quick Reference for Common Tasks

### Add a new blog post
```sh
cp content/blog/_template.md content/blog/my-post.md
# Edit frontmatter: title, description, date, authors
# Write body in Markdown
zola check
```

### Add a new project
```sh
cp content/projects/_template.md content/projects/my-project.md
# Edit frontmatter: title, description, slug, date, category
# Write body in Markdown
zola check
```

### Change the navigation
Edit `templates/partials/nav.html`. Add or remove `<a>` tags inside `.nav-items`. Use `get_url(path='@/...')` for proper URL resolution.

### Change the footer
Edit `templates/partials/footer.html`. External links use `target="_blank" rel="noopener noreferrer"` and include the arrow partial.

### Add a new page-level template
1. Create `templates/my-template.html` extending `base.html`
2. Override blocks as needed (`title`, `description`, `social`, `content`)
3. In the content file, set `template = "my-template.html"`
4. Follow the pattern in `contacts.html` for simple pages

### Deploy
```sh
zola check && zola build
# Deploy public/ to GitHub Pages
```

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Frontend Mentor "Time tracking dashboard" challenge solution — a responsive dashboard with Daily/Weekly/Monthly timeframes for activity cards (Work, Play, Study, Exercise, Social, Self Care).

The user's behavioral guidelines for AI assistance live in `./AGENTS.md` (junior-level mentor mode, no copy-paste solutions, Socratic guidance). **Read AGENTS.md first** — it overrides default behavior for this repo.

## Development Commands

```bash
npm install           # install deps
npm run dev           # vite dev server (port 5173, strictPort, opens browser)
npm run build         # production build to dist/
npm run preview       # preview the production build

npm run lint:scss     # stylelint check
npm run lint:scss:fix # stylelint --fix --cache
npm run lint:html     # html-validate *.html
npm run lint:html:fix # html-validate --fix *.html
npm run format        # prettier --write
npm run format:check  # prettier --check
npm run editorconfig  # npx editorconfig-cli check
npm run images        # generate WebP + AVIF from src/assets/images/*
```

Pre-commit hook (Husky + lint-staged) runs Prettier, Stylelint, editorconfig-cli, and html-validate on staged files. `npm run prepare` installs the hook.

## Architecture

### Build system

- **Vite 8** with `type: "module"`. `vite.config.js` switches the `base` path between `/` (dev) and `/time-tracking-dashboard/` (production, for GitHub Pages).
- **PostCSS** runs `autoprefixer` only.
- **Sass** compiles `src/scss/main.scss` into the bundle imported from `src/main.js`.
- **No framework** — vanilla JS, vanilla SCSS.

### Entry point chain

`index.html` → `src/main.js` (one import: `./scss/main.scss`)

The HTML at root (`index.html`) is the canonical document. All page content lives there — there is no templating layer.

### SCSS structure (7-1 pattern, modern `@use`/`@forward`)

`src/scss/main.scss` is the manifest. Order matters and is intentional:

1. `abstracts/` — variables, mixins, functions, placeholders (no CSS output; `@use`d with `as *` or namespace)
2. `vendors/` — `_normalize.scss` (third-party reset; ignored by stylelint)
3. `base/` — reset, fonts, typography, base element styles, utilities
4. `layout/` — page shell (header, grid)
5. `components/` — reusable UI (currently empty)
6. `pages/` — page-specific styles (currently empty)
7. `themes/` — theme variants (currently empty)

Each directory has an `_index.scss` that `@forward`s its partials. Add a new partial in a directory and forward it from the local `_index.scss` to make it available.

When you need a variable or mixin in another file: `@use '../abstracts' as abstracts;` then `abstracts.$name` or `@use 'abstracts' as *;` for bare access.

### Data source

`data.json` at repo root holds the activity stats (title + daily/weekly/monthly current+previous). It is the canonical mock data referenced by the challenge; the dashboard's `index.html` contains hardcoded values that mirror this JSON. New code should fetch from `data.json` via `import data from '../data.json'` (Vite supports JSON imports).

### Assets

- `src/assets/images/` — source PNG/JPG inputs. `npm run images` (Sharp) generates `.webp` and `.avif` siblings in-place. Use `<picture>` with AVIF → WebP → original fallback.
- `src/assets/fonts/Rubik/` — local woff2 files.
- `public/` — files served as-is (favicons, `site.webmanifest`, `robots.txt`, `og-image.png`).

### Design tokens

`src/scss/abstracts/_variables.scss` defines design tokens. Color values are referenced in `docs/style-guide.md` (HSL primaries: Purple 600, Orange/Blue/Pink/Green/Purple/Yellow 300-400 per activity; Navy 200/900/950 neutrals). Typography: Rubik 300/400/500 at 18px body.

CSS custom properties (declared in `src/scss/base/_base.scss`) provide runtime tokens (`--font-family-sans-serif`, `--font-size-base`).

## Stylelint

Config at `stylelint.config.js` extends `stylelint-config-standard-scss` and adds `stylelint-order` with grouped property ordering: `layout → box-model → flex → grid → visual → typography → interaction`. Out-of-order declarations are reported as **warnings**, not errors. Treat them as errors anyway — CI doesn't enforce but consistency matters.

`_normalize.scss` is ignored.

## Deployment

`.github/workflows/deploy.yml` runs on push to `main`: `npm ci` → `npm run build` → upload `dist/` → `actions/deploy-pages@v4`. Site is at `https://vlrnsnk.github.io/time-tracking-dashboard/`, which is why `vite.config.js` sets `base: '/time-tracking-dashboard/'` in production mode.

## VS Code setup

`.vscode/settings.json` enables format-on-save (Prettier) and Stylelint fix-on-save. Recommended extensions in `.vscode/extensions.json`: stylelint, auto-rename-tag, editorconfig, prettier.

## Things to be aware of

- `index.html` is currently a flat text dump with HTML comments as data delimiters — it needs restructuring into semantic markup with the JS layer rendering from `data.json`. The daily/weekly/monthly toggle is the headline JS feature.
- The Junior-mentor posture from AGENTS.md means: explain "why", give hints, don't paste solutions. When the user asks "how do I…", ask what they've tried first.
- ESLint, Prettier, and Stylelint run on pre-commit via lint-staged. Don't bypass with `--no-verify`.
- The README has unfilled `{{LEARNING_*}}` and `{{PERF_*}}` placeholders — those are the user's personal write-up fields, not for you to fill.

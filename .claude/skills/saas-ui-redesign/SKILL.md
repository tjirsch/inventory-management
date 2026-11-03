---
name: saas-ui-redesign
description: Redesign a Vue 3 application's UI into a modern SaaS-style interface - left vertical sidebar navigation instead of a top nav bar, a design-token system, consistent spacing, and a polished professional look. Use when asked to modernize, restyle, or "make professional" a Vue 3 app's layout, or to replace a top navigation bar with a sidebar.
---

# SaaS UI Redesign (Vue 3)

Turn an existing Vue 3 app into a clean SaaS-style interface: fixed left sidebar, slim top bar, token-driven styles, consistent spacing. This is a **visual and layout** change. Behavior, routes, data flow, and API calls stay exactly as they are.

## Ground rules

- **Preserve functionality.** Do not change routes, props, emits, API calls, composables, or computed logic. If something looks broken, note it in the final report instead of fixing it as part of the redesign.
- **Tokens before components.** Every color, radius, shadow, and spacing value must come from a CSS custom property. Hard-coded hex values or arbitrary pixel values in component styles count as a defect.
- **One spacing scale.** Use only the 4px-based scale below. Do not use values like 13px, 18px, or 22px.
- **Respect project rules.** If the project's CLAUDE.md requires delegating `.vue` edits to a subagent (e.g. `vue-expert`), do so and pass this skill's spec in the brief. Respect "no emojis" and similar rules, and use inline SVG icons instead.
- **No new dependencies** unless the user asks. Plain CSS and inline SVG icons are enough.
- **Comment non-obvious choices** (e.g. why a breakpoint or a z-index is what it is).

## Workflow

### 1. Audit (read-only)

Before editing, collect:
- The layout shell (usually `App.vue`): header, nav, global bars (filters, etc.), `<router-view>`, and global modals.
- The router (`main.js` / `router/`): every route, plus any view files that are not routed.
- Global styles: where they live, and which shared classes views depend on (`.card`, `.stat-card`, `.page-header`, `.badge`, tables).
- Hard-coded colors and spacing values: `grep -rhoE '#[0-9a-fA-F]{3,6}\b' src | sort | uniq -c | sort -rn` shows the de-facto palette.
- i18n: nav labels may come from `t('...')`. Keep them and don't hard-code strings.
- Header widgets that must move (language switcher, profile menu, notifications).

Write a short plan: what moves where, which shared classes get restyled, and which files change.

### 2. Design tokens

Put the tokens in the global stylesheet (e.g. the unscoped `<style>` in `App.vue` or `src/styles/tokens.css` imported in `main.js`). Adapt the palette to the project's design system if it has one, and keep the token names.

```css
:root {
  /* Surfaces */
  --color-bg: #f8fafc;          /* page background */
  --color-surface: #ffffff;     /* cards, sidebar, top bar */
  --color-surface-muted: #f1f5f9;
  --color-border: #e2e8f0;

  /* Text */
  --color-text: #0f172a;
  --color-text-muted: #64748b;
  --color-text-subtle: #94a3b8;

  /* Brand / interactive */
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-primary-soft: #eff6ff;

  /* Status */
  --color-success: #16a34a;  --color-success-soft: #f0fdf4;
  --color-info: #2563eb;     --color-info-soft: #eff6ff;
  --color-warning: #d97706;  --color-warning-soft: #fffbeb;
  --color-danger: #dc2626;   --color-danger-soft: #fef2f2;

  /* Spacing - 4px scale; use ONLY these */
  --space-1: 4px;  --space-2: 8px;  --space-3: 12px; --space-4: 16px;
  --space-5: 20px; --space-6: 24px; --space-8: 32px; --space-10: 40px; --space-12: 48px;

  /* Shape & depth */
  --radius-sm: 6px; --radius-md: 8px; --radius-lg: 12px;
  --shadow-sm: 0 1px 2px rgb(15 23 42 / 0.05);
  --shadow-md: 0 4px 12px rgb(15 23 42 / 0.08);

  /* Type */
  --font-sans: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --text-xs: 12px; --text-sm: 13px; --text-base: 14px; --text-lg: 16px; --text-xl: 20px; --text-2xl: 24px;

  /* Layout */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --topbar-height: 56px;
  --content-max-width: 1400px;
}
```

### 3. Layout shell

Replace the top nav with this structure in the root layout component:

```
.app-shell (CSS Grid: sidebar | main)
├── <aside class="sidebar">           fixed, full height, left
│   ├── .sidebar-brand                logo / product name (height = --topbar-height)
│   ├── <nav class="sidebar-nav">     router-links with icon + label
│   │   └── optional .sidebar-section-label groups ("Operations", "Finance", ...)
│   └── .sidebar-footer               collapse toggle, optionally user/profile
└── .app-main
    ├── <header class="topbar">       sticky; page title or breadcrumb left; language switcher, profile menu right
    ├── global bars (e.g. FilterBar)  directly under topbar, same horizontal padding as content
    └── <main class="content">        padding var(--space-8); max-width var(--content-max-width)
        └── <router-view />
```

```css
.app-shell { display: grid; grid-template-columns: var(--sidebar-width) 1fr; min-height: 100vh; background: var(--color-bg); }
.app-shell.is-collapsed { grid-template-columns: var(--sidebar-width-collapsed) 1fr; }
.sidebar { position: sticky; top: 0; height: 100vh; display: flex; flex-direction: column;
  background: var(--color-surface); border-right: 1px solid var(--color-border); overflow-y: auto; }
.app-main { min-width: 0; /* lets wide tables scroll instead of stretching the grid */ }
.topbar { position: sticky; top: 0; z-index: 20; height: var(--topbar-height); display: flex; align-items: center;
  justify-content: space-between; padding: 0 var(--space-8); background: var(--color-surface); border-bottom: 1px solid var(--color-border); }
.content { padding: var(--space-8); max-width: var(--content-max-width); margin: 0 auto; }
```

Extract the sidebar into its own component (`components/AppSidebar.vue`) with the nav items as a data array (`{ to, labelKey, icon }`) rendered with `v-for` keyed by `to`. Do not hand-write repeated `<router-link>` blocks.

### 4. Sidebar spec

- **Nav item:** height 36-40px, padding `var(--space-2) var(--space-3)`, gap `var(--space-3)` between icon and label, `border-radius: var(--radius-md)`, font `var(--text-sm)` weight 500, color `--color-text-muted`.
- **Hover:** background `--color-surface-muted`, color `--color-text`.
- **Active:** background `--color-primary-soft`, color `--color-primary`, icon also primary. Use Vue Router's built-in `exact-active-class` / `active-class` (set `exact` behavior for `/`) instead of manual `$route.path ===` comparisons.
- **Icons:** 18px inline SVG, `stroke="currentColor"`, stroke-width 1.75, outline style (Lucide/Feather geometry). Keep one consistent icon set.
- **Section labels (optional):** `--text-xs`, uppercase, letter-spacing 0.05em, `--color-text-subtle`, margin `var(--space-6) 0 var(--space-2)`.
- **Collapse:** a toggle in `.sidebar-footer` switches to icon-only (`--sidebar-width-collapsed`). Hide labels and show them as a `title` attribute. Persist the state in `localStorage`, wrapped in try/catch.
- **Accessibility:** `<nav aria-label="Main">`, `aria-current="page"` on the active link (router-link does this), a visible `:focus-visible` ring (`outline: 2px solid var(--color-primary); outline-offset: 2px`), and the collapse button gets `aria-expanded` and `aria-label`.

### 5. Responsive behavior

- **>= 1024px:** sidebar expanded by default.
- **768-1023px:** sidebar collapsed (icon-only) by default.
- **< 768px:** sidebar becomes an off-canvas drawer. Show a hamburger button in the topbar. The drawer slides over the content with a backdrop and closes on backdrop click, Escape, and route change. Grid becomes a single column. Content padding drops to `var(--space-4)`.
- No horizontal page scroll at any width. Wide tables scroll inside their own container (`overflow-x: auto`).

### 6. Restyle shared components

Apply these globally so every view benefits without per-view rewrites:

| Element | Spec |
|---|---|
| Page header | Title `--text-2xl` / 600, description `--text-sm` muted, margin-bottom `var(--space-6)`. Actions are right-aligned on the same row. |
| Card | `--color-surface`, 1px `--color-border`, `--radius-lg`, `--shadow-sm`, padding `var(--space-6)`. Header row margin-bottom `var(--space-4)`. |
| Stat / KPI card | Label `--text-xs` uppercase muted, value `--text-2xl` / 600 with `font-variant-numeric: tabular-nums`, optional delta badge. |
| Grids | `display: grid; gap: var(--space-6)`. KPI rows use `repeat(auto-fit, minmax(220px, 1fr))`. |
| Tables | Header `--text-xs` uppercase muted on `--color-surface-muted`. Rows `--text-sm` with padding `var(--space-3) var(--space-4)`, 1px bottom border, hover `--color-surface-muted`. Right-align numbers with tabular-nums. |
| Badges | Pill, `--text-xs` / 500, padding `2px var(--space-2)`, status `*-soft` background with the matching status text color. |
| Buttons | Height 36px, `--radius-md`, primary = `--color-primary` fill. Secondary = surface with border. Every button gets a focus-visible ring. |
| Inputs / selects (filter bar) | Height 36px, 1px border, `--radius-md`, focus border `--color-primary` plus a soft ring. |
| Modals | `--radius-lg`, `--shadow-md`, padding `var(--space-6)`, backdrop `rgb(15 23 42 / 0.4)`. |

Then sweep each view's scoped styles and replace hard-coded colors and spacing with tokens. Leave chart and data-visualization colors alone unless they clash, since they may encode meaning.

### 7. Verify

1. Start the app (use the project's start skill/command if one exists).
2. With Playwright (or the project's preferred browser tool), visit **every route** at 1440px, 900px, and 375px widths. Screenshot each and check:
   - The sidebar's active state matches the current route, including `/` not staying active on sub-routes.
   - Collapse toggle works and the state persists across reload.
   - The mobile drawer opens and closes (backdrop, Escape, navigation).
   - Header widgets (language switcher, profile menu, dropdowns) still open and aren't clipped by the sidebar or the sticky topbar (check z-index).
   - Global filters still filter the data.
   - No horizontal scroll, and no console errors.
3. Re-run the hex-color grep from step 1. Remaining hard-coded colors should be token definitions, chart palettes, or justified exceptions.

### 8. Report

Summarize what changed (files, new components, tokens), before/after screenshots or paths, any pre-existing bugs you noticed but didn't touch, and any follow-ups (e.g. dark mode is now just a token override away).

## Anti-patterns

- Rewriting view logic "while you're in there".
- Mixing a new token system with leftover hard-coded values in the same component.
- Using `index` as the `v-for` key for nav items.
- Making the sidebar `position: fixed` without reserving its width. The grid approach above avoids content sliding under it.
- Heavy shadows, gradients, or more than one accent color. A SaaS look comes from restraint, whitespace, and alignment.

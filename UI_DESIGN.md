# UI Design — Neumorphism Layer

How the Mori-Neo redesign is structured, and how to extend it.

## Philosophy

The redesign is a **CSS-only overlay** on the upstream UI. Markup, class
names, JavaScript behavior, i18n keys, and preset storage are unchanged, so
upstream updates merge cleanly. Neo-brutalist surfaces (hard
`1px solid var(--primary)` borders, `4px 4px 0px` offset shadows) are replaced
with soft neumorphic elevation: dual shadows (light top-left highlight,
dark bottom-right shade) on warm-neutral surfaces.

**Rules of the system:**

- Raised surfaces cast a dual shadow; inset surfaces are carved into the
  background; accent-filled elements (near-black / cream) use a solid depth
  shadow; pressed states go inset.
- Depth changes alone never communicate state — always paired with color,
  icon, or text changes.
- All body text meets WCAG AA (≥ 4.5:1); control boundaries meet ≥ 3:1.

## File map

| File                        | Role                                                        |
| --------------------------- | ----------------------------------------------------------- |
| `public/css/neumorphism.css`| **The design layer** — tokens, primitives, all overrides. Loaded after every upstream sheet, before `rtl.css`. |
| `public/css/variables.css`  | Palette tokens (light/dark), radius presets, **Elevation presets**, alias tokens. |
| `public/css/rtl.css`        | Gains RTL flips for the toggle knob.                        |
| `public/css/style.css`      | Import order: `variables → base → components → home → history → settings → modals → **neumorphism** → rtl`. |

Because `neumorphism.css` loads last, equal-specificity ties resolve in its
favor; upstream `!important` declarations and higher-specificity rules
(`[data-theme="dark"] …`) must be matched explicitly — several rules here
exist solely for that purpose.

## Tokens

### Palette (in `variables.css`)

| Token             | Light                | Dark                 |
| ----------------- | -------------------- | -------------------- |
| `--bg-color`      | `#e7e5df` (warm paper)| `#26262b` (graphite) |
| `--surface`       | `#edebe5`            | `#2d2d34`            |
| `--primary`       | `#1a1917` (ink)      | `#f8f8fa` (cream)*   |
| `--on-primary`    | `#f6f4ee`            | `#1c1c20`            |
| `--text-main`     | `#1a1917` (14.7:1)   | `#eceded` (11.7:1)   |
| `--text-secondary`| `#5f5b53` (5.7:1)    | `#a3a5ad` (5.6:1)    |
| `--color-danger`  | `#c62828`            | `#c62828` (white text 5.6:1) |
| `--danger-text`   | `#c62828` (4.7:1)    | `#ff6b6b` (4.9:1)    |

\* `appearance.js` also sets `--primary` inline per theme
(`#1a1917` / `#fffbf2`); `--primary-rgb` tracks it.

Alias tokens for upstream rules that predate the redesign: `--border`,
`--border-color`, `--text-muted`, `--text-primary`, `--surface-hover`,
`--bg-card`, `--bg-secondary`, `--bg-main`, `--radius-md`.

### Elevation (in `neumorphism.css`, geometry in `variables.css`)

```
--neo-y / --neo-blur        raised shadow offset & blur
--neo-hover-y / -hover-blur  hover deepening
--neo-i-y / --neo-i-blur     inset depth
--neo-hi / --neo-sh          light/dark shadow colors (theme-aware)
--neo-raise / -raise-sm / -float / -inset / -inset-sm / -raise-hover
--neo-accent-sh / -press     depth shadow for accent-filled elements
--neo-divider                hairline separators (9% of primary)
```

**Elevation presets** (Settings → "Elevation", stored under the legacy
`glass-*` class names so existing settings keep working):

| Preset (body class) | Effect                             |
| ------------------- | ---------------------------------- |
| `glass-off` ("Flat")    | 1px / 3px blur — nearly flush  |
| `glass-subtle` ("Soft") | 4px / 11px — default           |
| `glass-deep` ("Deep")   | 7px / 18px — dramatic relief   |

The `--color-danger`/`--danger-text` split exists because JS writes
`color: #ffffff` inline on `var(--color-danger)` badges; the token itself
must keep white text readable.

## Component treatments

| Component               | Treatment                                              |
| ----------------------- | ------------------------------------------------------ |
| Header / cards / lists  | Raised (`--neo-raise`), radius `--radius-l/xl`         |
| URL input, textarea, wells, counters, slider track | Inset (carved) + focus ring `0 0 0 2px` |
| Bottom nav              | Floating pill (`--neo-float`), active item = inset `bg-color` |
| Primary CTA (`#downloadBtn`, `.dl-all-btn`, done/confirm) | Accent fill + `--neo-accent-sh`; hover via `color-mix()`; press = inset |
| Toasts, dropdowns, modals, download bubble | Float (`--neo-float`)                          |
| Toggle switch           | Inset track (50% primary) + raised knob                |
| Chips / pills / badges  | Smallest raise or 5–8% primary tint                    |
| Skeletons / progress    | Tinted shimmer, accent progress bar                    |

Corner presets (`corner-sharp` / `corner-modern` / `corner-round`) and
`compact-mode` still apply; presets remap `--radius-*` and reach the floating
nav too. `prefers-reduced-motion` zeroes animations/transitions.

## Label changes (English only)

| Key                    | Before          | After       |
| ---------------------- | --------------- | ----------- |
| `label-glassmorphism`  | Glassmorphism   | **Elevation** |
| `glass-off`            | Off (Solid)     | **Flat**    |
| `glass-subtle`         | Subtle          | **Soft**    |
| `glass-deep`           | Deep Frosted    | **Deep**    |

HTML fallback text in `public/index.html` matches. Other locales keep their
original wording (keys and stored values unchanged, so settings migrate
transparently).

## Accessibility

- Global `:focus-visible` outline rings (including visually-hidden checkbox
  inputs); text inputs regain an outline over upstream `outline: none`.
- WCAG AA verified for the palette (contrast ratios above) and for tinted
  status chips (tint + `--text-main`).
- `prefers-reduced-motion` honored; slim themed scrollbars; `::selection`
  tinted.
- `share.html` no longer disables pinch zoom (`user-scalable=no` removed);
  `viewport-fit=cover` on both entry pages.
- RTL: divider uses `border-inline-end`; toggle knob direction flips in
  `rtl.css`.

## Verification (local)

```bash
npx esbuild public/css/style.css --bundle --outfile=/dev/null   # parse + import graph
node /tmp/opencode/uicheck/boot.js                              # jsdom boot smoke (light)
node /tmp/opencode/uicheck/boot.js --dark                       # jsdom boot smoke (dark)
```

The boot harness bundles `app.js` with esbuild, boots the real page in
jsdom over `http://localhost:8123`, and asserts theme, body preset classes,
pages/nav, i18n, and zero script errors.

Selector audit (every class/id styled by `neumorphism.css` must exist in
markup/JS) and a WCAG contrast sweep were run over the final layer; see
`PROJECT_STATUS.md` for results.

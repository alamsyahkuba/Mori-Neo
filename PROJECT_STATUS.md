# PROJECT_STATUS — Mori-Neumorphism (Mori-Neo)

Baseline analysis of the repository **before** any UI or CI/CD changes.
Fork of <https://github.com/coflyn/Mori> at `v4.4.0` (`origin/main` == `upstream/main` at time of writing).

---

## 1. Technology stack

| Area             | Technology                                                                 |
| ---------------- | -------------------------------------------------------------------------- |
| Frontend         | Vanilla JavaScript (ES modules), no framework, no bundler                  |
| Markup           | Single static `public/index.html` (+ `public/share.html`)                  |
| Styling          | Plain CSS — 9 files imported through `public/css/style.css`                |
| Design tokens    | `public/css/variables.css` (CSS custom properties, light/dark themes)      |
| Theme system     | `data-theme` attribute on `<html>`; preset classes on `<body>`             |
| Desktop          | Tauri 2 (`src-tauri/`, `frontendDist: ../public`)                          |
| Mobile           | Capacitor 7 (`capacitor.config.json`, `android/`, `ios/`)                  |
| Scraper core     | Precompiled IIFE bundle at `public/js/scrapers/bundle.js` (sources private) |
| i18n             | `public/js/i18n/` — 9 locales, English fallback                            |
| Package manager  | npm with `package-lock.json` (lockfile v3)                                 |
| Node.js          | No `engines` field; existing CI used Node 20 (works on Node 22)            |

## 2. Build & development commands (actual, from `package.json`)

| Command                        | Purpose                                   | Status                          |
| ------------------------------ | ----------------------------------------- | ------------------------------- |
| `npm ci` / `npm install`       | Install dependencies                      | ✅ works                        |
| `npm run tauri:dev`            | Desktop dev (Tauri)                       | ⚠️ requires Rust toolchain      |
| `npm run tauri:build`          | Desktop release build                     | ⚠️ requires Rust toolchain      |
| `npm run build:android`        | `cap sync` + Gradle debug APK             | ⚠️ requires JDK 17+ / Android SDK |
| `npm run build:android:release`| `cap sync` + Gradle release APK (unsigned if no keystore) | ⚠️ requires JDK 17+ |
| `npm run build:ios:ipa`        | Unsigned iOS IPA                          | ⚠️ macOS + Xcode only           |
| `npm run build:macos`          | Tauri build + dmg packaging (hardcoded v4.4.0) | ⚠️ macOS + Rust            |
| `npm start`                    | `node server.js`                          | ❌ **broken — `server.js` not in repo** |
| `npm test`                     | `node test/runner.js`                     | ❌ **broken — `test/` is gitignored (private)** |
| `npm run compile:scrapers`     | esbuild scraper bundle                    | ❌ **broken — `scripts/` is gitignored (private)** |

Pre-existing issues (recorded, not caused by this fork's changes):

- `server.js`, `test/`, `scripts/`, `src-scrapers/` are intentionally absent from the public
  repository (see `.gitignore`: "Private Core Scrapers Source & Build Tools"), so
  `start` / `test` / `compile:scrapers` cannot run in a public checkout.
- **There is no lint, type-check, or test infrastructure** in this repository.
  No ESLint, no Prettier, no TypeScript, no stylelint. Any CI can only validate what
  genuinely exists (dependency install, JS/CSS parse/bundle checks, Rust compile check).
- `npm ci` reports 9 known vulnerabilities (1 low, 2 moderate, 3 high, 3 critical) —
  pre-existing dependency state, left untouched.
- There is **no frontend build step**: `public/` is served/generated as-is
  (Tauri `frontendDist: ../public`, Capacitor `webDir: "public"`).

## 3. Validation tooling available

| Check                              | How                                                                 |
| ---------------------------------- | ------------------------------------------------------------------- |
| JS module graph + syntax           | `npx esbuild public/js/app.js --bundle --outfile=/dev/null --format=esm` (esbuild is already a devDependency); same for `share.js` |
| CSS parsing + `@import` resolution | `npx esbuild public/css/style.css --bundle --outfile=/dev/null`      |
| Scraper bundle present             | `test -s public/js/scrapers/bundle.js`                              |
| Rust/Tauri compile check           | `cargo check --manifest-path src-tauri/Cargo.toml` (needs webkit2gtk dev libs on Linux) |

Baseline (pre-change) results: **all four pass** (Node 22.22.3, npm 10.9.8).

## 4. Platform support

| Platform            | In repo                | Shipped on Releases today           | Buildable on GitHub-hosted runners |
| ------------------- | ---------------------- | ----------------------------------- | ---------------------------------- |
| Windows (Tauri)     | ✅                     | ✅ `.exe` (NSIS) + `.msi`           | ✅ `windows-latest`                |
| macOS (Tauri)       | ✅                     | ✅ `.dmg` + `.app.tar.gz` (arm64)   | ✅ `macos-latest`                  |
| Android (Capacitor) | ✅                     | ✅ `.apk` (local builds only today) | ✅ `ubuntu-latest` (JDK 17)        |
| iOS (Capacitor)     | ✅                     | ✅ unsigned `.ipa` (existing CI)    | ✅ `macos-latest` (signing later)  |
| Linux (Tauri)       | ✅ (Tauri supports it) | ❌ not shipped                      | possible — **out of scope this iteration** |
| Plain web           | ✅ static files        | landing page only (Vercel)          | n/a                                |

This iteration's release matrix (agreed): **Windows + macOS + Android (unsigned APK)**.
iOS/Linux excluded for now; private native code (`android/app/src/main/cpp/`, `src-tauri/cpp/`)
is gitignored but guarded conditionally by the build files, and prebuilt `jniLibs` are
committed, so CI builds do not need the private sources.

## 5. UI structure (where the redesign lives)

```
public/
├── index.html          # single page: homePage, historyPage, settingsPage (+7 sub-pages), modals, bottom-nav
├── share.html          # share-target entry (small)
├── css/
│   ├── style.css       # master @import list (10 lines)
│   ├── variables.css   # design tokens, light/dark themes, glass/corner/font/anim presets
│   ├── base.css        # reset, app layout, header, bottom navigation
│   ├── components.css  # toasts, download-bubble, primary/secondary buttons
│   ├── home.css        # input area, analyze button, media/result card, platform chips, skeleton loader, player
│   ├── history.css     # history list/stats
│   ├── settings.css    # settings lists, selects, toggles, sub-pages
│   ├── modals.css      # modals, action sheets, PIN pad, path picker, batch modal
│   └── rtl.css         # RTL overrides
└── js/
    ├── app.js          # entry, page switching (switchPage), navigation
    ├── ui.js           # small UI helpers
    ├── ui/             # result.js, resultModal.js, downloadBubble.js, nativeDownload.js
    ├── modules/        # download.js, history.js, settings/, modals.js, bgAnimation.js …
    └── utils/          # toast.js, device.js, media.js …
```

Pages: **Home** (URL input → analyze → result card → download list), **History**
(list, stats, edit mode, item modal), **Settings** (main menu + General / Appearance /
Live Background / Storage / Network / Advanced / About sub-pages), plus fixed
**bottom navigation** (3 items) and modal overlays.

Dynamic UI is rendered from JS with **existing class names** (`.dl-item`, `.dbd-item`,
`.history-item`, `.media-card`, `.skeleton-*`, …) — a CSS-only redesign can target them
without touching JS.

Existing theme/appearance machinery (must keep working):

- `data-theme="light|dark"` on `<html>` (set by `modules/settings/appearance.js`)
- `<body>` preset classes: `font-*`, `anim-*`, `glass-*`, `corner-*`, `compact-mode`
- Presets configured from Settings → Appearance; stored in `localStorage`

## 6. Existing CI/CD (pre-change)

- `.github/workflows/build-desktop.yml` — tag/release triggered; Windows + macOS Tauri
  builds; asset renaming; `softprops/action-gh-release@v2`; Node 20; npm cache;
  `swatinem/rust-cache@v2`.
- `.github/workflows/build-mobile.yml` — tag/release triggered; unsigned iOS IPA on
  macOS runner.
- **No validation CI** (no workflow runs on pull requests).
- No lint/test/typecheck jobs exist to run.

Both workflows are superseded by `ci.yml` + `release.yml` in this fork
(useful behavior preserved: asset naming, release upload, caches, Node 20).

## 7. Version sources (must match release tags)

| File                       | Field        | Value at baseline |
| -------------------------- | ------------ | ----------------- |
| `package.json`             | `version`    | `4.4.0`           |
| `src-tauri/tauri.conf.json`| `version`    | `4.4.0`           |
| `android/app/build.gradle` | `versionName`| `4.4.0`           |
| `android/app/build.gradle` | `versionCode`| `20`              |
| `src-tauri/Cargo.toml`     | `version`    | `0.1.0` (not used for bundling) |

Release tags (`vX.Y.Z`) must equal the three `X.Y.Z` sources; the release workflow
validates this and fails if they diverge. Version bumps follow the upstream convention
of a `release: update to vX.Y.Z` commit.

## 8. Environment notes (this development machine)

- Node 22.22.3 / npm 10.9.8 available.
- **No Rust toolchain, no Java/JDK** installed → desktop and Android builds cannot be
  verified locally; they are verified through GitHub Actions CI runs.
- `upstream` remote configured (fetch-only); never push to it.

---

## 9. Final verification (post-implementation)

### Branch layout (nothing merged to `main`)

| Branch                 | Base     | Commits | Contents |
| ---------------------- | -------- | ------- | -------- |
| `feat/neumorphism-ui`  | `b4ddad2`| 6       | baseline doc, neumorphism design layer, label relabels, a11y fixes, review fixes, design docs |
| `feat/github-actions`  | `b4ddad2`| 3       | `ci.yml` + `release.yml` (old workflows removed), `docs/CI.md`, `docs/RELEASE.md` |
| `main`                 | —        | untouched | `b4ddad2 release: update to v4.4.0` |

### Validation results — all pass

| Check | Result |
| ----- | ------ |
| `npx esbuild public/css/style.css --bundle` | ✅ (~97 kb, parses + resolves all `@import`s) |
| `npx esbuild public/js/app.js --bundle --format=esm` | ✅ |
| `npx esbuild public/js/share.js --bundle --format=esm` | ✅ |
| `test -s public/js/scrapers/bundle.js` | ✅ (82391 bytes) |
| Selector audit — every class/id in `neumorphism.css` exists in markup/JS | ✅ 0 missing |
| Undefined `var(--…)` references across the design layer | ✅ 0 |
| jsdom boot smoke, light theme | ✅ theme `light`, preset classes applied, Elevation labels, 0 errors |
| jsdom boot smoke, dark theme | ✅ theme `dark`, `glass-deep`, accent `#fffbf2`, 0 errors |
| `npx --yes js-yaml .github/workflows/*.yml` | ✅ both files parse (invalid YAML exits 1 — verified) |
| `actionlint v1.7.7` on both workflows | ✅ 0 findings |
| WCAG contrast sweep (palette, tints, toggle, danger text, badges) | ✅ all ≥ 4.5:1 text / ≥ 3:1 controls |
| Independent design-layer review (29 findings) | ✅ all addressed (2 intentional keeps: `.neo-*` primitives, defensive modal rule) |

### Environment limits (unchanged)

- No Rust/Java/GUI here → Tauri/Android builds and browser screenshots are
  validated by the GitHub Actions runs after these branches are pushed.
- jsdom harness lives outside the repo (`/tmp/opencode/uicheck/boot.js`);
  commands to recreate it are in `UI_DESIGN.md`.

### Next steps for the maintainer

1. Review the two branches; merge `feat/github-actions` and
   `feat/neumorphism-ui` into `main` deliberately (in either order — they
   touch disjoint files; both add root-level markdown only).
2. Push to exercise `ci.yml` (PR/push to `main`) and `release.yml`
   (push a `v*` tag) on real runners.
3. On the next version bump, update the three version sources listed in §7
   together — `release.yml` fails fast if they diverge from the tag.


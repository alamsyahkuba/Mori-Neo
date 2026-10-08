# CI

Workflow: [`.github/workflows/ci.yml`](../.github/workflows/ci.yml)

## When it runs

| Trigger             | Ref                          |
| ------------------- | ---------------------------- |
| `push`              | `main`                       |
| `pull_request`      | targets `main`               |
| `workflow_dispatch` | manual, any checked-out ref  |

Concurrent runs on the same ref are cancelled (`ci-${{ github.ref }}`).

## What it checks

All checks are real commands that also run locally:

1. **Workflow YAML syntax** — `npx --yes js-yaml .github/workflows/*.yml`
   (exits non-zero on invalid YAML).
2. **Install** — `npm ci` (lockfile-exact install).
3. **Frontend bundles** — esbuild parses/bundles the full import graph:
   - `npx esbuild public/js/app.js --bundle --outfile=/dev/null --format=esm`
   - `npx esbuild public/js/share.js --bundle --outfile=/dev/null --format=esm`
   - `npx esbuild public/css/style.css --bundle --outfile=/dev/null`
4. **Scraper bundle** — `test -s public/js/scrapers/bundle.js` (the bundled
   scraper artifact must be present and non-empty).
5. **Tauri backend** — `cargo check --manifest-path src-tauri/Cargo.toml`
   on `ubuntu-latest` after installing the WebKitGTK/AppIndicator system
   packages, using the stable Rust toolchain with `swatinem/rust-cache`.

## What it deliberately does not run

There is no lint, typecheck, or unit-test command in this repository
(`npm test` / `npm start` depend on files excluded from version control),
so CI does not pretend otherwise. When such tooling is added upstream,
add a matching step here.

## Local equivalents

```bash
npm ci
npx --yes js-yaml .github/workflows/ci.yml
npx esbuild public/js/app.js --bundle --outfile=/dev/null --format=esm
npx esbuild public/js/share.js --bundle --outfile=/dev/null --format=esm
npx esbuild public/css/style.css --bundle --outfile=/dev/null
test -s public/js/scrapers/bundle.js
cargo check --manifest-path src-tauri/Cargo.toml   # needs Tauri system deps
```

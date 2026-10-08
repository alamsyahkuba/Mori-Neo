# Release

Workflow: [`.github/workflows/release.yml`](../.github/workflows/release.yml)

Replaces the former `build-desktop.yml` + `build-mobile.yml`.

## When it runs

| Trigger             | Notes                                              |
| ------------------- | -------------------------------------------------- |
| `push` of tag `v*`  | primary path                                       |
| `release: published`| publishing a release also triggers a build         |
| `workflow_dispatch` | manual build; version/tag check is skipped         |

## Release procedure

1. Bump the version in **all three places** so they agree:

   | File                      | Field                        |
   | ------------------------- | ---------------------------- |
   | `package.json`            | `"version"`                  |
   | `src-tauri/tauri.conf.json` | `"version"`                |
   | `android/app/build.gradle`  | `versionName "X.Y.Z"`      |

   (`versionCode` in `android/app/build.gradle` must also be incremented for
   Android; `src-tauri/Cargo.toml`'s crate version is inert — Tauri uses
   `tauri.conf.json`.)

   Following upstream convention, the bump is committed as
   `release: update to vX.Y.Z`.

2. Tag and push (or create a GitHub release for the tag):

   ```bash
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```

3. `verify-version` fails the whole workflow fast if the tag does not equal
   the three version strings above.

## Jobs

### `build-desktop` (matrix: `windows-latest`, `macos-latest`)

Preserves the upstream build and asset-naming behavior exactly:

- Node 20 + npm cache, stable Rust + `swatinem/rust-cache`, `npm ci`,
  `npx tauri build`.
- Windows assets:
  - `Mori-v{VERSION}-Windows-x64-Setup.exe` (NSIS)
  - `Mori-v{VERSION}-Windows-x64.msi` (MSI)
- macOS assets (arm64 runner):
  - `Mori-v{VERSION}-macOS-arm64.dmg` (manual `hdiutil` packaging with
    `src-tauri/README.txt` + `/Applications` symlink)
  - `Mori-v{VERSION}-macOS-arm64.app.tar.gz`
- Assets are attached with `softprops/action-gh-release@v2` (only when the
  ref is a tag).

### `build-android` (`ubuntu-latest`)

- Node 20, Temurin JDK 17 (AGP 8.x), Gradle cache via `setup-java`.
- `npx cap sync android` then `./gradlew assembleRelease --no-daemon`.
- **No signing**: `android/app/build.gradle` only applies a signing config
  when `release.keystore` exists, which it does not in CI — the APK stays
  unsigned.
- The output is renamed to `Mori-v{VERSION}-android-unsigned.apk`, uploaded
  as the `Mori-Android-Unsigned-APK` workflow artifact, and attached to the
  release when the ref is a tag.

## Scope notes

- Platforms: **Windows, macOS, Android** — Linux and iOS are intentionally
  out of scope for this workflow set (the old iOS job was removed).
- No signing secrets or notarization are configured in this iteration; add
  them via repository secrets before enabling signed builds.

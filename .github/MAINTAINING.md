# Maintaining

## Release

1. **Source APK** — the original YouTube APK (arm64, full APK) is attached to the draft release `source`. Replace it when a new YouTube version is supported. Do not delete or publish this draft.
2. **Actions → Release YT Morphe → Run workflow**
   - `rlx`: `release` (latest RLX Patches release), a release tag such as `v2.4.6`, or a branch of RLX Patches (test builds only)
   - `publish`: unchecked builds a test APK (Original_BBK only) into the run's *Artifacts* only
   - `update_preview` (test builds): the APK offers the latest release, to see the update dialog
   - `r2_test` (test builds, with `update_preview`): the APK and its update info are uploaded to R2 as test files (`apk/test_*`, `updates/*-test.json`) and the APK checks those; the next published build removes them

The changelog of a published release is generated again, without a new build, with **Actions → Update release notes** (`tag`: empty for the latest release): the files table stays, the in-app changelog is replaced if the update info is still the one of that release.

Old releases are removed with **Actions → Remove old releases** (`keep`: the newest published releases to keep; drafts such as `source` are never removed).

Every published build becomes the latest release. The workflow builds [RLX Patches](https://github.com/Ramik4/rlx-patches) from source with the additions in `extras/` (update checker, experimental notice), patches the source APK with the patches RLX Patches enables by default, verifies package name and signature, uploads the two APKs to R2, publishes the release (links to the APKs on R2) and updates `updates/*.json` for the in-app update check. RLX Patches already has the settings, theme colors, icons, voice-over translation and Save to device, so the mod adds none of these.

| Variant | File | Package | Theme colors |
|---|---|---|---|
| `morphe` | `YT_Morphe-v<YouTube>_RLX-v<RLX>_Amoled.apk` | `app.morphe.android.youtube` | All in-app themes, Pure Black / White by default (Theme option `inAppDefaultColors`: `pure_black`) |
| `hiby` | `YT_Morphe-v<YouTube>_RLX-v<RLX>_Original_BBK.apk` | `com.hiby.music` (Clone app) | All in-app themes, Modern Dark #0F0F0F / Soft Light #F0F0F0 by default (`rlx`) |

Test builds add the build number: `..._RLX-v<RLX>-b<build>_Original_BBK.apk`. Release tags are `v<YouTube>-rlx<RLX>-p<Morphe>-b<build>` (earlier releases: `v<YouTube>-p<Morphe>-b<build>`).

The changelog is generated from the Morphe changelog included in RLX Patches (YouTube only), for the Morphe version RLX Patches is based on, newest first: a stable version lists the new features of its own section; a dev version lists the new features of all dev versions since the last one released here. Morphe features that do not apply to the mod are listed in `notes/skipped-features.md` and left out. Bug fixes are not listed one by one: the notes always end with the heading `🔧 Other improvements`, with the important changes of the mod under it from `## <Morphe version>` in `notes/improvements.md` (most releases have none, then the heading stands alone). The release notes start with the reminder to install the downloaded APK; the app shows its own note in the app language, and the two headings in the app language too. Only when a release skips versions (such as the first dev build after a stable one), important features of the skipped versions can be added under `## <Morphe version>` in `notes/extra-features.md`; they are listed after the Morphe new features of that version. The release notes and the in-app dialog show only two headings, `✨ New features` and `🔧 Other improvements`; the release notes then add the RLX Patches line and the files. Release notes use `##` headings (GitHub draws a thin rule under them), the in-app dialog shows each title bold with a thin rule under it. Optional translations for the in-app dialog: `changelog/<language>.md` whose first line is `<!-- <Morphe version> -->`.

## 4PDA post

`notes/4pda-post.txt` is the post for the 4PDA topic, updated for each release (YouTube and patches versions, What's new, links).

- An empty line after each `[spoiler=...]` title
- No "test version" note next to dev patches versions, users could misread it
- What's new is a plain list, without a line for the YouTube and patches versions (such as "YouTube 21.44.528, Morphe Patches 2.1.0 (стабильная версия)"). A stable version lists its full changelog, the best YouTube features first; a dev version lists the new features of all dev versions since the last release
- Link titles in `[color="#808080"]`, followed by the file size
- Section order: header, What's new, Features, Download (Amoled, then Original), Mod info, Help (MicroG RE, playback fixes)

## Configuration

| File | Purpose |
|---|---|
| `addons/rlx/` | Changes of the mod to RLX Patches files (`git apply` patches, applied in name order; skipped once RLX Patches includes them, the build fails if one no longer applies): `settings-top-screens.patch` (Updates right below About) and `theme-default-colors.patch` (the Theme option `inAppDefaultColors`), both in RLX Patches after 2.4.5 |
| `extras/` | Additional patches: update check and experimental notice. Strings of `extras/strings/<locale>/` are appended to the YouTube ones (the build fails if RLX Patches already has one of them). The update checker also turns off the patches update check of the RLX about page |

Variant options (package, theme colors) are set in `build_variant` calls of `.github/workflows/release.yml`. Settings, icons and theme defaults come from RLX Patches (`rlx/defaults.json` there).

## Downloads (Cloudflare R2)

Published builds are uploaded to the R2 bucket `R2_BUCKET` (the variable `DL_BASE_URL` is required to publish; the APKs are not attached to the GitHub release): the APKs to `apk/`, the update info to `updates/<variant>.json`, and for builds on a stable Morphe version also to `updates/<variant>-stable.json` (the update channel *Stable versions only* of the app checks that file; until the first stable build it does not exist, and the app finds no update). The apps check `DL_BASE_URL/updates/<variant>.json` and download the APK from `DL_BASE_URL/apk/`, so they no longer depend on GitHub. The APKs of the last 3 builds are kept. `cloudflare/worker.js` is the Worker (`cdn`) that serves the bucket (binding `BUCKET`, read only). `updates/*.json` in this repository stays up to date for builds that still check GitHub.

With the optional D1 binding `STATS` (database `stats`), the Worker counts per day the APK downloads per file and the update checks per variant, which is about the number of installs in use; only the path, the day and a count are stored. The page `DL_BASE_URL/stats?key=<STATS_KEY>` shows the counts, once the secret `STATS_KEY` is set on the Worker (Settings → Variables and Secrets); without it the page does not exist. Ready-made queries for the D1 console: `cloudflare/stats.sql`.

The update address is built into each APK: `DL_BASE_URL` must not change, or the builds that use it stop getting updates.

## Secrets (Settings → Secrets and variables → Actions)

| Name | Type | Value |
|---|---|---|
| `KEYSTORE_B64` | secret | Keystore, base64 |
| `KEYSTORE_PASSWORD` | secret | Keystore password (empty: Morphe default) |
| `KEY_ALIAS` | secret | Key alias (empty: `Morphe`) |
| `KEY_PASSWORD` | secret | Key password (empty: `Morphe`) |
| `CERT_SHA256` | variable | SHA-256 fingerprint of the signing certificate |
| `R2_ACCOUNT_ID` | secret | Cloudflare account ID |
| `R2_ACCESS_KEY_ID` | secret | R2 API token: access key ID (Object Read & Write, bucket only) |
| `R2_SECRET_ACCESS_KEY` | secret | R2 API token: secret access key |
| `R2_BUCKET` | variable | R2 bucket name |
| `DL_BASE_URL` | variable | Address of the Worker, without a trailing slash |

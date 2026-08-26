# Independent Me — Google Play submission pack (rev. 26 Aug 2026)

Supersedes the 22 Aug "Clickeasy" pack. The build is done; what remains is store
listing paperwork.

## Fixed facts — do not change these

| Thing | Value |
|---|---|
| App name | **Independent Me** |
| Package name | `ie.clickeasy.independentme` — **permanent**, already uploaded |
| Source repo | github.com/anchoill1/independent-me (**public**) |
| Capacitor project | `clickeasy_app/independentme-cap/` |
| Web source of truth | `clickeasy_app/independentme.html` → copied to `independentme-cap/www/index.html` |
| Developer account | **Organization** — the 12-testers-for-14-days rule does NOT apply |
| Current version | `versionCode 2`, `versionName 1.0.1` |
| Privacy policy | https://anchoill1.github.io/independent-me/ (GitHub Pages, repo must stay public) |

## Where things stand

**Done:** Capacitor project rebuilt, targetSdk 36 confirmed, keystore created,
Play App Signing enrolled, app created in Play Console, internal testing track
live, sound bug found and fixed, privacy policy hosted.

**Remaining:** rebranded icon and feature graphic, screenshots, listing copy,
the four questionnaires, payments profile, price.

## Clocks

| Clock | Length | Starts when |
|---|---|---|
| API 36 requirement | hard cutoff **31 Aug 2026** | n/a — build already targets 36 |
| Families / target-audience review | 7 days or longer | submission |
| Payments profile verification | days, not instant | you submit bank + tax details |

---

## The two build loops — keep them straight

This caused real confusion and cost an afternoon. There are two completely
separate ways to build, and they are not interchangeable.

**Testing on your own device — green ▶ in Android Studio.**
Compiles a debug APK, installs straight over USB, takes seconds. No signing, no
version bump, never touches Play. Use this for everything except uploads.

**Uploading to Play — Build → Generate Signed App Bundle.**
Slower, needs the keystore password, and needs `versionCode` incremented **every
single time** or Play rejects it.

Two traps inside that:

- `./gradlew bundleRelease` produces an **UNSIGNED** bundle at
  `android/app/build/outputs/bundle/release/`. Play refuses it with "All uploaded
  bundles must be signed." The Studio wizard writes to a **different** path,
  `android/app/release/app-release.aab`. Same filename, different folder — check
  the timestamp, not the name.
- A debug build and a Play build **cannot coexist on one device**. Different
  signatures, so installing one requires uninstalling the other, which wipes app
  data. Export via Settings → Backup & Transfer first if it matters.

```bash
# confirm you are about to upload a FRESH bundle, not yesterday's
ls -lT android/app/release/app-release.aab
```

## Shipping a change

```bash
# 1. edit independentme.html, then:
cd independentme-cap && cp ../independentme.html www/index.html && npx cap copy android

# 2. bump the version (edit android/app/build.gradle, or:)
sed -i '' 's/versionCode 2/versionCode 3/; s/versionName "1.0.1"/versionName "1.0.2"/' android/app/build.gradle

# 3. Android Studio → Build → Generate Signed App Bundle → release
# 4. upload to Play Console → Internal testing → Create new release

# 5. commit
cd .. && git add -A && git commit -m "message" && git push
```

If Android Studio has `build.gradle` open, click **Reload** when the "File
changed on disk" banner appears, or it will write back the old version numbers.

---

## Step A — Rebranding debt (longest lead time, start here)

Every store asset still says Clickeasy:

| Asset | Status |
|---|---|
| `clickeasy-icon-512.png` | **redo** — 512×512 store icon |
| `clickeasy-icon-1024.png` | **redo** — source for the launcher icon |
| `clickeasy-feature-graphic-1024x500.png` | **redo** — slowest item, blocks the listing |
| `clickeasy-play-store-listing.md` | **rewrite** — name, short and full description |
| Privacy policy contact | says `joe@clickeasy.ie`; site and app use `info@clickeasy.ie` — make consistent |

Also confirm nothing user-visible still says Clickeasy:

```bash
grep -nE '<title>|<h1' independentme.html
```

Internal identifiers (`clickeasyExportPayload`, `clickeasy_photos` IndexedDB
store, the `app:'clickeasy'` marker in backups) must **stay** — renaming them
breaks Backup & Transfer for every existing export, with no migration path.

## Step B — Screenshots

2–8, phone portrait, captured off the current build: a routine in progress, the
celebration screen, the home list, the stars calendar, the step editor.

## Step C — Questionnaires

- **Data safety:** no data collected. Backup & Transfer is on-device only.
- **Target audience:** children bands + 18 and over → triggers the Families review.
- **Content rating:** all clean → 3+.
- **Ads:** none. This is what keeps Families compliance simple.
- **App access:** no login. The in-app PIN is local, not an account credential.
- **Category:** Education / Parenting.
- **Privacy policy URL:** as above.

Google checks the declared audience against the app's own imagery and language.
With stars and celebrations it reads as a children's app whatever you tick, so
declare it honestly rather than trying to dodge the review.

## Step D — Selling it

Two irreversible things, both before the **first production release**:

1. **Payments profile verified.** Bank and tax details; not instant. Without it
   the price field stays greyed out.
2. **Price set.** A free app can never be changed to paid. Paid → free works
   once, and is also one-way. For a free introductory period, launch paid and
   hand out promo codes.

---

## Testing checklist before promoting to production

Install from the internal testing link (not a debug build) and confirm:

- Opens and works fully offline with aeroplane mode on.
- **Step read-aloud speaks.** Tap the same step again → stops. Tap a different
  step mid-sentence → switches cleanly. Leave the routine → goes quiet.
- Backup & Transfer exports on one device and imports on another.
- The hardware back button doesn't dump users out mid-routine.
- Data survives a force-quit and a device restart.

The speech checks matter most. Android's WebView has **no Web Speech API at all**
— `speechSynthesis` is undefined, not empty — so read-aloud failed silently once
the app was wrapped. It now routes through `@capacitor-community/text-to-speech`
via a shim in `independentme.html` that falls back to `speechSynthesis` when the
file is opened in a desktop browser. Anything that adds speech must go through
that shim, never `speechSynthesis` directly.

## Device debugging notes

- `chrome://inspect/#devices` in Chrome on the Mac, with the device plugged in,
  unlocked, and the app in the foreground. `location.href` should be
  `https://localhost/`.
- **Samsung Auto Blocker silently blocks adb.** Settings → Security and privacy →
  Auto Blocker → off (or just its "Block USB commands" toggle). No cable swap
  will fix it. Turn it back on afterwards.
- Samsung also refuses to change security settings from a floating or split-screen
  window. Close all recents and open Settings from the app drawer.
- Enabling developer options needs the device's screen-lock PIN. There is no way
  round it.

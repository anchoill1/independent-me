# Clickeasy — Google Play submission pack (rev. 22 Aug 2026)

Supersedes the June version. What changed and why:

- The old `clickeasy-android/` folder was **not a Capacitor project** — no `variables.gradle`,
  no `capacitor.settings.gradle`, no `node_modules`. The old step 1 ("run `npx cap copy android`")
  could never have worked against it. Rebuilding from scratch instead.
- No keystore was ever generated (`keystore.properties.template` is still a template), and
  nothing was ever uploaded to Play. So there is no signing key, package name or versionCode
  to preserve. Clean slate.
- New hard deadline: **31 Aug 2026**, target API 36. Applies to every track, testing included.
- Declaring a children's age band puts this under the **Families policy**, which adds an
  extended review on top of everything else. Budget for it.

## Deadlines and clocks (the actual critical path)

Account type confirmed **Organization** (22 Aug 2026). The 12-testers-for-14-days rule does
**not** apply — that is personal accounts only. You can publish straight to production.

| Clock | Length | Starts when |
|---|---|---|
| API 36 requirement | hard cutoff **31 Aug 2026** | n/a — build against 36 now |
| Families / target-audience review | 7 days or longer | submission |
| Payments profile verification | days, not instant | you submit bank + tax details |

Only two things now gate the launch: getting a compliant .aab uploaded before 31 Aug, and the
Families review. Everything else is paperwork you control.

## Step 1 — Confirm which HTML is actually current

The pack was written in June and the product has been upgraded since. `clickeasy-v38.html`
and `clickeasy-v38old.html` both exist. Confirm which file is the newest build before it gets
baked into an .aab — shipping the wrong one to testers wastes a fortnight.

```bash
ls -lt clickeasy-v38*.html
```

## Step 2 — Rebuild as a real Capacitor project

```bash
mkdir -p clickeasy-cap/www && cd clickeasy-cap && npm init -y \
  && cp ../clickeasy-v38.html www/index.html \
  && npm i @capacitor/core @capacitor/android @capacitor/app && npm i -D @capacitor/cli \
  && npx cap init "Clickeasy" ie.clickeasy.app --web-dir=www && npx cap add android
```

The package name `ie.clickeasy.app` is **permanent** once published. Choose it deliberately.

Then confirm `android/variables.gradle` reads:

```
minSdkVersion = 24
compileSdkVersion = 36
targetSdkVersion = 36
```

Capacitor 8 needs Android Studio Otter (2025.2.1) or newer. If Studio is older, update it
first — Gradle sync will fail in confusing ways otherwise.

### Before building, check v38 for network dependencies

Anything loaded from a CDN — a font, an icon set, a chart library — works on your Mac and
fails silently on a phone in aeroplane mode. For a genuinely offline app every asset must be
inline or inside `www/`.

```bash
grep -nE 'https?://' www/index.html
```

Also verify Backup & Transfer still round-trips inside the WebView: export on one device,
import on another, confirm a routine comes back. WebView storage behaves differently from
desktop Chrome, and this is the feature most likely to break in the move.

## Step 3 — Keystore (do this once, then back it up)

```bash
keytool -genkey -v -keystore clickeasy-upload.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Fill in `keystore.properties` from the template. Then:

- Back the `.jks` and its passwords up somewhere that is not this laptop.
- Add `*.jks` and `keystore.properties` to `.gitignore` **before the first commit.**
  A signing key pushed to a repo is unrecoverable.
- Enrol in Play App Signing at first upload. Google then holds the real key and your upload
  key becomes resettable via support — which converts "lost the key, app is dead" into a
  support ticket.

## Step 4 — Build the bundle

```bash
npx cap copy android && cd android && ./gradlew bundleRelease
```

Output: `android/app/build/outputs/bundle/release/app-release.aab`

Every subsequent upload needs a **higher `versionCode`** in `android/app/build.gradle`.
Play rejects any bundle that doesn't strictly increase it. This is the single most common
update rejection.

## Step 5 — Internal testing first, then production

No tester quota applies, but do not skip straight to production. Push the .aab to the
**internal testing** track first — it goes live to your own devices in minutes, with no review,
and it is the only realistic way to catch WebView-specific breakage before strangers see it.

Test on at least two real devices, ideally one phone and one tablet, and specifically confirm:

- The app opens offline with aeroplane mode on.
- Backup & Transfer exports *and* re-imports (see Step 2).
- The Android hardware back button doesn't dump users out of the app mid-routine.
- Data survives a force-quit and a device restart.

Then promote the same build to production. Screenshots (Step 6) can be captured off the
internal build while you're at it.

## Step 6 — Store listing (reuse from the June pack, all still valid)

| Field | Source file |
|---|---|
| App icon (512×512) | `clickeasy-icon-512.png` |
| Feature graphic (1024×500) | `clickeasy-feature-graphic-1024x500.png` |
| Launcher icon | `clickeasy-icon-1024.png` → Android Studio → Image Asset, resize ~80% |
| Name, short + full description | `clickeasy-play-store-listing.md` |
| Privacy policy URL | `clickeasy-privacy-policy.html` — replace `[YOUR CONTACT EMAIL]`, host on GitHub Pages, paste URL |

**Screenshots — still outstanding.** 2–8, phone portrait: a routine in progress, the
celebration/stars screen, the home list, the stars calendar, the step editor. Capture from a
real device or emulator once the .aab runs.

## Step 7 — Console questionnaires

- **Data safety:** no data collected. Backup & Transfer is on-device only and does not change
  this answer — nothing is transmitted off the device.
- **Target audience:** children bands + 18 and over. This triggers the Families policy.
- **Content rating:** all clean → 3+.
- **Ads:** none. This is what makes Families compliance straightforward — no certified ad SDK
  requirement, no neutral age screen needed.
- **App access:** no login, so nothing to declare. (The in-app PIN is local and is not an
  account credential.)
- **Category:** Education / Parenting.

## Step 8 — Selling it

Two irreversible things, both of which must happen *before* the first production release:

1. **Complete the payments profile.** Bank and tax details, and verification is not instant.
   Without it the price field stays greyed out.
2. **Set the price.** A free app can never be changed to paid after publishing. Paid → free
   works once, and is also one-way.

Note that a paid, fully offline app is trivially extractable from the .aab. For a low-priced
utility this usually isn't worth defending against; the Play Integrity API is the mechanism
if it turns out to matter.

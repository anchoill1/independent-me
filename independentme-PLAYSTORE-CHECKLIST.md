# IndependentMe — Google Play submission pack (rev. 22 Aug 2026)

Supersedes the June "Clickeasy" pack. What changed and why:

- **App renamed** Clickeasy → IndependentMe. Source file is now `independentme.html`.
  The rename affects the store assets, not just the HTML — see "Rebranding debt" below.
- The old `clickeasy-android/` folder was **not a Capacitor project** — no `variables.gradle`,
  no `capacitor.settings.gradle`, no `node_modules`. The old step 1 ("run `npx cap copy android`")
  could never have worked against it. Rebuilding from scratch instead.
- No keystore was ever generated (`keystore.properties.template` is still a template), and
  nothing was ever uploaded to Play. No signing key, package name or versionCode to preserve.
- New hard deadline: **31 Aug 2026**, target API 36. Applies to every track.
- Account type confirmed **Organization**, so the 12-testers-for-14-days rule does not apply.
- Declaring a children's age band puts this under the **Families policy** and its extended review.

## Deadlines and clocks (the actual critical path)

| Clock | Length | Starts when |
|---|---|---|
| API 36 requirement | hard cutoff **31 Aug 2026** | n/a — build against 36 now |
| Families / target-audience review | 7 days or longer | submission |
| Payments profile verification | days, not instant | you submit bank + tax details |

Only two things gate the launch: a compliant .aab uploaded before 31 Aug, and the Families
review. Everything else is paperwork you control.

## Step 0 — Rebranding debt (do this first, it has the longest lead time)

The store assets in this folder still say Clickeasy. Each needs redoing before submission:

| Asset | Status | Notes |
|---|---|---|
| `clickeasy-icon-512.png` | **redo** | If the mark contains the word "Clickeasy", it must be regenerated |
| `clickeasy-icon-1024.png` | **redo** | Source for the launcher icon |
| `clickeasy-feature-graphic-1024x500.png` | **redo** | Almost certainly has the old name rendered in — longest lead time of the four |
| `clickeasy-play-store-listing.md` | **rewrite** | Name, short description, full description, keywords all reference the old brand |
| `clickeasy-privacy-policy.html` | **edit** | Must name the app and the legal entity correctly, plus the contact email |

Also check inside the HTML itself: page `<title>`, any heading or splash text, and the
`manifest.webmanifest` `name` / `short_name`. A store listing that says IndependentMe over an
app that says Clickeasy on launch is a target-audience/misrepresentation flag during review.

```bash
grep -ni 'clickeasy' independentme.html manifest.webmanifest
```

## Step 1 — Confirm which HTML is actually current

The June pack referenced `clickeasy-v38.html`, and `clickeasy-v38old.html` also exists.
Confirm `independentme.html` is genuinely the newest build before it gets baked into an .aab —
shipping the wrong file wastes a review cycle.

```bash
ls -lt *.html
```

## Step 2 — Rebuild as a real Capacitor project

```bash
mkdir -p independentme-cap/www && cd independentme-cap && npm init -y \
  && cp ../independentme.html www/index.html \
  && npm i @capacitor/core @capacitor/android @capacitor/app && npm i -D @capacitor/cli \
  && npx cap init "IndependentMe" ie.clickeasy.independentme --web-dir=www && npx cap add android
```

**Choose the package name deliberately — it is permanent once published and cannot be changed.**
`ie.clickeasy.independentme` assumes Clickeasy is the company/domain and IndependentMe is the
product, which leaves room for a second app later. If IndependentMe is itself the business,
use `ie.independentme.app` instead. Decide now.

Then confirm `android/variables.gradle` reads:

```
minSdkVersion = 24
compileSdkVersion = 36
targetSdkVersion = 36
```

Capacitor 8 needs Android Studio Otter (2025.2.1) or newer. If Studio is older, update it
first — Gradle sync will fail in confusing ways otherwise.

### Before building, check for network dependencies

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
keytool -genkey -v -keystore independentme-upload.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Fill in `keystore.properties` from the template. Then:

- Back the `.jks` and its passwords up somewhere that is not this laptop.
- Add `*.jks` and `keystore.properties` to `.gitignore` **before the first commit.**
  A signing key pushed to a repo is unrecoverable.
- Enrol in Play App Signing at first upload. Google then holds the real key and your upload
  key becomes resettable via support — which turns "lost the key, app is dead" into a
  support ticket.

## Step 4 — Build the bundle

```bash
npx cap copy android && cd android && ./gradlew bundleRelease
```

Output: `android/app/build/outputs/bundle/release/app-release.aab`

Every subsequent upload needs a **higher `versionCode`** in `android/app/build.gradle`.
Play rejects any bundle that doesn't strictly increase it. This is the most common update
rejection.

## Step 5 — Internal testing first, then production

No tester quota applies to an Organization account, but don't go straight to production.
Push the .aab to the **internal testing** track first — live on your own devices in minutes,
no review — and confirm on at least two real devices, ideally a phone and a tablet:

- Opens and works fully offline with aeroplane mode on.
- Backup & Transfer exports *and* re-imports (see Step 2).
- The Android hardware back button doesn't dump users out mid-routine.
- Data survives a force-quit and a device restart.

Then promote the same build to production. Capture screenshots off the internal build.

## Step 6 — Store listing

| Field | Source |
|---|---|
| App icon (512×512) | rebranded icon — see Step 0 |
| Feature graphic (1024×500) | rebranded graphic — see Step 0 |
| Launcher icon | 1024px source → Android Studio → Image Asset, resize ~80% |
| Name, short + full description | rewritten listing copy |
| Privacy policy URL | rebranded policy, hosted on GitHub Pages, URL pasted into App content |

**Screenshots — 2–8, phone portrait.** A routine in progress, the celebration/stars screen,
the home list, the stars calendar, the step editor.

## Step 7 — Console questionnaires

- **Data safety:** no data collected. Backup & Transfer is on-device only and does not change
  this — nothing is transmitted off the device.
- **Target audience:** children bands + 18 and over. This triggers the Families policy.
- **Content rating:** all clean → 3+.
- **Ads:** none. This is what makes Families compliance straightforward — no certified ad SDK
  requirement, no neutral age screen needed.
- **App access:** no login, nothing to declare. (The in-app PIN is local, not an account credential.)
- **Category:** Education / Parenting.

Google reviews the declared audience against the app's own imagery and language, so declare
the children's band honestly rather than trying to avoid the Families review.

## Step 8 — Selling it

Two irreversible things, both of which must happen *before* the first production release:

1. **Complete the payments profile.** Bank and tax details; verification is not instant.
   Without it the price field stays greyed out.
2. **Set the price.** A free app can never be changed to paid. Paid → free works once, and is
   also one-way. If you want a free introductory period, launch paid and hand out promo codes
   rather than launching free.

A paid, fully offline app is trivially extractable from the .aab. For a low-priced utility this
usually isn't worth defending against; the Play Integrity API is the mechanism if it matters.

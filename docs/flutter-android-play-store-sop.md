# Flutter Android Google Play SOP (Role-Based Guide)

## Table of Contents

- [1. Overview](#1-overview)
- [2. First Time App Release (Initial Launch)](#2-first-time-app-release-initial-launch)
  - [2.1 Purpose](#21-purpose-what-you-are-doing-here)
  - [2.2 Setup Phase](#22-setup-phase-prepare-google-play-side-first)
  - [2.3 Build Preparation](#23-build-preparation-make-app-ready-for-release)
  - [2.4 Store Listing Preparation](#24-store-listing-preparation-what-users-will-see-on-google-play)
  - [2.5 Upload & Submission](#25-upload--submission-send-app-to-google-play)
  - [2.6 Release](#26-release-go-live)
- [3. Follow-Up Release (Update / Version Upgrade)](#3-follow-up-release-update--version-upgrade)
  - [3.1 Purpose](#31-purpose-what-this-is-for)
  - [3.2 Code Update](#32-code-update-do-your-changes-first)
  - [3.3 Version Update](#33-version-update-very-important-step)
  - [3.4 Build & Upload Preparation](#34-build--upload-preparation)
  - [3.5 Submission](#35-submission-send-new-version-to-google-play)
  - [3.6 Release](#36-release-go-live-1)
- [4. Monitoring Phase (Post-Release)](#4-monitoring-phase-post-release)
  - [4.1 Purpose](#41-purpose-what-this-is-for-1)
  - [4.2 Stability Monitoring](#42-stability-monitoring-check-if-app-is-healthy)
  - [4.3 User Feedback](#43-user-feedback-what-users-are-saying)
  - [4.4 Analytics Monitoring](#44-analytics-monitoring-understand-user-behavior)
  - [4.5 Incident Handling](#45-incident-handling-what-to-do-when-something-breaks)
  - [Summary of Monitoring Phase](#summary-of-monitoring-phase)
- [5. Hotfix & Emergency Release Flow](#5-hotfix--emergency-release-flow)
  - [5.1 Purpose](#51-purpose-what-this-is-for-2)
  - [5.2 Identify the Problem](#52-identify-the-problem-confirm-it-is-really-urgent)
  - [5.3 Create Hotfix Branch](#53-create-hotfix-branch-start-fixing-immediately)
  - [5.4 Increase Version](#54-increase-version-required-before-upload)
  - [5.5 Build Hotfix Version](#55-build-hotfix-version)
  - [5.6 Upload Hotfix to Google Play](#56-upload-hotfix-to-google-play)
  - [5.7 Fast Review & Rollout](#57-fast-review--rollout-important-for-emergencies)
  - [5.8 Release Hotfix](#58-release-hotfix)
  - [5.9 Post Hotfix Review](#59-post-hotfix-review-very-important)
  - [Summary of Hotfix Flow](#summary-of-hotfix-flow)

---

## 1. Overview

This document defines a simplified, role-based workflow for publishing and maintaining Flutter Android applications on **Google Play**.

It is structured into four main operational modes:

1. **First Time Release** — initial launch → [Section 2](#2-first-time-app-release-initial-launch)  
2. **Follow-Up Release** — updates / new versions → [Section 3](#3-follow-up-release-update--version-upgrade)  
3. **Monitoring Phase** — post-release operations → [Section 4](#4-monitoring-phase-post-release)  
4. **Hotfix / Emergency Phase** — maintenance when production is broken → [Section 5](#5-hotfix--emergency-release-flow)  

Each section is designed so developers only follow what is relevant to their current task.

---

## 2. First Time App Release (Initial Launch)

### 2.1 Purpose (What you are doing here)

In this stage, you are preparing everything needed to put your Flutter app on **Google Play** for the very first time.

Think of it like this: you are not just building the app — you are also setting up Google’s console, signing, and policies so Play accepts your app.

### 2.2 Setup Phase (Prepare Google Play Side First)

**Step 1: Google Play Console access**

Sign in to [Google Play Console](https://play.google.com/console) and make sure:

- You can access the correct developer account (personal or organization)  
- You have permission to create apps and manage releases (often **Admin** or **Release manager**)  

If you cannot create an app or open **Policy** / **Release** sections, ask your Play Console admin.

**Step 2: Create the app listing**

In Play Console, click **Create app** and complete the initial questionnaire (app name, default language, app / game, free or paid).

You will get an empty **app** with its own Play Console dashboard. The **package name** (application ID) is chosen at creation time and **cannot be changed later** — it must match your Flutter Android project (`applicationId` in `android/app/build.gradle.kts` or `android/app/build.gradle`).

**Step 3: Signing — upload keystore and `key.properties` (where many teams get stuck)**

Google Play needs a **signed** release build. With **Play App Signing** (default), you keep an **upload key**; Google holds the app-signing key that end users receive.

Follow **either** Option A **or** Option B below. When that option is done, continue with **Common steps** in order.

**Rules (always)**

- Never commit `key.properties`, keystore files, or passwords to Git.  
- Back up the keystore and passwords in a team password manager. **Losing the upload key** (without going through Google’s reset process) **can block future updates.**  
- In CI, inject `key.properties` or secrets from your secret store — do not hardcode in the repo.  

**Option A — Use a keystore your team already created**

**Step 1:** Confirm you have all four: the keystore file (`.jks` or `.keystore`), **store password**, **key password**, and **key alias** (e.g. `upload`). If anything is missing, ask whoever owns release signing.

**Step 2:** Copy the keystore into your Flutter project. Pick one location and stay consistent:

- **`android/upload-keystore.jks`** — then in `key.properties` use `storeFile=../upload-keystore.jks` (Gradle resolves this from `android/app/build.gradle`).  
- **`android/app/upload-keystore.jks`** — then use `storeFile=upload-keystore.jks`.

**Step 3:** Create **`android/key.properties`** at the **android** root (next to `settings.gradle`, **not** inside `android/app/`). Example when the file is **`android/upload-keystore.jks`**:

```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

Replace `YOUR_*` and `keyAlias` with your real values. If the keystore lives under **`android/app/`**, use `storeFile=upload-keystore.jks` instead.

Then go to **Common steps** below.

---

**Option B — Create a new upload keystore (first app or new key)**

**Step 1:** Install a **JDK** (or use Android Studio’s bundled JDK) so `keytool` is available in a terminal. On macOS/Linux you can run `keytool -help` to verify.

**Step 2:** Open a terminal, go to your app’s **`android/`** folder, and generate the keystore:

```bash
cd android
keytool -genkey -v -keystore upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Answer the prompts. You choose **keystore password** and **key password** (they may be the same). The **alias** in the command is `upload` unless you change `-alias` — whatever you use must match `keyAlias` in the next steps.

**Step 3:** Store **keystore password**, **key password**, and **alias** in your team password manager. You will need them for every release build.

**Step 4:** Create **`android/key.properties`** at the **android** root (same place as in Option A). Because `upload-keystore.jks` was created inside **`android/`**, use:

```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

If you later move the keystore into **`android/app/`**, change `storeFile` to `upload-keystore.jks` to match.

**Step 5:** (Optional) If your security policy forbids keeping the `.jks` under the repo folder, move it to a secure path and set `storeFile` to an **absolute** path in `key.properties` for local builds; CI should copy or mount the keystore and generate `key.properties` without committing it.

Then go to **Common steps** below.

---

**Common steps (after Option A or Option B)**

**Step 1:** Add secrets to **`android/.gitignore`** so they are never committed:

```gitignore
key.properties
*.jks
*.keystore
```

**Step 2:** Wire **Gradle** to read `key.properties`. Flutter’s [Sign the app](https://docs.flutter.dev/deployment/android#sign-the-app) guide has the full example. For **`android/app/build.gradle`** (Groovy), add **above** `android {`:

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```

Inside **`android {`**, add:

```groovy
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
```

Inside **`buildTypes { release { ... } }`**, set:

```groovy
            signingConfig signingConfigs.release
```

If you use **`build.gradle.kts`**, apply the same pattern with Kotlin DSL (load `Properties` from `rootProject.file("key.properties")` and assign `signingConfigs.getByName("release")`).

**Step 3:** Verify signing end-to-end:

- Run **`flutter build appbundle --release`**. If it fails, re-check `key.properties` location, `storeFile` path (relative to `android/app/`), alias, and passwords.  
- Plan for the **first** Play upload: accept **Play App Signing** when Play Console prompts you, and keep the **upload** keystore backed up.  
- Confirm **`applicationId` / namespace** matches the Play Console package name **exactly**, and that **`minSdk`**, **`targetSdk`**, and **`compileSdk`** meet [Google Play target API requirements](https://developer.android.com/google/play/requirements/target-sdk) (policy updates over time).

**Step 4: (Recommended) Internal testing track first**

Before production, create an **internal testing** release with a small group. This verifies upload signing, installability, and basic flows without exposing the app to the public store.

### 2.3 Build Preparation (Make App Ready for Release)

**Step 1: Switch Flutter to production mode**

In your Flutter project, make sure:

- API URL is production (not staging/dev)  
- Debug prints are removed or disabled  
- No test data is used  
- ProGuard / R8 rules are correct if you use code shrinking (`minifyEnabled true`)  

**Step 2: Clean project (important before build)**

```bash
flutter clean
flutter pub get
```

This removes old build cache.

**Step 3: Open Android project (optional but useful)**

You can open `android/` in Android Studio to sync Gradle and catch configuration errors early:

```bash
# From project root — open the android folder in Android Studio, or:
cd android && ./gradlew :app:assembleRelease --dry-run
```

Check:

- No Gradle sync errors  
- `applicationId`, signing config, and SDK levels are as expected  

**Step 4: Set version number**

Open `pubspec.yaml`:

```yaml
version: 1.0.0+1
```

Simple rule (Flutter maps this to Android):

- `1.0.0` → **versionName** (user-visible on Play)  
- `+1` → **versionCode** (integer; must **increase** for every Play upload)  

**Step 5: Build release App Bundle (preferred on Play)**

```bash
flutter build appbundle --release
```

Output is typically:

`build/app/outputs/bundle/release/app-release.aab`

Google Play’s standard upload format is **AAB** (not APK) for new store listings. Use APK only if you have a specific distribution need outside this SOP.

If the build succeeds, you are ready to upload the `.aab` to Play Console.

### 2.4 Store Listing Preparation (What users will see on Google Play)

This is where you prepare everything visible to Google reviewers and users.

**Step 1: Prepare store listing text**

In Play Console → **Grow users** → **Store presence** → **Main store listing** (paths may shift slightly as the UI updates):

- Short description, full description  
- App icon, feature graphic  
- Support email / URL, optional marketing URL  

**Step 2: Prepare screenshots and graphics**

You need phone (and possibly tablet / large screen) screenshots per [current specs](https://support.google.com/googleplay/android-developer/answer/9866151).

Make sure:

- No broken UI  
- Real app content (not dummy data)  
- Main features are shown clearly  

**Step 3: Privacy, content, and declarations**

Complete required sections honestly, including:

- **Data safety** form (what data is collected / shared)  
- **Privacy policy** URL (usually mandatory)  
- Content rating questionnaire  
- Target audience / ads declarations if applicable  

Missing or inaccurate declarations are a common rejection reason.

### 2.5 Upload & Submission (Send App to Google Play)

**Step 1: Choose a release track**

For the first public launch you still typically:

- Upload to **Closed testing** or **Open testing** first (recommended), then promote to **Production**, **or**  
- Go straight to **Production** if your organization already validated internally  

In **Release** → pick the track → **Create new release**.

**Step 2: Upload the App Bundle**

- Upload `app-release.aab`  
- Add release notes (even for first version — e.g. “Initial release”)  
- Save; resolve any Play Console errors (version code conflicts, missing compliance, etc.)  

**Step 3: Review release and send for review**

Use **Preview and confirm** / **Review release** (wording varies). Fix blocking issues (policy, missing forms, country targeting).

When ready, **Send** the release for **Google Play review** (for production or open testing as configured).

**Step 4: Provide test credentials if needed**

If the app requires login, add **App access** instructions in Play Console (test account, demo mode, or license keys) so reviewers can sign in.

### 2.6 Release (Go Live)

**Step 1: Wait for Google review**

Google checks policy, behavior, and declared data use. Timing varies (often hours to a few days).

**Step 2: Managed publishing (recommended control)**

If **managed publishing** is on, the update stays **approved but not live** until you manually publish — useful for coordinating marketing and server switches.

For the first launch, many teams use managed publishing or a **staged rollout** (e.g. 20% → 100%) to limit blast radius.

**Step 3: After release**

After the app is available on Play:

- Install from Play on a real device and verify version  
- Test login and main flows  
- Watch **Android vitals** (ANRs, crashes) and your crash SDK (Crashlytics, Sentry) in the first hours  

**Step 4: If rejected or issues**

- Read the **Policy status** / email and Play Console messages  
- Fix policy, listing, or app issues  
- Bump **versionCode** (and usually versionName)  
- Upload a new `.aab` and resubmit  

---

## 3. Follow-Up Release (Update / Version Upgrade)

### 3.1 Purpose (What this is for)

This is used when your app is already live on Google Play and you want to ship a new version.

This can happen when:

- You fix bugs  
- You add features  
- You improve performance  
- You update backend API or UI logic  

Think of it as: **same app, improved version sent to Google Play again.**

### 3.2 Code Update (Do your changes first)

**Step 1: Pull latest code**

- Pull latest from Git (`main` / `develop`)  
- Confirm your local tree matches what you expect to ship  

**Step 2: Do your development work**

- Fix bugs, add features, improve UX  
- Update API integration if needed  

**Step 3: Test everything locally**

- Run on devices/emulators (`flutter run`)  
- Test login, navigation, core features  
- If anything is broken locally, stop and fix before building release  

### 3.3 Version Update (Very important step)

Open `pubspec.yaml`, for example:

```yaml
version: 1.0.1+2
```

Rules:

- `1.0.1` = user-visible version (**versionName**)  
- `+2` = **versionCode** — must be **strictly greater** than any build ever uploaded for this package on Play  

Important:

- Every new upload needs a **higher versionCode** than the last one on Play (no exceptions)  
- Bump **versionName** when you want users to see a new marketing version  

| Release   | Example `pubspec` version |
|-----------|---------------------------|
| First     | `1.0.0+1`                 |
| Second    | `1.0.1+2`                 |
| Third     | `1.0.2+3`                 |

### 3.4 Build & Upload Preparation

**Step 1: Clean project (recommended)**

```bash
flutter clean
flutter pub get
```

**Step 2: Verify Android config**

- Signing config still valid (upload key not expired or rotated incorrectly)  
- `applicationId` unchanged  
- No Gradle errors  

**Step 3: Build release bundle**

```bash
flutter build appbundle --release
```

If this succeeds, upload the new `.aab`.

### 3.5 Submission (Send new version to Google Play)

**Step 1: Create release in the right track**

Production updates: **Release** → **Production** → **Create new release** (or edit draft).

**Step 2: Upload the new App Bundle**

- Upload the new `.aab`  
- Ensure versionCode is higher than the live build  

**Step 3: Release notes**

Fill **Release notes** (“What’s new”) with short, user-facing bullets.

**Step 4: Review and send for review**

Complete any new compliance prompts, then submit for review.

### 3.6 Release (Go Live)

**Step 1: Wait for Google review**

Google may scan changes in behavior, permissions, and data safety relevance.

**Step 2: Rollout strategy**

- **Staged rollout** — increase percentage gradually (recommended for risky changes)  
- **Full rollout** — 100% when confident  
- **Managed publishing** — hold until you click publish after approval  

**Step 3: After release**

- Confirm new **versionName** on a device installed from Play  
- Re-test main flows  
- Monitor vitals, ANRs, and crash dashboards  

**Step 4: If rejected or blocked**

- Read Play’s messages carefully  
- Fix app or declaration issues  
- Increase **versionCode** again  
- Re-upload and resubmit  

---

## 4. Monitoring Phase (Post-Release)

### 4.1 Purpose (What this is for)

This stage starts right after users can install your app from Google Play.

Think of it as: **the app is live; your job is to watch if anything breaks or users complain.**

Main focus:

- Is the app stable?  
- Are users happy?  
- ANRs / crashes under control?  
- Do we need a hotfix?  

### 4.2 Stability Monitoring (Check if app is healthy)

**Step 1: Check crashes and ANRs daily**

Use:

- Firebase Crashlytics / Sentry  
- Play Console → **Quality** → **Android vitals** (crashes, ANRs, excessive wakeups, etc.)  

Watch:

- Spikes after a release  
- Stack traces tied to specific screens or devices  
- **ANR** rate (Android-specific “app not responding”)  

**Step 2: Check app performance**

- Cold start time  
- API latency and error rates  
- Login success rate  
- “Stuck loading” / blank screen patterns  

**Step 3: Identify patterns**

- Same crash on one manufacturer / Android version?  
- Same API failing?  
- ANRs on main thread due to heavy work?  

Classify: UI, backend, device-specific, or Play system behavior.

### 4.3 User Feedback (What users are saying)

**Step 1: Play Store reviews**

In Play Console → **Grow users** → **Ratings and reviews** (wording may vary):

- Read recent reviews and reply where helpful  
- Watch for repeated complaints (crash, login, payment)  

**Step 2: Support channels**

WhatsApp, email, tickets — same discipline as iOS: repeated themes are signal.

**Step 3: Group feedback**

Bucket into bugs, feature requests, and performance — prioritize by severity and frequency.

### 4.4 Analytics Monitoring (Understand user behavior)

**Step 1: Active users**

- DAU / MAU from analytics (Firebase, Amplitude, etc.)  

Sharp drops after a release warrant investigation.

**Step 2: User flows**

- Funnels: where do users drop?  
- Login failures vs successes  

**Step 3: Backend health**

- API 5xx / 429 rates  
- Timeouts  

Backend failures often look like “app broken” in reviews.

### 4.5 Incident Handling (What to do when something breaks)

**Step 1: Classify severity**

| Level | Meaning | Action |
|-------|---------|--------|
| **P0** (Critical) | Startup crash, login broken for most users, payments down | Hotfix **now** |
| **P1** (High) | Major feature broken but app partly usable | Patch quickly |
| **P2** (Low) | Minor UI / edge-case bugs | Next planned release |

**Step 2: Decide action**

- **P0** → hotfix track  
- **P1** → scheduled patch  
- **P2** → backlog  

**Step 3: Prepare hotfix if needed**

- Small branch, minimal change set  
- New **versionCode** for every upload  

**Step 4: Inform team**

Share what broke, ETA, and verification plan with dev, product, and support.

### Summary of Monitoring Phase

After release, your daily routine is:

1. Watch crashes and ANRs  
2. Read user feedback  
3. Monitor analytics and APIs  
4. Decide if a fix is needed  

**Goal:** keep the app stable and users happy.

---

## 5. Hotfix & Emergency Release Flow

### 5.1 Purpose (What this is for)

Used when production is seriously broken and you cannot wait for a normal release train.

Examples:

- Startup crash affecting many users  
- Login broken broadly  
- Payments or another critical path broken  
- Severe backend incident surfaced in the app  

### 5.2 Identify the Problem (Confirm it is really urgent)

**Step 1: Confirm the issue is real**

- Crash / ANR spike in vitals  
- Multiple independent user reports  
- Support alignment  
- Local reproduction when possible  

**Step 2: Check severity**

- **P0** → hotfix immediately  
- **P1** → fast patch  
- **P2** → normal cycle  

Only **P0** / critical **P1** belong in this hotfix flow.

### 5.3 Create Hotfix Branch (Start fixing immediately)

**Step 1: Create branch**

Examples:

- `hotfix/login-crash`  
- `hotfix/anr-main-thread`  

**Step 2: Fix ONLY the problem**

- No feature work, no drive-by refactors  
- Minimal diff, easy to review in minutes  

**Step 3: Test locally**

- Reproduce and verify fix  
- Smoke-test main user journeys  

### 5.4 Increase Version (Required before upload)

```yaml
version: 1.0.2+4
```

**versionCode** (`+4` here) must exceed whatever is already on Play.

### 5.5 Build Hotfix Version

```bash
flutter clean
flutter pub get
flutter build appbundle --release
```

### 5.6 Upload Hotfix to Google Play

**Step 1: Create production (or patched track) release**

Upload the new `.aab` to the same track users receive (usually **Production**).

**Step 2: Release notes**

Clearly state this is a **critical bugfix** (users and Google both read this).

**Step 3: Submit for review**

Send for review; fix any blocking policy or form issues immediately.

### 5.7 Fast Review & Rollout (Important for emergencies)

- Write concise **review notes** for Google: what broke, what you changed, scope of impact  
- If rollout is already bad, consider **halt** on a staged rollout (if applicable) while the fix is reviewed  
- Play does not offer a guaranteed Apple-style “expedited review,” but clear notes + legitimate regressions are still important  
- After approval, prefer **immediate smaller staged rollout** (e.g. 5–20%) then expand after vitals look good  

### 5.8 Release Hotfix

**Step 1: After approval**

Publish (or enable managed publish step) as soon as you are ready.

**Step 2: Verify**

- Install from Play on a clean device  
- Confirm crash/ANR rates fall  
- Spot-check the broken scenario  

### 5.9 Post Hotfix Review (Very important)

After the emergency:

1. **Check the root cause** — Why did it happen? Could it be prevented?  
2. **Document** — Incident log, timeline, contributing factors  
3. **Improve process** — Tests, monitoring alerts, feature flags, rollout discipline  

### Summary of Hotfix Flow

When production breaks:

1. Confirm the issue is real  
2. Classify severity (P0 / P1)  
3. Create hotfix branch  
4. Fix **only** the issue  
5. Test locally  
6. Increase version / **versionCode**  
7. Build `appbundle`  
8. Upload to Play Console  
9. Clear review notes; manage rollout safely  
10. Publish as soon as appropriate  
11. Verify in production  
12. Document root cause  

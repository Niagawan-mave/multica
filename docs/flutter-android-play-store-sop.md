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

Google Play needs a **signed** release build. With **Play App Signing** (default), you keep an **upload key**; Google holds the app-signing key users receive.

**Rules**

- Never commit `key.properties`, keystore files, or passwords to Git.  
- Add secrets to `android/.gitignore` (see below) and inject them in CI from a secret store.  
- Back up the keystore + passwords in a team password manager. **If you lose the upload key and cannot use Play’s reset flow, you may be unable to ship updates.**

---

**A) You already have a keystore from your team**

You should receive:

- A file such as `upload-keystore.jks` (or `.keystore`)  
- **Store password** (keystore password)  
- **Key password** (often the same as store password; confirm)  
- **Key alias** (e.g. `upload`)  

Put the keystore where your `storeFile` path can point to it. Two common layouts:

- Keystore in **`android/app/`** (next to `build.gradle`): use `storeFile=upload-keystore.jks` in `key.properties`.  
- Keystore in **`android/`** (parent of `app/`): use `storeFile=../upload-keystore.jks` (paths are resolved from **`android/app/`** when Gradle uses `file(...)` in `app/build.gradle`).

Create **`android/key.properties`** at the **android** project root (same level as `settings.gradle` — **not** inside `android/app/`):

```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

Example above assumes the keystore file is **`android/upload-keystore.jks`**. If you instead put the file in **`android/app/upload-keystore.jks`**, use `storeFile=upload-keystore.jks`.

Skip to **D) Wire Gradle** below.

---

**B) Create a new upload keystore (first app or new key)**

1. Install a JDK (or use Android Studio’s embedded JDK so `keytool` is on your `PATH`).  
2. From a safe directory (often your project’s `android/` folder):

```bash
cd android
keytool -genkey -v -keystore upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

- You will be prompted for **keystore password** and **key password** (you can set them the same; record both).  
- `-alias upload` is a common alias; you can choose another name — it must match `keyAlias` in `key.properties`.  
- `-validity 10000` is ~27 years; adjust if your policy requires.  

3. Create **`android/key.properties`** next to the keystore (same contents as in **A)**), using the passwords and alias you just chose. If you created the file inside **`android/`** as `android/upload-keystore.jks`, use:

```properties
storePassword=…
keyPassword=…
keyAlias=upload
storeFile=../upload-keystore.jks
```

If you moved the keystore into **`android/app/`**, use `storeFile=upload-keystore.jks` instead.

---

**C) Keep secrets out of Git**

In **`android/.gitignore`**, ensure at least:

```gitignore
key.properties
*.jks
*.keystore
```

If the keystore lives outside `android/`, still ignore `key.properties` and never commit the keystore path if it embeds secrets.

---

**D) Wire Gradle to load `key.properties`**

Flutter’s [official Android signing steps](https://docs.flutter.dev/deployment/android#sign-the-app) show the full file. Summary for **`android/app/build.gradle`** (Groovy):

1. **Above** the `android {` block:

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```

2. **Inside** `android {`:

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

3. **Inside** `buildTypes { release { ... } }`, set:

```groovy
            signingConfig signingConfigs.release
```

If you use **`android/app/build.gradle.kts`**, the same idea applies: load `Properties` from `rootProject.file("key.properties")` and map `signingConfigs.release` — follow Android’s Kotlin DSL signing docs or align with your template.

Then run **`flutter build appbundle --release`**. If signing fails, check: path to `storeFile`, alias, passwords, and that `key.properties` is really under **`android/`**.

---

**E) Play App Signing on first upload**

The first time you upload an `.aab`, Play Console will guide you to accept **Play App Signing**. Keep a secure backup of the **upload** keystore; Google manages the rest for store installs.

---

**F) Match package name and SDK policy**

Make sure:

- `applicationId` / namespace matches the package name registered in Play Console **exactly**  
- `minSdk`, `targetSdk`, and `compileSdk` meet [Play’s targets](https://developer.android.com/google/play/requirements/target-sdk) (policy changes over time — check current requirements)  

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

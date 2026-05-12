# Flutter Android Google Play SOP (Role-Based Guide)

## Table of Contents

- [1. Overview](#1-overview)
- [2. First Time App Release (Initial Launch — Android / Google Play)](#2-first-time-app-release-initial-launch--android--google-play)
  - [2.1 Purpose](#21-purpose-what-this-is-for)
  - [2.2 Setup Phase](#22-setup-phase-prepare-google-play-account-first)
  - [2.3 Connect Signing to Gradle](#23-connect-signing-to-gradle-make-android-build-signed)
  - [2.4 Build Preparation](#24-build-preparation-make-app-ready)
  - [2.5 Store Listing Setup](#25-store-listing-setup-what-users-will-see)
  - [2.6 Upload & First Release](#26-upload--first-release)
  - [2.7 Release (Go Live)](#27-release-go-live)
- [3. Follow-Up Release (Update / Version Upgrade — Android)](#3-follow-up-release-update--version-upgrade--android)
  - [3.1 Purpose](#31-purpose-what-this-is-for)
  - [3.2 Code Update](#32-code-update-do-your-changes-first)
  - [3.3 Version Update](#33-version-update-very-important-step)
  - [3.4 Build Preparation](#34-build-preparation-make-app-ready-again)
  - [3.5 Submission](#35-submission-send-new-version-to-google-play)
  - [3.6 Release](#36-release-go-live-1)
  - [Summary of Update Release Flow](#summary-of-update-release-flow)
- [4. Monitoring Phase (Post-Release)](#4-monitoring-phase-post-release)
  - [4.1 Purpose](#41-purpose-what-this-is-for-1)
  - [4.2 Immediate Post-Release Verification](#42-immediate-post-release-verification-first-12-hours)
  - [4.3 Stability Monitoring](#43-stability-monitoring-daily-monitoring)
  - [4.4 User Feedback Monitoring](#44-user-feedback-monitoring)
  - [4.5 Analytics Monitoring](#45-analytics-monitoring-understand-user-behavior)
  - [4.6 Security & Store Monitoring](#46-security--store-monitoring)
  - [4.7 Incident Handling](#47-incident-handling-what-to-do-when-something-breaks)
  - [4.8 Summary of Monitoring Phase](#48-summary-of-monitoring-phase)
- [5. Hotfix & Emergency Release Flow](#5-hotfix--emergency-release-flow)
  - [5.1 Purpose](#51-purpose-what-this-is-for-2)
  - [5.2 Confirm the Issue First](#52-confirm-the-issue-first-do-not-panic-release)
  - [5.3 Create Hotfix Branch](#53-create-hotfix-branch)
  - [5.4 Testing Before Release](#54-testing-before-release-never-skip-even-during-emergency)
  - [5.5 Version Update](#55-version-update-required-before-upload)
  - [5.6 Build Hotfix Release](#56-build-hotfix-release)
  - [5.7 Upload Emergency Release](#57-upload-emergency-release)
  - [5.8 Rollout Strategy](#58-rollout-strategy-very-important)
  - [5.9 Communication During Incident](#59-communication-during-incident)
  - [5.10 Post-Incident Review](#510-post-incident-review-very-important)
  - [5.11 Summary of Hotfix Flow](#511-summary-of-hotfix-flow)

---

## 1. Overview

This document defines a simplified, role-based workflow for publishing and maintaining Flutter Android applications on **Google Play**.

It is structured into four main operational modes:

1. **First Time Release** — initial launch → [Section 2](#2-first-time-app-release-initial-launch--android--google-play)  
2. **Follow-Up Release** — updates / new versions → [Section 3](#3-follow-up-release-update--version-upgrade--android)  
3. **Monitoring Phase** — post-release operations → [Section 4](#4-monitoring-phase-post-release)  
4. **Hotfix / Emergency Phase** — maintenance when production is broken → [Section 5](#5-hotfix--emergency-release-flow)  

Each section is designed so developers only follow what is relevant to their current task.

---

## 2. First Time App Release (Initial Launch — Android / Google Play)

### 2.1 Purpose (What this is for)

This is the first time you want to publish your Flutter app to **Google Play**.

Think of it like:  
👉 **You are not just building the app — you are also preparing Google’s system to accept it.**

You will be doing:

- Google Play setup  
- App signing setup  
- First release build  
- Upload to Play Console  
- First review submission  

### 2.2 Setup Phase (Prepare Google Play Account First)

**Step 1: Login to Google Play Console**

Go to [Google Play Console](https://play.google.com/console) and make sure:

- You are using the correct developer account  
- You have permission to create and publish apps (**Admin** / **Release Manager**)  

If you cannot create apps, ask your account owner.

**Step 2: Create New App**

In Play Console:

- Click **Create app**  

Fill in:

- App name  
- Default language  
- App type (App / Game)  
- Free or Paid  

After this, Google will create your app dashboard.

**Important:**  
👉 The **package name** (`applicationId`) is fixed here and **cannot change later**.

**Step 3: Setup App Signing (Very Important Step)**

Google uses **Play App Signing**, which means:

- Google stores the final signing key  
- You only upload an **upload key**  

You will need:

- Keystore file (`.jks` / `.keystore`)  
- `key.properties` file (password config)  

**Step 4: Create or Configure Keystore**

If you don’t have a keystore yet, go to the **`android/`** folder and run:

```bash
cd android
keytool -genkey -v -keystore upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

You will be asked for:

- Password for keystore  
- Key password  
- Alias name  

👉 **Save all passwords safely** — they are very important; recovery is difficult or impossible if lost.

**Step 5: Create `key.properties` file**

Create the file:

`android/key.properties`

Add:

```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=upload
storeFile=../upload-keystore.jks
```

Adjust `storeFile` if your keystore path differs (paths are resolved from `android/app/` when Gradle uses `file(...)` in `android/app/build.gradle` — e.g. use `upload-keystore.jks` if the file lives in `android/app/`).

**Step 6: Ignore secrets in Git**

Open `android/.gitignore` and make sure you add:

```gitignore
key.properties
*.jks
*.keystore
```

👉 **Never push the keystore or `key.properties` to Git.**

### 2.3 Connect Signing to Gradle (Make Android Build Signed)

**Step 1: Load `key.properties` in Gradle**

Open `android/app/build.gradle` (or your Groovy Gradle entry for the `app` module).

Add this **above** `android {`:

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```

**Step 2: Add signing config**

Inside `android {`:

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

**Step 3: Enable release signing**

Inside `buildTypes`:

```groovy
        release {
            signingConfig signingConfigs.release
        }
```

If you use Kotlin DSL (`build.gradle.kts`), apply the same idea with the Kotlin Gradle APIs. See Flutter’s [Sign the app](https://docs.flutter.dev/deployment/android#sign-the-app) guide.

### 2.4 Build Preparation (Make App Ready)

**Step 1: Switch Flutter to production mode**

Make sure:

- API is production (not dev/staging)  
- Debug logs are removed or gated off  
- No test data used  

**Step 2: Clean project**

```bash
flutter clean
flutter pub get
```

**Step 3: Set version number**

Open `pubspec.yaml` and set:

```yaml
version: 1.0.0+1
```

Rules:

- `1.0.0` = visible version (users see on Play)  
- `+1` = **versionCode** — must increase on **every** upload  

**Step 4: Build Android App Bundle**

```bash
flutter build appbundle --release
```

Output file:

`build/app/outputs/bundle/release/app-release.aab`

👉 **Google Play requires `.aab` format** for standard store distribution (not APK for this flow).

### 2.5 Store Listing Setup (What users will see)

**Step 1: Fill app details in Play Console**

Go to **Store listing** (wording in the console may vary slightly).

Fill in:

- App name  
- Short description  
- Full description  
- App icon  
- Feature graphic  

**Step 2: Upload screenshots**

Make sure:

- Real app screens (not fake UI)  
- No broken UI  
- Main features are shown clearly  

**Step 3: Complete required forms**

You must fill:

- **Data safety** form  
- **Privacy policy** URL  
- **Content rating**  
- **Target audience**  

Wrong or incomplete info here increases **rejection risk** — one of the fastest ways to get stuck in review.

### 2.6 Upload & First Release

**Step 1: Choose release track**

You can choose:

- **Internal testing** (recommended first)  
- **Closed testing**  
- **Production** (direct release)  

**Step 2: Upload `.aab` file**

Go to **Release** → **Create new release** → upload `app-release.aab`.

**Step 3: Add release notes**

Examples:

- Initial release  
- First version of the app  

**Step 4: Review and submit**

Fix any warnings:

- Missing privacy info  
- Version conflicts  
- Policy issues  

Then click **Submit for review**.  
👉 Fix warnings first — do not submit with blocking policy or version errors.

### 2.7 Release (Go Live)

**Step 1: Wait for Google review**

Google will:

- Check app behavior  
- Check policy compliance  
- Validate data declarations  

Time: **a few hours to a few days** (varies).

**Step 2: After approval**

You choose:

- **Staged rollout** (recommended)  
- **Full rollout**  

**Step 3: After release check**

After the app goes live:

- Install from Play Store  
- Test login and main features  
- Check crashes (**Android vitals** / **Crashlytics**)  

**Step 4: If rejected**

Do this:

- Read the rejection reason  
- Fix the issue  
- Increase **versionCode**  
- Re-upload  

---

## 3. Follow-Up Release (Update / Version Upgrade — Android)

### 3.1 Purpose (What this is for)

This section is used when your Android app is **already live** on Google Play and you want to release a new version.

Think of it like:  
👉 **The app is already on users’ phones — you are sending an improved version through Google Play.**

This usually happens when:

- You fix bugs  
- You add new features  
- You improve performance  
- You update API or backend logic  

### 3.2 Code Update (Do your changes first)

**Step 1: Pull latest code**

Before doing anything:

- Pull latest code from Git (`main` or `develop` branch)  
- Make sure your local project is up to date  

**Step 2: Do your changes**

Do your development work:

- Fix bugs  
- Add new features  
- Improve UI/UX  
- Update API or business logic  

**Step 3: Test everything locally**

Before building release:

- Run the app in debug mode  
- Test main flows: login, navigation, core features  
- Make sure nothing crashes  

If something is broken here → fix it first before continuing.

### 3.3 Version Update (Very important step)

Open `pubspec.yaml` and update version, for example:

```yaml
version: 1.0.1+2
```

Simple rules:

- `1.0.1` = app version (users see this on Play)  
- `+2` = **versionCode** (internal Google tracking)  

Important rules:

- You **must** increase **versionCode** on every upload — even for a tiny bugfix  
- Google Play **rejects** duplicate `versionCode`  

Example:

| Release      | Version   |
|--------------|-----------|
| First        | `1.0.0+1` |
| Second       | `1.0.1+2` |
| Third        | `1.0.2+3` |

### 3.4 Build Preparation (Make app ready again)

**Step 1: Clean project (recommended)**

```bash
flutter clean
flutter pub get
```

**Step 2: Check Android signing**

Make sure:

- Keystore still exists  
- `key.properties` is correct  
- Signing config is not broken  

If signing fails → fix before continuing.

**Step 3: Build App Bundle**

```bash
flutter build appbundle --release
```

Output:

`build/app/outputs/bundle/release/app-release.aab`

This file is what you upload to Google Play.

### 3.5 Submission (Send new version to Google Play)

**Step 1: Go to Play Console**

Open your app → **Release** section.

Choose **Production** or **Internal / Closed testing** (if you need a gate first).

**Step 2: Create new release**

Click **Create new release** → upload the new `.aab`.

Make sure:

- `versionCode` is higher than the previous release  
- No errors shown in Play Console  

**Step 3: Write release notes**

Keep it short and user-friendly, for example:

- Fixed login bug  
- Improved performance  
- UI improvements  
- Bug fixes and stability improvements  

**Step 4: Review release**

Check:

- Missing privacy declarations  
- Policy warnings  
- Version conflicts  

Fix any blocking issue before continuing.

**Step 5: Submit for review**

Click **Submit for review**. Google will process your update.

### 3.6 Release (Go Live)

**Step 1: Wait for Google review**

Google will:

- Check app behavior  
- Check policy compliance  
- Verify data safety declarations  

**Step 2: Choose rollout method**

After approval:

- **Staged rollout** (recommended): e.g. 5% → 20% → 50% → 100%  
- **Full rollout** (immediate release to everyone)  

**Recommended:**  
👉 Use staged rollout for safety.

**Step 3: After release**

Once the update is live:

- Install from Play Store  
- Test main features  
- Check crash rate  
- Monitor Android vitals  

**Step 4: If something goes wrong**

If issues appear after release:

- **Pause rollout** immediately (if staged rollout is still in progress)  
- Fix the issue in code  
- Increase **versionCode** again  
- Re-upload a new build  

### Summary of Update Release Flow

When updating the app:

1. Pull latest code  
2. Make changes  
3. Test locally  
4. Update version name + **versionCode**  
5. Build `.aab`  
6. Upload to Play Console  
7. Fill release notes  
8. Submit for review  
9. Roll out gradually  
10. Monitor after release  

---

## 4. Monitoring Phase (Post-Release)

### 4.1 Purpose (What this is for)

This phase starts **immediately** after the app is released to Google Play.

Think of it like:  
👉 **The app is now used by real users — monitor whether everything stays stable.**

At this stage your job is to:

- Monitor crashes  
- Watch user complaints  
- Check backend stability  
- Ensure a new release does not break production  

Many issues **only appear** after real users start using the app.

### 4.2 Immediate Post-Release Verification (First 1–2 Hours)

This is the **most important** monitoring window.

Right after rollout:

**Step 1: Install app from Play Store yourself**

Do **not** only test the debug build.

Install directly from the **Google Play Store** production version, then test:

- Login / logout  
- Navigation  
- API loading  
- Push notifications (if applicable)  
- Payment flow (if applicable)  

👉 This confirms the **release build** behaves correctly for users.

**Step 2: Verify correct version released**

Check:

- `versionName`  
- `versionCode`  
- Release notes  

Make sure users received the **correct** build (not an old artifact by mistake).

**Step 3: Monitor crash spikes immediately**

Open:

- Firebase Crashlytics  
- Sentry  
- **Android vitals** (Play Console)  

Watch for:

- Sudden crash increase  
- ANR increase  
- Startup crash  

If startup crashes spike → users may not even open the app. Treat as **P0** immediately.

**Step 4: Check backend / API traffic**

Coordinate with the backend team if needed.

Monitor:

- API error rate  
- Login failures  
- Server CPU spikes  
- Timeout increases  

👉 Sometimes the **app release is fine**, but the **backend** cannot handle traffic.

### 4.3 Stability Monitoring (Daily Monitoring)

**Step 1: Check crash reports daily**

Go to:

- Firebase Crashlytics  
- Sentry  
- Google Play Console → **Android vitals**  

Monitor:

- Crash-free users %  
- Top crash screen  
- Affected Android version  
- Affected device model  

**The same crash repeated many times** → treat as a real production issue.

**Step 2: Monitor ANR (App Not Responding)**

ANRs matter a lot on Android.

Common causes:

- Heavy work on the UI thread  
- Blocked main thread  
- Slow database query  
- Poor API handling  

If ANRs increase → users feel freezes / lag. Play may also reduce visibility if ANRs are too high.

**Step 3: Monitor app performance**

Check:

- App startup speed  
- API loading speed  
- Image loading performance  
- Memory usage  
- Battery impact (if applicable)  

Common user complaints:

- “App slow after update”  
- “Phone becomes hot”  
- “Battery drain”  

These often indicate a **performance regression**.

**Step 4: Watch device-specific issues**

Android is fragmented. Check:

- Samsung-only issue?  
- Xiaomi-only issue?  
- Android 13-only issue?  
- Tablet-only issue?  

Some bugs only reproduce on **specific** devices.

### 4.4 User Feedback Monitoring

**Step 1: Check Play Store reviews**

Open **Play Console → Ratings & reviews**.

Watch for:

- Repeated complaints  
- Sudden rating drop  
- Negative reviews after an update  

Important keywords:

- crash  
- login fail  
- slow  
- cannot open  
- payment failed  

If **multiple** users report the same thing → assume the issue is **real**.

**Step 2: Monitor support channels**

Check:

- WhatsApp support  
- Customer service tickets  
- Email support  
- Internal bug reports  

Pay attention to:

- Same issue repeated  
- Urgent business impact  
- High-priority customer complaints  

**Step 3: Group feedback properly**

Do **not** react message-by-message in isolation.

Group into:

- Crashes  
- UI bugs  
- Performance issues  
- Feature requests  
- Backend issues  

This helps prioritize fixes.

### 4.5 Analytics Monitoring (Understand user behavior)

**Step 1: Monitor active users**

Check:

- DAU (Daily Active Users)  
- MAU (Monthly Active Users)  

If DAU **suddenly drops** after a release → the release likely introduced a serious issue.

**Step 2: Monitor user flow**

Track:

- Login success rate  
- Checkout / payment completion  
- Screen drop-off rate  
- Onboarding completion  

If users stop at the same screen → likely **UX issue** or **hidden bug**.

**Step 3: Monitor API health**

Check:

- API timeout rate  
- 4xx / 5xx errors  
- Slow endpoints  
- Database performance  

👉 Sometimes the Flutter app is healthy but the **backend** is failing.

**Step 4: Monitor notification delivery (if using push)**

Check:

- Firebase notification delivery  
- Token registration issues  
- Delayed notifications  

If notifications fail → users may think the app is “broken.”

### 4.6 Security & Store Monitoring

**Step 1: Watch Play Console warnings**

Google Play may show:

- Policy warnings  
- SDK security warnings  
- Outdated dependency alerts  

Do not ignore these — some warnings can eventually **block updates** or lead to **removal** if ignored long-term.

**Step 2: Monitor SDK compatibility**

Check:

- Target SDK requirements  
- Deprecated API usage  
- Play policy changes  

Google updates requirements **frequently**.

### 4.7 Incident Handling (What to do when something breaks)

**Step 1: Classify severity**

| Level | Meaning | Action |
|-------|---------|--------|
| **P0** | App unusable / startup crash / login broken | Immediate hotfix |
| **P1** | Major feature issue | Fast patch |
| **P2** | Minor issue | Next release |

**Step 2: Decide response**

- **P0** → immediate hotfix  
- **P1** → quick update release  
- **P2** → backlog  

**Step 3: Prepare hotfix if needed**

If the issue is critical:

- Create a hotfix branch  
- Fix **only** the affected issue  
- Increase **versionCode**  
- Rebuild and upload quickly  

Do **not** mix new features into a hotfix.

**Step 4: Inform internal team**

Always notify:

- Developers  
- Support team  
- Product owner  
- Backend team (if API-related)  

Everyone should understand:

- Impact  
- Fix timeline  
- Workaround (if any)  

### 4.8 Summary of Monitoring Phase

After release, your routine should be:

- Monitor crashes & ANRs  
- Monitor backend health  
- Check user feedback  
- Monitor analytics  
- Detect release problems early  
- Prepare a hotfix if needed  

👉 **Goal:** keep production stable and reduce user impact as fast as possible.

---

## 5. Hotfix & Emergency Release Flow

### 5.1 Purpose (What this is for)

This flow is used when something **critical** breaks in production and **cannot** wait for the normal release cycle.

Think of it like:  
👉 **Production has a serious issue — fix and release as fast and safely as possible.**

Usually used for:

- App crash on startup  
- Login completely broken  
- Payment failure  
- Critical API issue  
- Major production bug affecting many users  

Hotfix releases should always focus on:  
👉 **Fixing the problem as fast and safely as possible** (minimal extra risk).

### 5.2 Confirm the Issue First (Do not panic release)

**Step 1: Confirm issue is real**

Before creating a hotfix, check:

- Firebase Crashlytics  
- Android vitals  
- Support complaints  
- Backend logs  

Confirm:

- The issue affects real users  
- The issue is reproducible (when possible)  
- It is not a one-off isolated user problem  

👉 Do **not** ship a hotfix only because **one** user reported something strange.

**Step 2: Check impact level**

| Level | Meaning | Action |
|-------|---------|--------|
| **P0** | App unusable / startup crash / login broken | Immediate hotfix |
| **P1** | Major feature partially broken | Fast patch |
| **P2** | Minor issue | Next normal release |

Only **P0** and urgent **P1** should trigger the emergency release flow.

**Step 3: Decide whether rollout should be paused**

If the issue comes from the latest release:

- Go to Google Play Console  
- **Pause staged rollout** immediately (if rollout is still active)  

This limits how many users receive the broken version.

### 5.3 Create Hotfix Branch

**Step 1: Create dedicated hotfix branch**

Examples:

- `hotfix/login-crash`  
- `hotfix/payment-failure`  

Keep the branch name focused and simple.

**Step 2: Fix ONLY the affected issue**

Do **not**:

- Add new features  
- Refactor unrelated code  
- Upgrade dependencies unnecessarily unless required for the fix  

Hotfix should contain the **smallest possible safe change**.  
👉 Emergency releases must **reduce** risk, not create new risk.  
👉 Emergency releases must **reduce** risk, not create new risk.

**Step 3: Review root cause**

Before coding blindly, check:

- Why the issue happened  
- When it started  
- Whether backend is involved  
- Whether it is device-specific  

This avoids “fake fixes” that do not address the real cause.

### 5.4 Testing Before Release (Never skip even during emergency)

**Step 1: Reproduce issue locally**

Try to reproduce:

- The same crash  
- The same API failure  
- The same broken flow  

If you cannot reproduce → investigate more before releasing.

**Step 2: Verify fix works**

After fixing:

- Repeat the same steps  
- Confirm the issue is gone  
- Confirm you did not introduce obvious new breakage  

**Step 3: Test critical flows again**

Even during a hotfix, always test:

- Login  
- App startup  
- API loading  
- Navigation  
- Payment flow (if applicable)  

Hotfixes can accidentally break unrelated areas.

**Step 4: Test release build (important)**

Do **not** only test debug mode.

Build a **release** `.aab` (or install the release build) and validate the **actual** release configuration.

Some issues only happen in **release** mode.

### 5.5 Version Update (Required before upload)

Open `pubspec.yaml` and update version, for example:

```yaml
version: 1.0.2+4
```

Rules reminder:

- Always increase **versionCode**  
- Google Play rejects duplicate **versionCode**  
- Emergency fixes still require a new **versionCode**  

### 5.6 Build Hotfix Release

**Step 1: Clean project**

```bash
flutter clean
flutter pub get
```

**Step 2: Build App Bundle**

```bash
flutter build appbundle --release
```

Output:

`build/app/outputs/bundle/release/app-release.aab`

**Step 3: Verify correct build generated**

Before upload:

- Verify `versionName`  
- Verify `versionCode`  
- Verify correct environment / API endpoints  

Wrong-environment release is a common production mistake.

### 5.7 Upload Emergency Release

**Step 1: Go to Google Play Console**

Open your app → **Production** release section (or the track your users receive).

**Step 2: Create emergency release**

Upload the new `.aab`.

Double-check:

- Correct **versionCode**  
- Correct build artifact  
- Release notes updated  

**Step 3: Write release notes clearly**

Examples:

- Fixed startup crash affecting some users  
- Fixed login issue after latest update  
- Emergency stability improvements  

Keep notes simple and honest.

### 5.8 Rollout Strategy (Very important)

**Step 1: Prefer staged rollout first**

Even for a hotfix, it is often safer to roll out gradually first, for example:

- 5% → 20% → 50% → 100%  

Reason: verify the hotfix does **not** create a new production issue.

**Step 2: Monitor immediately after rollout**

After rollout:

- Monitor crash rate  
- Monitor ANR  
- Monitor support complaints  
- Monitor backend load  

The first **1–2 hours** are critical.

**Step 3: Stop rollout if a new issue appears**

If a new serious issue appears:

- Halt rollout immediately  
- Investigate before continuing  

Never continue rollout blindly.

### 5.9 Communication During Incident

**Step 1: Inform internal team**

Always update:

- Developers  
- Support team  
- Product owner  
- Backend team  

Include:

- What happened  
- Severity level  
- Estimated fix time  
- Workaround (if available)  

**Step 2: Prepare support response**

Support should know:

- What users may experience  
- Whether a workaround exists  
- Expected resolution timing  

This reduces customer confusion.

### 5.10 Post-Incident Review (Very important)

**Step 1: Investigate root cause**

After the issue stabilizes, review:

- Why the bug escaped testing  
- Whether monitoring missed warning signs  
- Whether process failed somewhere  

**Step 2: Improve prevention**

Add:

- Better validation  
- Automated tests  
- Monitoring alerts  
- QA checklist improvements  

**Goal:** prevent the same incident from happening again.

**Step 3: Document incident**

Record internally:

- Issue summary  
- Affected version  
- Impact  
- Fix applied  
- Timeline  

This helps future troubleshooting.

### 5.11 Summary of Hotfix Flow

When a critical production issue happens:

1. Confirm the issue is real  
2. Classify severity  
3. Pause rollout if needed  
4. Create hotfix branch  
5. Fix only the affected issue  
6. Test carefully (including release build)  
7. Increase **versionCode**  
8. Build release `.aab`  
9. Upload emergency release  
10. Roll out carefully  
11. Monitor production closely  
12. Document the incident and improve process  

👉 **Goal:** restore production stability as fast and safely as possible.

# Flutter iOS App Store SOP (Role-Based Guide)

## Table of Contents

- [1. Overview](#1-overview)
- [2. First Time App Release (Initial Launch)](#2-first-time-app-release-initial-launch)
  - [2.1 Purpose](#21-purpose-what-you-are-doing-here)
  - [2.2 Setup Phase](#22-setup-phase-prepare-apple-side-first)
  - [2.3 Build Preparation](#23-build-preparation-make-app-ready-for-release)
  - [2.4 Store Listing Preparation](#24-store-listing-preparation-what-users-will-see-in-app-store)
  - [2.5 Upload & Submission](#25-upload--submission-send-app-to-apple)
  - [2.6 Release](#26-release-go-live)
- [3. Follow-Up Release (Update / Version Upgrade)](#3-follow-up-release-update--version-upgrade)
  - [3.1 Purpose](#31-purpose-what-this-is-for)
  - [3.2 Code Update](#32-code-update-do-your-changes-first)
  - [3.3 Version Update](#33-version-update-very-important-step)
  - [3.4 Build & Upload Preparation](#34-build--upload-preparation)
  - [3.5 Submission](#35-submission-send-new-version-to-apple)
  - [3.6 Release](#36-release-go-live-1)
- [4. Monitoring Phase (Post-Release)](#4-monitoring-phase-post-release)
  - [4.1 Purpose](#41-purpose-what-this-is-for-1)
  - [4.2 Stability Monitoring](#42-stability-monitoring-check-if-app-is-healthy)
  - [4.3 User Feedback](#43-user-feedback-what-users-are-saying)
  - [4.4 Analytics Monitoring](#44-analytics-monitoring-understand-user-behavior)
  - [4.5 Incident Handling](#45-incident-handling-what-to-do-when-something-breaks)
- [5. Hotfix & Emergency Release Flow](#5-hotfix--emergency-release-flow)
  - [5.1 Purpose](#51-purpose-what-this-is-for-2)
  - [5.2 Identify the Problem](#52-identify-the-problem-confirm-it-is-really-urgent)
  - [5.3 Create Hotfix Branch](#53-create-hotfix-branch-start-fixing-immediately)
  - [5.4 Increase Version](#54-increase-version-required-before-upload)
  - [5.5 Build Hotfix Version](#55-build-hotfix-version)
  - [5.6 Upload Hotfix to Apple](#56-upload-hotfix-to-apple)
  - [5.7 Fast Review Request](#57-fast-review-request-important-for-emergencies)
  - [5.8 Release Hotfix](#58-release-hotfix)
  - [5.9 Post Hotfix Review](#59-post-hotfix-review-very-important)

---

## 1. Overview

This document defines a simplified, role-based workflow for publishing and maintaining Flutter iOS applications on the Apple App Store.

It is structured into three main operational modes:

1. **First Time Release** — initial launch  
2. **Follow-Up Release** — updates / new versions  
3. **Monitoring Phase** — post-release operations  

Each section is designed so developers only follow what is relevant to their current task.

---

## 2. First Time App Release (Initial Launch)

### 2.1 Purpose (What you are doing here)

In this stage, you are preparing everything needed to put your Flutter app on the Apple App Store for the very first time.

Think of it like this: you are not just building the app — you are also setting up Apple’s system so it accepts your app.

### 2.2 Setup Phase (Prepare Apple Side First)

#### Step 1: Login to Apple Developer

Go to Apple Developer account and make sure:

- You can access the team account  
- You have permission to create apps  

If you cannot see App Store Connect access, ask your team admin.

#### Step 2: Create App in App Store Connect

Go to App Store Connect and click **New App**. Fill in:

- **App Name** — what users will see  
- **Primary Language** — English or others  
- **Bundle ID** — must match your Flutter/Xcode project  
- **SKU** — internal ID (e.g. `app_001`)  

Once done, Apple will create a “container” for your app.

#### Step 3: Setup Signing (This is where most people get stuck)

Open Apple Developer portal. You need:

- iOS Distribution Certificate  
- Provisioning Profile (App Store type)  

Make sure:

- Bundle ID matches **exactly** with Xcode project  
- Team selected is correct in Xcode  

If you are using automatic signing in Xcode:

- Correct Apple account selected  
- “Automatically manage signing” is enabled  

### 2.3 Build Preparation (Make App Ready for Release)

#### Step 1: Switch Flutter to production mode

In your Flutter project, make sure:

- API URL is production (not staging/dev)  
- Debug prints are removed or disabled  
- No test data is used  

#### Step 2: Clean project (important before build)

```bash
flutter clean
flutter pub get
```

This removes the old build cache.

#### Step 3: Open iOS project in Xcode

```bash
open ios/Runner.xcworkspace
```

Inside Xcode check:

- Signing is correct  
- Bundle ID is correct  
- No errors shown at top  

#### Step 4: Set version number

Open `pubspec.yaml` and set:

```yaml
version: 1.0.0+1
```

Simple rule:

- `1.0.0` = app version (users see this)  
- `+1` = build number (must increase every upload)  

#### Step 5: Build release version

```bash
flutter build ios --release
```

If this succeeds, your app is ready for Xcode archive.

### 2.4 Store Listing Preparation (What users will see in App Store)

This is where you prepare everything visible to Apple reviewers and users.

#### Step 1: Prepare app information

In App Store Connect, fill in:

- App description (what your app does)  
- Subtitle (short summary)  
- Keywords (helps search)  
- Support URL / Privacy Policy URL  

#### Step 2: Prepare screenshots

You need screenshots for iPhone sizes. Make sure:

- No broken UI  
- Real app content (not dummy data)  
- Main features are shown clearly  

#### Step 3: Privacy information

You must tell Apple:

- What data you collect  
- Why you collect it  
- Whether user data is shared  

Also add a Privacy Policy link.

### 2.5 Upload & Submission (Send App to Apple)

#### Step 1: Open Xcode Archive

In Xcode:

- **Product → Archive**  
- Wait until build finishes  

#### Step 2: Upload to Apple

After archive finishes:

- Open **Organizer**  
- Click **Distribute App**  
- Choose **App Store Connect**  
- Click **Upload**  
- Wait for processing  

#### Step 3: Check App Store Connect

In the browser:

- Build appears  
- Status becomes **Processed**  

If it fails, fix signing or build issues.

#### Step 4: Submit for review

- Select build  
- Fill review form  
- If your app needs login: provide test account for Apple reviewer  
- Click **Submit for Review**  

### 2.6 Release (Go Live)

#### Step 1: Wait for Apple review

Apple will:

- Test your app  
- Check UI  
- Check policy compliance  

This can take hours to days.

#### Step 2: If approved

You choose:

- **Manual release** — you control when it goes live  
- **Automatic release** — go live immediately  

Recommended: manual release the first time.

#### Step 3: After release

After the app goes live:

- Open App Store and verify it appears  
- Test login and main features  
- Check crash reports (very important in the first few hours)  

#### Step 4: If rejected

Don’t panic:

- Read Apple’s rejection reason  
- Fix the issue  
- Increase build number  
- Resubmit  

---

## 3. Follow-Up Release (Update / Version Upgrade)

### 3.1 Purpose (What this is for)

This is used when your app is already live in the App Store, and you want to release a new version.

This can happen when:

- You fix bugs  
- You add new features  
- You improve performance  
- You update backend API or UI logic  

Think of it as: **same app, improved version sent to Apple again.**

### 3.2 Code Update (Do your changes first)

#### Step 1: Pull latest code

Before doing anything:

- Pull latest code from Git (`main` / `develop` branch)  
- Make sure your local project is updated  

#### Step 2: Do your development work

Apply your changes:

- Fix bugs  
- Add features  
- Improve UI/UX  
- Update API integration if needed  

#### Step 3: Test everything locally

Before building:

- Run app in debug mode  
- Test main flows: login, navigation, core features  
- Make sure nothing crashes  

If anything is broken locally, stop here and fix it first.

### 3.3 Version Update (Very important step)

Open `pubspec.yaml` and update version, for example:

```yaml
version: 1.0.1+2
```

Simple rules:

- First part (`1.0.1`) = App version (user-visible in the App Store)  
- Second part (`+2`) = Build number (internal tracking)  

Important:

- Always increase version when releasing updates  
- Always increase build number for **every** upload (no exception)  

Example progression:

| Release   | Version   |
|-----------|-----------|
| First     | `1.0.0+1` |
| Second    | `1.0.1+2` |
| Third     | `1.0.2+3` |

### 3.4 Build & Upload Preparation

#### Step 1: Clean project (recommended)

```bash
flutter clean
flutter pub get
```

This removes old cached builds and avoids weird issues.

#### Step 2: Open iOS project in Xcode

```bash
open ios/Runner.xcworkspace
```

Inside Xcode check:

- Signing still correct  
- Bundle ID unchanged  
- No red errors  

#### Step 3: Build release version

```bash
flutter build ios --release
```

If this succeeds, you are ready to upload.

### 3.5 Submission (Send new version to Apple)

#### Step 1: Archive in Xcode

**Product → Archive** — wait until build completes.

#### Step 2: Upload to App Store Connect

After archive finishes:

- Open **Organizer**  
- Select archive  
- **Distribute App** → **App Store Connect** → **Upload**  

Wait until Apple processes the build.

#### Step 3: Select build in App Store Connect

- Open your app  
- Go to the new version page (e.g. `1.0.1`)  
- Select uploaded build  

#### Step 4: Fill “What’s New”

Write a short explanation for users, for example:

- Fixed login issue  
- Improved app performance  
- Bug fixes and UI improvements  
- Enhanced stability  

Keep it short and easy to understand.

#### Step 5: Submit for review

Click **Submit for Review**. Apple will review the new version again.

### 3.6 Release (Go Live)

#### Step 1: Wait for Apple review

Apple will:

- Test updated features  
- Check app stability  
- Ensure no policy violations  

#### Step 2: If approved

Choose release method:

- **Manual release** (recommended)  
- **Automatic release**  

Recommended: use **manual release** so you control rollout timing.

#### Step 3: After release

After the update goes live:

- Verify new version appears in the App Store  
- Test main features again  
- Monitor crash reports  
- Check user feedback  

#### Step 4: If rejected

If Apple rejects the update:

- Read rejection reason carefully  
- Fix the issue in code or metadata  
- Increase build number again  
- Resubmit  

---

## 4. Monitoring Phase (Post-Release)

### 4.1 Purpose (What this is for)

This stage starts right after your app is published on the App Store.

Think of it as: **the app is live; your job is to watch if anything breaks or users complain.**

You are not building features here — you are monitoring real users.

Main focus:

- Is the app stable?  
- Are users happy?  
- Is anything crashing?  
- Do we need a hotfix?  

### 4.2 Stability Monitoring (Check if app is healthy)

#### Step 1: Check crash reports daily

Go to your crash tools, for example:

- Firebase Crashlytics  
- Sentry  
- App Store Connect → Crashes  

Check:

- New crashes after release  
- Which screen caused the crash  
- Which device / iOS version is affected  

If you see a **sudden spike in crashes** → treat as urgent.

#### Step 2: Check app performance

Open your monitoring dashboard and look at:

- App startup speed  
- API response time  
- Login success rate  
- Loading screen stuck issues  

Watch for user complaints like:

- “App very slow”  
- “Stuck loading”  
- “Blank screen”  

These usually mean performance or API issues.

#### Step 3: Identify pattern (very important)

Don’t fix random reports one by one. Instead check:

- Same crash happening repeatedly?  
- Same screen affected?  
- Same API failing?  

Then classify:

- UI issue  
- Backend issue  
- Device-specific issue  

This helps you find where the real problem is.

### 4.3 User Feedback (What users are saying)

#### Step 1: Check App Store reviews

In App Store Connect:

- Read latest reviews  
- Look for repeated complaints  

Pay attention to:

- “App crash”  
- “Cannot login”  
- “Payment failed”  
- “App stuck”  

If multiple users report the same issue → likely a real bug.

#### Step 2: Check support messages

Check:

- WhatsApp support  
- Email support  
- Internal ticket system  

Look for:

- Repeated complaints  
- Same issue from different users  
- Critical complaints (login / payment)  

#### Step 3: Group feedback

Don’t treat each message as a separate problem. Group into:

- Bug reports  
- Feature requests  
- Performance issues  

This helps you decide what to fix first.

### 4.4 Analytics Monitoring (Understand user behavior)

#### Step 1: Check active users

Look at:

- Daily Active Users (DAU)  
- Monthly Active Users (MAU)  

If DAU **suddenly drops** after release → something may be wrong in the app.

#### Step 2: Check user flow

Track user journey:

- Where users stop using the app  
- Login success vs failure  
- Main feature usage  

If many users drop at the same screen → that screen likely has a bug or bad UX.

#### Step 3: Check API health

Look at backend metrics:

- API error rate  
- API timeout rate  
- Slow endpoints  

If the API is failing → the problem may be backend, not the Flutter app.

### 4.5 Incident Handling (What to do when something breaks)

#### Step 1: Classify severity

| Level | Meaning | Action |
|-------|---------|--------|
| **P0** (Critical) | App crash on startup, login completely broken, payment not working | Fix **now** — hotfix required |
| **P1** (High) | Feature not working but app usable, partial API failure | Fix ASAP in next update or quick patch |
| **P2** (Low) | UI misalignment, small bug, minor issue | Fix in next planned release |

#### Step 2: Decide action

Based on severity:

- **P0** → Immediate hotfix release  
- **P1** → Schedule quick update  
- **P2** → Add to backlog  

#### Step 3: Prepare hotfix if needed

If critical issue found:

- Create fix branch  
- Fix **only** the issue (no extra changes)  
- Increase build number  
- Release as fast as possible  

#### Step 4: Inform team

Always update developers, product owner, and support so everyone knows:

- What is broken  
- What is being fixed  
- When the fix will be released  

### Summary of Monitoring Phase

After release, your daily routine is:

1. Watch crashes  
2. Read user feedback  
3. Monitor analytics  
4. Decide if a fix is needed  

**Goal:** keep the app stable and users happy.

---

## 5. Hotfix & Emergency Release Flow

### 5.1 Purpose (What this is for)

This section is used when something is seriously broken in production and you cannot wait for the normal release cycle.

Think of it as: **the app is live, but something is wrong — we must fix it immediately.**

Usually used when:

- App crashes on launch  
- Login is broken for all users  
- Payment or a key feature is not working  
- Critical API failure affecting users  

### 5.2 Identify the Problem (Confirm it is really urgent)

#### Step 1: Confirm issue is real

Before acting fast, make sure:

- Crash reports are increasing  
- Users are reporting the same issue  
- Support team confirms complaints  
- You can reproduce the issue  

If only one user reports → might not be urgent. If many users report → treat as real.

#### Step 2: Check severity

- **P0** → App broken / unusable → **HOTFIX NOW**  
- **P1** → Important feature broken → fix quickly  
- **P2** → Minor bug → normal release later  

Only **P0** and critical **P1** go into the hotfix flow.

### 5.3 Create Hotfix Branch (Start fixing immediately)

#### Step 1: Create new branch

From main code, create a branch such as:

- `hotfix/login-crash`  
- `hotfix/api-failure`  

Keep names simple and clear.

#### Step 2: Fix ONLY the problem

Important rules:

- Do **not** add new features  
- Do **not** refactor unrelated code  
- **Only** fix the issue  

Examples: fix crash line, API endpoint, null error, login logic.

#### Step 3: Test fix locally

Before building:

- Run app  
- Reproduce issue  
- Confirm fix works  
- Test main flow still works  

Do not skip this step, even for a hotfix.

### 5.4 Increase Version (Required before upload)

Open `pubspec.yaml` and update, for example:

```yaml
version: 1.0.2+4
```

Rule reminder:

- Always increase build number for emergency fixes — **no exceptions.**

### 5.5 Build Hotfix Version

#### Step 1: Clean project

```bash
flutter clean
flutter pub get
```

#### Step 2: Build release

```bash
flutter build ios --release
```

If this passes → continue.

### 5.6 Upload Hotfix to Apple

#### Step 1: Archive in Xcode

**Product → Archive** — wait until build completes.

#### Step 2: Upload build

- Open **Organizer**  
- Select archive  
- **Distribute App** → upload to App Store Connect  

#### Step 3: Select build in App Store Connect

- Open correct app version  
- Select uploaded build  
- Make sure version matches hotfix  

### 5.7 Fast Review Request (Important for emergencies)

When submitting:

- Mark issue as critical bug fix  
- Explain clearly in review notes: what was broken, what was fixed, why it is urgent  

If needed: request **expedited review** (Apple sometimes allows this for critical issues).

### 5.8 Release Hotfix

#### Step 1: Wait for approval

Apple will often review hotfixes faster.

#### Step 2: Release immediately after approval

Choose manual release (recommended) or immediate release if needed.

#### Step 3: Verify fix in production

After release:

- Test the fixed issue again  
- Check crash reports drop  
- Confirm users can use the app normally  

### 5.9 Post Hotfix Review (Very important)

After an emergency fix:

1. **Check root cause** — Why did it happen? Could it be prevented?  
2. **Document** — Add to internal bug list; record what went wrong  
3. **Improve process** — Add validation, test cases, better monitoring  

### Summary of Hotfix Flow

When production breaks:

1. Confirm the issue is real  
2. Classify severity (P0 / P1)  
3. Create hotfix branch  
4. Fix **only** the issue  
5. Test locally  
6. Increase version / build  
7. Build release  
8. Upload to Apple  
9. Request fast review if appropriate  
10. Release ASAP  
11. Verify fix  
12. Document root cause  

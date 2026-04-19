# Signaro: Advanced macOS Code Signing & Notarization Utility

Signaro is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, notarizing, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, comprehensive extended attributes cleaning, advanced persistent logging, working folders project management, professional DMG creation, intelligent notarization requirements validation, modular MVVM architecture, and comprehensive code documentation, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

Current Status: Feature-complete core functionality with ongoing enhancements. See roadmap for upcoming features.

---
## What's New in Version 4.1 Build 1.1

Follow-up to 4.1 Build 1.0. Fixes two silent no-ops in the App Distribution workflow's Advanced DMG Options: the custom **App icon x/y position** and the **Text size** stepper both rode the full configuration pipeline but never reached the produced DMG.

### Bug Fixes

- **Custom App icon position is now applied.** The dialog's icon-position x/y stepper previously looked wired but fell back to the hardcoded `(140, 180)` because the AppleScript builder looked up the position under a different dictionary key than the UI wrote. Because the fallback matched the dialog's initial values, the regression was invisible — adjusting the stepper produced no change in the mounted DMG.
- **Text size is now emitted into the Finder layout script.** The "Text: 12 pt" stepper had no effect on the final DMG. The value flowed through the workflow but was never written to Finder's icon-view options. It is now applied when set.

### Known Good

- Other Advanced DMG Options (custom volume icon, background image, window size, icon size, Applications alias position) were verified end-to-end during this pass and behave as documented in 4.1 Build 1.0.

---

## What's New in Version 4.1 Build 1.0

Minor release. Custom volume icons, background images, and Finder window layouts now actually appear on mounted DMGs — the previous implementation silently dropped them because the DMG customization pipeline wrote its metadata in a spot Apple's image tool discards. The App Distribution workflow (sign → notarize → staple → DMG) gains the full customization surface directly in its dialog.

### Headline Fix

- **Custom volume icons now land on mounted DMGs.** Previously the icon file was copied onto the disk but the "this volume has a custom icon" flag never persisted, so Finder rendered the generic drive icon instead. Fixed by rebuilding the customization pipeline to apply icons, backgrounds, and window layout on the writable mount post-creation.

### New Features

- **Advanced DMG Options in the App Distribution workflow.** The notarization dialog adds a collapsed "Advanced DMG Options" disclosure with controls for custom volume icon (`.icns`), background image (PNG/TIFF/JPG), window width and height, icon size, text size, and icon positions for the app bundle and the Applications alias.
- **Clearer primary action.** The main button now reads "Sign, Notarize, Staple & Distribute" instead of just "Distribute…" so the full workflow is visible at a glance.
- **Explicit success alert after DMG creation.** A confirmation dialog surfaces the filename, size, and save location — the prior status-bar text was easy to miss on a multi-second operation.

### Reliability

- **Finder layout now persists.** Background picture binding, window bounds, and icon positions actually survive to the final DMG. Earlier builds didn't wait long enough for Finder to flush its window state before unmounting.
- **File pickers open reliably.** Both the volume-icon picker and background-image picker in the Create DMG and App Distribution dialogs fire every time — a SwiftUI quirk previously shadowed them.
- **AppleScript control of Finder is now properly requested.** On first DMG with a custom layout, macOS prompts "Signaro would like to control Finder." Click OK. You can manage this later in System Settings → Privacy & Security → Automation. If denied, the layout step fails gracefully with a visible error instead of producing a DMG with incomplete customization.
- **Better errors when layout can't be applied** — AppleScript failures now surface in the creation log instead of being swallowed.

### First-Run Note

The first time you produce a DMG with a custom window layout, macOS will prompt *"Signaro would like to control Finder."* This prompt is required because Signaro uses Finder to render the window background, set window bounds, and position icons. Click OK. If you miss the prompt or click Don't Allow, custom backgrounds and layouts won't persist — you can re-enable in System Settings → Privacy & Security → Automation.

---

## What's New in Version 4.0 Build 1.1

Point release. Adds custom volume icon support to the DMG creation dialog (without the working pipeline that ships in 4.1 — see above). Part of the Advanced DMG Customization & Automation track.

- **Custom Volume Icon toggle.** The DMG Creation dialog's Appearance section gains a "Custom Volume Icon" toggle with a file picker restricted to `.icns`. The chosen icon becomes the mounted volume's icon.
- **Layout script compatibility.** Custom icon choices round-trip through Signaro's declarative DMG layout JSON alongside window bounds and background image.

---

## What's New in Version 4.0 Build 1.0

Major architecture and UI overhaul. 13 of 16 items in the 4.0 roadmap shipped; build is clean on macOS 13.5+ with zero errors or warnings.

### UI

- **Sign on the default row** alongside Select Files, Analyze, Create DMG, and Clear — the app's namesake action is always one click away.
- **Distribute elevated to the primary accent button** for the end-to-end release flow.
- **Advanced disclosure** collapses the lower-frequency primitives (Unsign, Check Attributes, Notarize, Check Status, Staple Tickets); state persists across launches.
- **Toolbar consolidation.** Primary action shelf reduces from ~10 controls to three verbs — Check Requirements · Distribute · Create DMG — plus a single overflow menu (Credentials, Working Folders, Logs, Preferences).
- **Working Folders sidebar.** When enabled, the main view switches to a split layout with a sidebar; folder selection drives file signature analysis.
- **Certificate Status Pill** on the selected identity surfaces days-to-expiration with escalating color (neutral → advisory → warning → error).
- **Unified help popover** replaces ad-hoc info buttons with one consistent title/message/learn-more component.
- **Preferences expanded to seven tabs:** General, Logging, Certificates, DMG Defaults, Analytics, Notifications, and Advanced Timing.

### New Features

- **Certificate Expiration Monitoring** — every Developer ID identity evaluated locally on launch and daily thereafter. No network calls.
- **Working Folder Templates** — save, apply, export, and import project templates as JSON. Secrets are scrubbed on save and refused on import.
- **Distribution Analytics** — every sign/notarize/staple/DMG event recorded to local per-day JSON with 90-day retention, CSV/JSON export, and a purge control. The Analytics tab renders totals, success rate, median/p95 notarization duration, outcome breakdown, and top identities. On-device only.
- **Enhanced Retry Logic** — exponential backoff with jitter around notarization submissions. Transient 5xx responses, DNS failures, and timeouts retry automatically (5 attempts, 2-second base, 60-second cap).
- **Workflow Checkpoints + Resume Banner** — interrupted App or PKG distribution workflows surface on next launch with a "Discard All" action. Non-secret state only.
- **Advanced DMG Layout Authoring** — a declarative JSON format captures window bounds, icon positions, background image, license resources, view style, icon/text size, and compression. The Advanced DMG Options dialog gains Load Layout…, Save Layout…, and Reset to Default.
- **Performance instrumentation** — every subprocess invocation and the full notarization polling wait are tagged with begin/end signposts visible under the Signaro category in Xcode Instruments.
- **Localization catalog** — ~940 user-facing strings auto-extracted from the interface. Non-English locales are now a translation-only workflow.

### Architecture

- **Design tokens** (spacing, radii, palette, typography, stroke) replace ad-hoc literals across screens.
- **Four oversized view files decomposed** into focused sub-files, each reduced to an 8-10 line forwarding shim at the original location. Zero regressions in the rename+split pass.
- **Service-layer separation** for certificate monitoring, working folder templates, distribution analytics, workflow checkpoints, DMG layout scripting, and layout loading. All strictly-concurrent under Swift 6. Zero network calls.
- **Retry utility** with extensible error classification (retryable / fatal / authentication failure).

### Bug Fixes

- **Inspector crash revert.** An earlier attempt to migrate the distribution dialogs into the native inspector column on macOS 14+ was rolled back because the dismiss environment behaves differently outside a sheet scope. Dialogs are now reliably presented as sheets again.
- **Concurrency and cast warnings** eliminated across the notarization flow, working folder manager, and distribution metrics store.
- **DMG layout adapter fixes** so saved layouts round-trip cleanly when reloaded.

---

## What's New in Version 3.5 Build 1.0

### Critical Bug Fix: App Distribution Workflow Stapling

- **Fixed:** App Distribution workflow was failing at the stapling step with "File is already stapled!" even when notarization had succeeded.
  - **Root Cause:** The workflow's pre-validation check treated the "already stapled" state as a failure.
  - **Scenario:** Triggered when Apple stapled automatically during notarization, or when re-running the workflow on the same file.
  - **Resolution:** Pre-staple validation now recognizes "already stapled" and treats it as success ("✅ App bundle already stapled successfully"). Only genuine validation failures are treated as errors.
  - **Impact:** Workflow completes cleanly on previously processed files; false-positive failures are gone; re-running the workflow is safe.

### Technical Details

- Affected the App Distribution workflow only; the PKG workflow already handled this case correctly.
- Enhanced staple-step logic for both app bundles and DMGs.
- Clearer distinction between "already stapled" (success) and actual validation errors (failure).

### User Experience Improvements

- Clearer success messages when stapling steps are skipped.
- More accurate workflow completion status.
- Reduced confusion when re-processing previously notarized files.
- Better handling of Apple's automatic ticket stapling behavior.

---

## What's New in Version 3.2 Build 1.1

### Bug Fixes & UI Improvements

- **Fixed:** UI layout in the Preferences dialog.
  - Improved spacing and alignment across all preference tabs.
  - Better visual hierarchy and readability.

---

## What's New in Version 3.2 Build 1.0

### Bug Fixes & UI Improvements

#### Notarization Credential Dialog Fix

- **Fixed:** App-Specific Password input not enabling the Continue button.
  - **Issue:** Typing in the app-specific password field did not activate Continue, forcing users to switch tabs as a workaround.
  - **Resolution:** Password-field validation now updates immediately, enabling Continue as soon as all required fields are filled.
  - **Impact:** Smoother credential entry with immediate UI feedback.

#### Notarization Workflow Dialog Layout

- **Fixed:** UI elements being cut off in the notarization workflow dialog.
  - **Issue:** A fixed dialog height clipped file lists, results, and action buttons when content grew.
  - **Resolution:** Flexible sizing (min 500, ideal 600, max 800) plus a scrollable content area.
  - **Improvements:** All content scrollable; action buttons fixed at the bottom with consistent padding; better text wrapping; scrollable file list; toast message repositioned away from buttons.
  - **User Experience:** Dialog displays all content regardless of file count or message length.

#### Code Quality & Localization

- **Fixed:** Memory leak during certificate loading — Core Foundation objects now clean up correctly when certificate data is invalid.
- **Improved:** All user-facing strings prepared for full localization with proper comment annotations for translators.

### Technical Improvements

- Enhanced UI responsiveness in credential dialogs.
- Better memory management in certificate handling.
- Improved layout system for dialogs with dynamic content.
- Cleaner console output.
- More robust validation state management.

---

## What's New in Version 3.1 Build 1.5

### Stability, Concurrency, and Build Reliability

- Unified all shell interactions through a single shared process runner for clarity and consistency.
- Hardened output collection to satisfy Swift 6 strict concurrency requirements — eliminates data races and the related warnings.
- Cleaned up unnecessary async boilerplate and unreachable error paths.
- Standardized string trimming and splitting for more predictable behavior across locales.

### Notarization Flow Improvements

- Notarization operations consistently use the shared process runner for submissions, stapling, and Gatekeeper checks.
- Fixed team-ID argument handling and several small parsing issues for more reliable status checks and log retrieval.
- Hardened stapling pre-checks and result handling with clearer user messaging.

### DMG Creation and Distribution Workflows

- Unified local runner calls in DMG creation.
- Added an explicit file-system parameter to all DMG creation paths (defaults to user preference or APFS).
- Resolved segmentation, background image, and layout customizations to be robust under concurrent UI updates.

### Code Signing

- Fixed trailing-closure parse errors in the in-place signing path.
- Ensured signing and unsigning paths call the shared runner consistently.

### UI & Project Structure

- Added Preferences UI (Logging, Certificate visibility, and DMG default filesystem).
- Restored clean project references; eliminated stale references to removed files.

### Developer Experience

- Standardized redaction for sensitive values (like app-specific passwords) in logs.
- Reduced spurious warnings by removing unreachable error branches and simplifying async paths.

### Versioning & Metadata

- Marketing version bumped to 3.1 (build remains 1.5).

### Bug Fixes & Reliability

- **PKG Signing:** Resolved a persistent "Bad file descriptor" error when signing `.pkg` files from directories without write permissions. Signing now uses a temporary directory, so the source file's location no longer matters.

---

## What's New in Version 2.8 Build 1.0

### New: PKG Distribution Workflow

- End-to-end workflow tailored for `.pkg` installers:
  - Signs the package with a Developer ID Installer certificate.
  - Optional notarization and stapling.
  - Optional DMG packaging containing the signed package.
  - Optional DMG signing (Developer ID Application), notarization, and stapling.
  - Output directory, volume name, and identity selection in the UI.
  - Credential modes: Apple ID/App Password, Keychain Profile, ASC API Key.
- Integrated validation with clear recovery guidance.
- Status, step-by-step progress, and results with "Reveal in Finder."

### Distribution Workflow Enhancements (App and PKG)

- Unified Distribution Workflow entry:
  - Selection containing `.app` → App Distribution dialog.
  - Selection containing `.pkg` → PKG Distribution dialog.
  - Credential gating prompts for missing auth, then resumes your intended flow.
- Intelligent pre-flight validation and skipping of redundant steps.
- Improved progress weights and user feedback.

### Notarization Credentials & Validation

- Unified credentials UI with three modes:
  - Apple ID + App-Specific Password.
  - Keychain Profile (via `xcrun notarytool store-credentials`).
  - App Store Connect API Key (.p8).
- "Test Profile" validation for Keychain Profile and improved error feedback.
- Comprehensive notarization requirements validation (Detailed mode) with category scoring.

### UX and Reliability

- Flow gating: sensitive actions bring up credential sheets when needed.
- Clearer help and status strings across dialogs and toolbars.
- Expanded results displays with friendly summaries, Request IDs, and stapling calls-to-action.

---

## Core Features

- **Advanced Code Signing:** In-place or copy-based signing with Developer ID certificates, full hardened runtime support, and extended attributes cleaning.
- **Intelligent Notarization Requirements Validation:** Pre-submission analysis with detailed requirement checks and actionable recommendations.
- **Working Folders Project Management:** Organize files into projects with persistent state tracking and batch operations.
- **Professional DMG Creation:** Customizable disk images with Applications alias, background images, window layouts, and multiple compression formats — with the volume icon, background, and window layout now reliably persisted (v4.1+).
- **App Distribution Workflow:** End-to-end `.app` signing, notarization, stapling, and DMG packaging in one automated workflow. Gains full DMG customization in v4.1.
- **PKG Distribution Workflow:** Complete `.pkg` signing, notarization, and optional DMG packaging with Developer ID Installer certificates.
- **Apple Notarization Integration:** Seamless submission to Apple's notarization service with multiple credential modes and automatic status tracking.
- **Advanced Persistent Logging:** Comprehensive operation logs with command-line details, Apple responses, and timestamped audit trails.
- **Certificate Management:** Automatic discovery and validation of Developer ID certificates with expiration tracking.
- **File Analysis & Verification:** Deep inspection of signatures, entitlements, notarization status, and Gatekeeper assessment.
- **Distribution Analytics:** On-device record of every sign/notarize/staple/DMG event with success rates, median/p95 durations, and CSV/JSON export.
- **Professional UI & UX:** Native SwiftUI interface with drag-and-drop, real-time feedback, and accessibility support.

---

## Screenshots & Interface Overview

- Main Interface
- Notarization Requirements Validation
- Working Folders Management
- DMG Creation Interface (with Advanced DMG Options disclosure in v4.1+)
- App Distribution Workflow
- PKG Distribution Workflow
- Enhanced Logging System
- Extended Attributes Management
- Certificate Management
- Advanced Timing (Polling & Stapling)

---

## How-To: App Distribution (End-to-End)

- Select your `.app` in Signaro and choose your Developer ID Application certificate.
- Click **Sign, Notarize, Staple & Distribute** (toolbar or primary button).
- In the dialog:
  - If not already signed with the selected cert, Signaro will sign in place (hardened runtime expected).
  - Submit to Apple for notarization (choose your credential mode).
  - Optionally wait for the result, then staple automatically when accepted.
  - Create a distribution DMG with Applications alias and brand-friendly layout.
  - Sign the DMG (Developer ID Application) and optionally notarize + staple the DMG.
- Expand **Advanced DMG Options** (v4.1+) to configure custom volume icon, background image, window size, icon/text size, and icon positions for the app and Applications alias.
- Results include Request IDs, durations, and "Reveal in Finder."

**Tips:**

- Already signed/notarized artifacts are detected to skip unnecessary steps.
- Files that are already stapled are recognized and skipped gracefully (v3.5+).
- For CI or team usage, prefer Keychain Profile or ASC API Key credentials.
- On the first run with a custom DMG layout, macOS will prompt for Automation access to Finder — click OK (v4.1+).

---

## How-To: PKG Distribution (End-to-End)

- Select your `.pkg` and choose your Developer ID Installer certificate.
- Click the distribution workflow button.
- In the dialog:
  - The installer is signed with a copy-based approach.
  - Optional notarization, wait, and staple.
  - Optional DMG creation containing the signed package.
  - Optional DMG signing (Developer ID Application) and notarization + stapling.
  - Choose the output folder and volume name; optionally notarize the DMG.
- Results include Request IDs and "Reveal in Finder."

**Tips:**

- Packages rarely need extended-attributes cleaning; keep it off unless needed.
- Enable DMG signing and notarization if you want the final container to carry a ticket too.

---

## Notarization Credentials — Modes, How to Get Them, and When to Use Each

### Credential Modes (all supported)

**Apple ID + App-Specific Password**

- **Get:** appleid.apple.com → Security → App-Specific Passwords → Generate.
- **Use:** Personal workflows, quick manual submissions.

**Keychain Profile (notarytool store-credentials)**

- **Get:** `xcrun notarytool store-credentials PROFILE --key AuthKey_XXXX.p8 --key-id <KID> --issuer <ISSUER>`
- **Use:** Day-to-day local development and CI on a machine with a login keychain; easiest to "just work" repeatedly.

**App Store Connect API Key (.p8)**

- **Get:** App Store Connect → Users and Access → Keys → API Keys → Generate → download `.p8`; note Key ID (KID) and Issuer ID.
- **Use:** CI/CD, headless agents, or restricted accounts where an Apple ID shouldn't be used. Provides clean rotation and separation.

### Best-Practice Examples

- **Indie dev, manual:** Apple ID + App Password is fine for occasional runs.
- **Team/CI on a Mac build agent:** Keychain Profile for reliability with notarytool.
- **Enterprise CI/CD (GitHub Actions, Jenkins):** ASC API Key with a least-privilege role, `.p8` stored securely, rotated regularly.

---

## Troubleshooting: Common Log Messages

- **"code object is not signed at all" / "valid=false":** Sign appropriately and re-verify.
- **"In architecture: x86_64/arm64":** Informational message about binary architectures.
- **Identity manager filtered certificate:** A non-applicable certificate type was ignored.
- **RenderBox / ViewBridge messages:** Benign system logs from the Xcode debugger.
- **"Unable to obtain a task name port right":** Benign Xcode debugger noise; safe to ignore.
- **"ViewBridge to RemoteViewService Terminated":** Normal system sheet / dialog close event.
- **Stapler error 65:** Raise grace wait or retry backoff (see Advanced Timing).
- **"File is already stapled!":** Handled gracefully in v3.5+ (workflow treats as success).
- **"Signaro would like to control Finder" prompt:** First-run Automation permission requested by v4.1+ so custom DMG window layouts can be applied. Click OK.
- **Custom icon or background not showing on a DMG:** (v4.1+) Confirm Signaro is granted Automation access in System Settings → Privacy & Security → Automation → Signaro → Finder. If toggled off, the layout step fails silently.

### Known Console Messages (Safe to Ignore)

These appear during normal operation and don't affect functionality:

- `Unable to obtain a task name port right for pid XXX: (os/kern) failure (0x5)` — Xcode debugger inspection.
- `ViewBridge to RemoteViewService Terminated: Error Domain=com.apple.ViewBridge Code=18` — normal dialog dismissal.
- Identity-manager "successfully loaded X code signing identities" printed multiple times — SwiftUI view refreshes.

---

## System Requirements

- macOS 13.5 or later.
- Xcode Command Line Tools.
- Apple Developer Account.
- Keychain Access permissions.
- Internet connection for notarization.
- Automation permission for Finder (v4.1+, granted on first use when a DMG with custom layout is produced).

---

- [GitHub](https://github.com/hov172)
- [PowerShell Gallery](https://www.powershellgallery.com/profiles/hov172)
- 📨 Slack: **@Hov172**
- 🕹️ Discord: **Jay172_**
- [LinkedIn](https://www.linkedin.com/in/jesus-a-785bb616?trk=people-guest_people_search-card)
- 🐦 [Twitter / X (@AyalaSolutions)](https://twitter.com/AyalaSolutions)
- <a href="https://bsky.app/profile/ayalasolutions.bsky.social"><img src="https://raw.githubusercontent.com/bluesky-social/social-app/main/assets/logo.png" width="20" alt="Bluesky Logo"></a> [@AyalaSolutions](https://bsky.app/profile/ayalasolutions.bsky.social)
- 📧 *Contact via GitHub, Social accounts issues or discussions.*

---

## Version Information

- **Current Version:** 4.1 Build 1.0
- **Release:** 2026
- **Platform:** macOS (Universal Binary)
- **Architecture:** SwiftUI + MVVM, advanced logging, working folders, professional DMG creation, intelligent validation, on-device analytics, workflow checkpoints

---

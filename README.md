# Signaro: Advanced macOS Code Signing & Notarization Utility

Signaro is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, notarizing, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, comprehensive extended attributes cleaning, advanced persistent logging, working folders project management, professional DMG creation, intelligent notarization requirements validation, modular MVVM architecture, and comprehensive code documentation, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

Current Status: Feature-complete core functionality with ongoing enhancements. See roadmap for upcoming features.

---

## What's New in Version 3.5 Build 1.0

### Critical Bug Fix: App Distribution Workflow Stapling

#### Fixed: "File is already stapled!" Causing Workflow Failure
- **Issue:** App Distribution Workflow would fail at the stapling step with error "Failed at Staple App Bundle: File is already stapled!" even though the file was successfully notarized
  - **Root Cause:** The workflow was performing a pre-validation check using `validateFileForStapling()` which returns `canStaple: false` when a file is already stapled. This was being treated as a failure instead of recognizing it as a successful state.
  - **Scenario:** This occurred when Apple automatically stapled the notarization ticket to the file during the notarization process, or when running the workflow multiple times on the same file.
  - **Resolution:** 
    - Added intelligent pre-staple validation that checks if `canStaple: false` is due to "already stapled" status
    - When a file is already stapled, the workflow now correctly treats this as a **success** and displays "✅ App bundle already stapled successfully"
    - Only non-"already stapled" validation failures are treated as actual errors
  - **Impact:** 
    - App Distribution Workflow now completes successfully when files are already stapled
    - Eliminates false-positive failures in the workflow
    - Better handles re-running workflows on previously processed files
    - Provides clear success messaging when stapling is skipped due to existing ticket

#### Technical Details
- **Files Modified:** 
  - `AppDistributionWorkflow.swift` - Enhanced `stapleAppBundle()` and `stapleDMG()` functions
  - Added pre-validation checks before stapling operations
  - Distinguishes between "already stapled" (success) vs. actual validation errors (failure)
- **Affected Workflows:** App Distribution Workflow only
  - PKG Distribution Workflow unaffected (already handles this correctly)

### User Experience Improvements
- Clearer success messages when stapling steps are skipped
- More accurate workflow completion status
- Reduced confusion when re-processing previously notarized files
- Better handling of Apple's automatic ticket stapling behavior

---

## What's New in Version 3.2 Build 1.1

### Bug Fixes & UI Improvements
- **Fixed:** UI Layout in Preferences dialog
  - Improved spacing and alignment across all preference tabs
  - Better visual hierarchy and readability

---

## What's New in Version 3.2 Build 1.0

### Bug Fixes & UI Improvements

#### Notarization Credential Dialog Fix
- **Fixed:** App-Specific Password input not enabling "Continue" button
  - **Issue:** When entering Apple ID credentials, typing in the app-specific password field would not activate the "Continue" button, requiring users to switch tabs (to Keychain Profile and back to Apple ID) as a workaround
  - **Root Cause:** Missing proper `onChange` handler integration for password field validation
  - **Resolution:** Implemented proper `onChange` handler with immediate validation state updates to enable the "Continue" button as soon as all required fields are filled
  - **Impact:** Smoother credential entry experience with immediate UI feedback, eliminating the need for the tab-switching workaround

#### Notarization Workflow Dialog Layout
- **Fixed:** UI elements being cut off in notarization workflow dialog
  - **Issue:** Fixed frame height caused content overflow, cutting off file lists, results, and action buttons
  - **Resolution:** Replaced fixed height with flexible sizing (`minHeight: 500, idealHeight: 600, maxHeight: 800`) and wrapped content in ScrollView
  - **Improvements:**
    - All content now scrollable to prevent cutoff
    - Fixed action buttons at bottom with consistent padding
    - Better text wrapping with `fixedSize(horizontal: false, vertical: true)`
    - Scrollable file list with max height constraint
    - Adjusted toast message positioning to avoid covering buttons
  - **User Experience:** Dialog now properly displays all content regardless of number of files or result messages

#### Code Quality & Localization
- **Fixed:** Memory leak in certificate loading (ViewController.mm line 264)
  - Proper cleanup of `serialCF` Core Foundation object when certificate data is invalid
  - Satisfies static analyzer requirements for memory management
  - Prevents potential memory accumulation during repeated certificate loading
- **Improved:** All user-facing strings now use `NSLocalizedString()` macro
  - Full localization support preparation across all Objective-C components
  - Proper comment annotations for translators
  - Affects alert messages, button titles, dialog titles, and error messages
  - Eliminates localization warnings from Xcode static analyzer

### Technical Improvements
- Enhanced UI responsiveness in credential dialogs
- Better memory management in Objective-C certificate handling
- Improved layout system for dialogs with dynamic content
- Cleaner console output (reduced Xcode debugger noise)
- More robust validation state management

---

## What's New in Version 3.1 Build 1.5

### Stability, Concurrency, and Build Reliability
- Reworked process execution to a single shared `ProcessRunner` actor and removed scattered runner references. All shell interactions now go through `await ProcessRunner.shared.run(...)` for clarity and consistency.
- Introduced a safe, lock‑backed output collector to satisfy Swift 6 `@Sendable` concurrency requirements and eliminate data races and warnings (no more "mutation of captured var in concurrently‑executing code").
- Cleaned up unnecessary `await` on actor singletons and eliminated unreachable `catch` blocks around non‑throwing async paths.
- Standardized string trimming and splitting calls to use explicit `CharacterSet` (e.g., `CharacterSet.whitespacesAndNewlines`) to fix inference errors.

### Notarization Flow Improvements
- Notarization operations now consistently rely on `ProcessRunner` for `notarytool`, `stapler`, and `spctl` invocations.
- Fixed team ID argument handling and several small parsing issues for more reliable status checks and log retrieval.
- Hardened stapling pre‑checks and result handling with clearer user messaging.

### DMG Creation and Distribution Workflows
- Unified Local runner calls in DMG creation with `ProcessRunner` and removed stray braces/closures that caused parse errors.
- Added explicit `fileSystem` parameter to all `DMGCreationOptions` call sites (defaults to user preference or APFS), fixing missing‑argument compile errors.
- Resolved segmentation, background image, and layout customizations to be robust even under concurrent UI updates.

### Code Signing
- Fixed an extra trailing closure/parse error in `CodeSigningOperations.signInPlace` and clarified `CharacterSet` usage.
- Ensured signing and unsigning paths call the shared runner consistently.

### UI/Project Structure
- Added Preferences UI (Logging, Certificates visibility, and DMG default filesystem). Wired the dialog into the main view and included in the target.
- Resolved Xcode project parse issues by restoring a clean project file and re‑adding missing source references (e.g., `ProcessRunner.swift`, `PreferencesView.swift`).
- Eliminated stale references to removed files and ensured all sources are present in the build phase.

### Developer Experience
- Standardized command redaction for sensitive values (e.g., app‑specific passwords) in logs.
- Reduced spurious warnings by removing unreachable `catch` blocks and simplifying async paths.

### Versioning & Metadata
- Bumped marketing version to 3.1 in Xcode project settings (build remains 1.5).

These changes collectively deliver a more reliable build, cleaner concurrency semantics, and stronger, more predictable notarization and distribution workflows.

### Bug Fixes & Reliability
- **PKG Signing:** Resolved a persistent issue where signing `.pkg` files would fail with a "Bad file descriptor" error, particularly when the app was running from a directory it didn't have write permissions to. The signing operation now uses a temporary directory, ensuring the process completes reliably regardless of the source file's location.

---

## What's New in Version 2.8 Build 1.0

### New: PKG Distribution Workflow
- End-to-end workflow tailored for .pkg installers:
  - Signs .pkg with Developer ID Installer (productsign)
  - Optional notarization and stapling
  - Optional DMG packaging containing the .pkg
  - Optional DMG signing (Developer ID Application) and notarization + stapling
  - Output directory, volume name, and identity selection UI
  - Credential modes: Apple ID/App Password, Keychain Profile, ASC API Key
- Integrated validation with clear recovery guidance
- Status, step-by-step progress, and results with "Reveal in Finder"

### Distribution Workflow Enhancements (App and PKG)
- Unified Distribution Workflow entry:
  - If selection contains .app → App Distribution Workflow dialog
  - If selection contains .pkg → PKG Distribution Workflow dialog
  - Credentials gating prompts for missing auth then resumes your intended flow
- Intelligent pre-flight validation and skip of redundant steps
- Improved progress weights and user feedback

### Notarization Credentials & Validation
- Unified credentials UI with three modes:
  - Apple ID + App-Specific Password
  - Keychain Profile (xcrun notarytool store-credentials)
  - App Store Connect API Key (.p8)
- "Test Profile" validation for Keychain Profile and improved error feedback
- Comprehensive Notarization Requirements Validation (Detailed mode) with category scoring

### UX and Reliability
- Flow gating: sensitive actions bring up credentials sheets when needed
- Clearer help and status strings across dialogs/toolbars
- Expanded results displays with friendly summaries, Request IDs, and stapling CTAs

---

## Core Features

- **Advanced Code Signing:** In-place or copy-based signing with Developer ID certificates, full hardened runtime support, and extended attributes cleaning
- **Intelligent Notarization Requirements Validation:** Pre-submission analysis with detailed requirement checks and actionable recommendations
- **Working Folders Project Management:** Organize files into projects with persistent state tracking and batch operations
- **Professional DMG Creation:** Customizable disk images with Applications alias, background images, window layouts, and multiple compression formats
- **App Distribution Workflow:** End-to-end .app signing, notarization, stapling, and DMG packaging in one automated workflow
- **PKG Distribution Workflow:** Complete .pkg signing, notarization, and optional DMG packaging with Developer ID Installer certificates
- **Apple Notarization Integration:** Seamless submission to Apple's notarization service with multiple credential modes and automatic status tracking
- **Advanced Persistent Logging:** Comprehensive operation logs with command-line details, Apple responses, and timestamped audit trails
- **Certificate Management:** Automatic discovery and validation of Developer ID certificates with expiration tracking
- **File Analysis & Verification:** Deep inspection of signatures, entitlements, notarization status, and Gatekeeper assessment
- **Professional UI & UX:** Native SwiftUI interface with drag-and-drop, real-time feedback, and accessibility support

---

## Screenshots & Interface Overview

- Main Interface
- Notarization Requirements Validation
- Working Folders Management
- DMG Creation Interface
- App Distribution Workflow
- PKG Distribution Workflow
- Enhanced Logging System
- Extended Attributes Management
- Certificate Management
- Advanced Timing (Polling & Stapling)

---

## How‑To: App Distribution (End‑to‑End)

- Select your .app in Signaro and choose your Developer ID Application certificate.
- Click Distribution Workflow (toolbar or Distribute… in controls).
- In the dialog:
  - If not already signed with the selected cert, Signaro will sign in place (hardened runtime expected).
  - Submit to Apple for notarization (choose your credential mode).
  - Optionally wait for result, then staple automatically when accepted.
  - Create a distribution DMG with Applications alias and brand‑friendly layout.
  - Sign the DMG (Developer ID Application) and optionally notarize + staple the DMG.
- Results include Request IDs, durations, and "Reveal in Finder."

**Tips:**
- Already signed/notarized artifacts are detected to skip unnecessary steps.
- Files that are already stapled will be recognized and skipped gracefully (v3.5+).
- For CI or team usage, prefer Keychain Profile or ASC API Key credentials.

---

## How‑To: PKG Distribution (End‑to‑End)

- Select your .pkg and choose your Developer ID Installer certificate.
- Click Distribution Workflow (toolbar) or Distribute… in controls.
- In the dialog:
  - productsign signs the .pkg (copy-based).
  - Optional notarize + wait + staple the .pkg.
  - Optional create a DMG containing the signed .pkg.
  - Optional DMG signing (Developer ID Application) and notarization + stapling.
  - Choose output folder and volume name; optionally notarize DMG too.
- Results include Request IDs and "Reveal in Finder."

**Tips:**
- .pkg rarely needs extended attributes cleaning; keep it off unless needed.
- Use DMG signing + notarization if you want the final container to carry a ticket too.

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
- **Get:** App Store Connect → Users and Access → Keys → API Keys → Generate → download .p8; note Key ID (KID) and Issuer ID.
- **Use:** CI/CD, headless agents, or restricted accounts where Apple ID shouldn't be used. Provides clean rotation and separation.

### Best‑Practice Examples
- **Indie dev, manual:** Apple ID + App Password is fine for occasional runs.
- **Team/CI on a Mac build agent:** Keychain Profile for reliability with notarytool.
- **Enterprise CI/CD (GitHub Actions, Jenkins):** ASC API Key with least‑privilege ASC role, store .p8 securely, rotate regularly.

---

## Troubleshooting: Common Log Messages

- **"code object is not signed at all" / "valid=false":** Sign appropriately and re-verify
- **"In architecture: x86_64/arm64":** Informational message about binary architectures
- **IdentityManager filtered certificate:** A non-applicable certificate type was ignored
- **RenderBox/ViewBridge messages:** Benign system logs from Xcode debugger
- **"Unable to obtain a task name port right":** Benign Xcode debugger noise, safe to ignore
- **"ViewBridge to RemoteViewService Terminated":** Normal system sheet/dialog close event
- **Stapler error 65:** Raise grace wait or retry backoff (see Advanced Timing)
- **"File is already stapled!":** This is now handled gracefully in v3.5+ (workflow treats as success)

### Known Console Messages (Safe to Ignore)
These messages appear in Xcode's console during normal operation and don't affect functionality:
- `Unable to obtain a task name port right for pid XXX: (os/kern) failure (0x5)` - Xcode debugger inspection
- `ViewBridge to RemoteViewService Terminated: Error Domain=com.apple.ViewBridge Code=18` - Normal dialog dismissal
- `IdentityManager: Successfully loaded X code signing identities` (multiple times) - SwiftUI view refreshes

---

## System Requirements

- macOS 13.5 or later
- Xcode Command Line Tools
- Apple Developer Account
- Keychain Access permissions
- Internet connection for notarization

---

- [GitHub](https://github.com/hov172)
- [PowerShell Gallery](https://www.powershellgallery.com/profiles/hov172)
- 📨 Slack: **@Hov172**
- 🕹️ Discord: **Jay172_**
- [LinkedIn](https://www.linkedin.com/in/jesus-a-785bb616?trk=people-guest_people_search-card)
- 🐦 [Twitter / X (@AyalaSolutions)](https://twitter.com/AyalaSolutions)
- <a href="https://bsky.app/profile/ayalasolutions.bsky.social"><img src="https://raw.githubusercontent.com/bluesky-social/social-app/main/assets/logo.png" width="20" alt="Bluesky Logo"></a> [@AyalaSolutions](https://bsky.app/profile/ayalasolutions.bsky.social)
- 📧 *Contact via GitHub, Social accounts issues or discussions*.

---

## Version Information

- **Current Version:** 3.5 Build 1.0
- **Release:** 2025
- **Platform:** macOS (Universal Binary)
- **Architecture:** SwiftUI + MVVM, advanced logging, working folders, professional DMG creation, intelligent validation

---

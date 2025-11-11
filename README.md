# Signaro: Advanced macOS Code Signing & Notarization Utility

Signaro is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, notarizing, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, comprehensive extended attributes cleaning, advanced persistent logging, working folders project management, professional DMG creation, intelligent notarization requirements validation, modular MVVM architecture, and comprehensive code documentation, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

Current Status: Feature-complete core functionality with ongoing enhancements. See roadmap for upcoming features.

---
## What's New in Version 3.2 Build 1.1
### Bug Fixes & UI Improvements
- Fix the UI Layout in Preferences 

## What's New in Version 3.2 Build 1.0

### Bug Fixes & UI Improvements

#### Notarization Credential Dialog Fix
- **Fixed:** App-Specific Password input not enabling "Continue" button
  - **Issue:** When entering Apple ID credentials, typing in the app-specific password field would not activate the "Continue" button, requiring users to switch tabs (to Keychain Profile and back to Apple ID) as a        workaround
  - **Root Cause:** Missing proper `onChange` handler integration for password field validation
  - **Resolution:** Implemented proper `onChange` handler with immediate validation state updates to enable the "Continue" button as soon as all required fields are filled
  - **Impact:** Smoother credential entry experience with immediate UI feedback, eliminating the need for the tab-switching workaround
 
## What's New in Version 3.1 Build 1.5

### Stability, Concurrency, and Build Reliability
- Reworked process execution to a single shared `ProcessRunner` actor and removed scattered runner references. All shell interactions now go through `await ProcessRunner.shared.run(...)` for clarity and consistency.
- Introduced a safe, lock‑backed output collector to satisfy Swift 6 `@Sendable` concurrency requirements and eliminate data races and warnings (no more “mutation of captured var in concurrently‑executing code”).
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


## What's New in Version 2.8 Build 1.0
- Bugfix

### New: PKG Distribution Workflow
- End-to-end workflow tailored for .pkg installers:
  - Signs .pkg with Developer ID Installer (productsign)
  - Optional notarization and stapling
  - Optional DMG packaging containing the .pkg
  - Optional DMG signing (Developer ID Application) and notarization + stapling
  - Output directory, volume name, and identity selection UI
  - Credential modes: Apple ID/App Password, Keychain Profile, ASC API Key
- Integrated validation with clear recovery guidance
- Status, step-by-step progress, and results with “Reveal in Finder”

### Distribution Workflow Enhancements (App and PKG)
- Unified Distribution Workflow entry:
  - If selection contains .app → App Distribution Workflow dialog
  - If selection contains .pkg → PKG Distribution Workflow dialog
  - Credentials gating prompts for missing auth then resumes your intended flow
- Intelligent pre-flight validation and skip of redundant steps
- Improved progress weights and user feedback

  ## Quick Reference for Development

- Distribution Workflow (App): AppDistributionWorkflow.*
- Distribution Workflow (PKG): PkgDistributionWorkflow.*
- Notarization credentials UI: NotarizationDialogs.swift; credential testing: NotarytoolCredentialTester.swift
- Help and Guides: BasicViews.swift (HelpSheet, NotarizationSetupGuide)
- Toolbar route for both app and pkg: MainContentToolbar.swift; inline Distribute…: SectionViews.swift

---

### Notarization Credentials & Validation
- Unified credentials UI with three modes:
  - Apple ID + App-Specific Password
  - Keychain Profile (xcrun notarytool store-credentials)
  - App Store Connect API Key (.p8)
- “Test Profile” validation for Keychain Profile and improved error feedback
- Comprehensive Notarization Requirements Validation (Detailed mode) with category scoring

### UX and Reliability
- Flow gating: sensitive actions bring up credentials sheets when needed
- Clearer help and status strings across dialogs/toolbars
- Expanded results displays with friendly summaries, Request IDs, and stapling CTAs

---

## Core Features

- Advanced Code Signing
- Intelligent Notarization Requirements Validation
- Working Folders Project Management
- Professional DMG Creation
- App Distribution Workflow
- PKG Distribution Workflow (New)
- Apple Notarization Integration
- Advanced Persistent Logging
- Certificate Management
- File Analysis & Verification
- Professional UI & UX

(See earlier section details)

---

## Screenshots & Interface Overview

- Main Interface
- Notarization Requirements Validation
- Working Folders Management
- DMG Creation Interface
- App Distribution Workflow
- PKG Distribution Workflow (New)
- Enhanced Logging System
- Extended Attributes Management
- Certificate Management
- Advanced Timing (Polling & Stapling)

### How‑To: App Distribution (End‑to‑End)

- Select your .app in Signaro and choose your Developer ID Application certificate.
- Click Distribution Workflow (toolbar or Distribute… in controls).
- In the dialog:
  - If not already signed with the selected cert, Signaro will sign in place (hardened runtime expected).
  - Submit to Apple for notarization (choose your credential mode).
  - Optionally wait for result, then staple automatically when accepted.
  - Create a distribution DMG with Applications alias and brand‑friendly layout.
  - Sign the DMG (Developer ID Application) and optionally notarize + staple the DMG.
- Results include Request IDs, durations, and “Reveal in Finder.”

Tips
- Already signed/notarized artifacts are detected to skip unnecessary steps.
- For CI or team usage, prefer Keychain Profile or ASC API Key credentials.

### How‑To: PKG Distribution (End‑to‑End)

- Select your .pkg and choose your Developer ID Installer certificate.
- Click Distribution Workflow (toolbar) or Distribute… in controls.
- In the dialog:
  - productsign signs the .pkg (copy-based).
  - Optional notarize + wait + staple the .pkg.
  - Optional create a DMG containing the signed .pkg.
  - Optional DMG signing (Developer ID Application) and notarization + stapling.
  - Choose output folder and volume name; optionally notarize DMG too.
- Results include Request IDs and “Reveal in Finder.”

Tips
- .pkg rarely needs extended attributes cleaning; keep it off unless needed.
- Use DMG signing + notarization if you want the final container to carry a ticket too.

### Notarization Credentials — Modes, How to Get Them, and When to Use Each

Credential Modes (all supported)
- Apple ID + App-Specific Password
  - Get: appleid.apple.com → Security → App-Specific Passwords → Generate.
  - Use: Personal workflows, quick manual submissions.
- Keychain Profile (notarytool store-credentials)
  - Get: xcrun notarytool store-credentials PROFILE --key AuthKey_XXXX.p8 --key-id <KID> --issuer <ISSUER>.
  - Use: Day-to-day local development and CI on a machine with a login keychain; easiest to “just work” repeatedly.
- App Store Connect API Key (.p8)
  - Get: App Store Connect → Users and Access → Keys → API Keys → Generate → download .p8; note Key ID (KID) and Issuer ID.
  - Use: CI/CD, headless agents, or restricted accounts where Apple ID shouldn’t be used. Provides clean rotation and separation.

Best‑Practice Examples
- Indie dev, manual: Apple ID + App Password is fine for occasional runs.
- Team/CI on a Mac build agent: Keychain Profile for reliability with notarytool.
- Enterprise CI/CD (GitHub Actions, Jenkins): ASC API Key with least‑privilege ASC role, store .p8 securely, rotate regularly.

---

## Troubleshooting: Common Log Messages

- “code object is not signed at all” / “valid=false”: sign appropriately and re-verify
- “In architecture: x86_64/arm64”: informational
- IdentityManager filtered certificate: a non-applicable type was ignored
- RenderBox/ViewBridge messages: benign system logs
- Stapler error 65: raise grace wait or retry backoff (see Advanced Timing)

---

## System Requirements

- macOS 13.5+
- Xcode Command Line Tools
- Apple Developer Account
- Keychain Access permissions
- Internet connection

---

## Version Information

- Current Version: 3.2 Build 1.1
- Release: 2025
- Platform: macOS (Universal Binary)
- Architecture: SwiftUI + MVVM, advanced logging, working folders, professional DMG creation, intelligent validation

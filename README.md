# Signaro: Advanced macOS Code Signing & Notarization Utility

Signaro is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, notarizing, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, comprehensive extended attributes cleaning, advanced persistent logging, working folders project management, professional DMG creation, intelligent notarization requirements validation, modular MVVM architecture, and comprehensive code documentation, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

Current Status: Feature-complete core functionality with ongoing enhancements. See roadmap for upcoming features.

---

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

- Current Version: 2.6 Build 2.0
- Release: 2025
- Platform: macOS (Universal Binary)
- Architecture: SwiftUI + MVVM, advanced logging, working folders, professional DMG creation, intelligent validation

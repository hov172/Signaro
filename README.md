# Signaro: Advanced macOS Code Signing & Notarization Utility

**Signaro** is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, **notarizing**, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, **comprehensive extended attributes cleaning**, **advanced persistent logging system**, **working folders project management**, **professional DMG creation**, **intelligent notarization requirements validation**, **modular MVVM architecture for enhanced performance**, and **comprehensive code documentation**, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

> **Current Status**: Feature-complete core functionality with ongoing enhancements. See [roadmap](#roadmap) for upcoming features.

---

## What's New in Version 2.6 Build 1.1

### Reliability and Bug Fixes
- Resolved intermittent crashes during batch unsign/sign operations on nested bundles
- Fixed an issue where file list selections could desync after rapid add/remove actions
- Corrected notarization status parsing for certain stapled apps where spctl output varied by macOS version
- Fixed DMG creation failures when background images contained spaces or non-ASCII characters
- Eliminated duplicate working folder entries caused by rapid re-importing of the same paths

### UX and Accessibility
- Improved confirmation dialogs with clearer action verbs and consistent button ordering
- Better keyboard navigation across toolbar and dialogs, plus refined VoiceOver labels
- Streamlined error messages with actionable next steps and compact technical detail sections
- Smarter certificate picker hints that reflect current selection context and file types

### Performance and Concurrency
- Faster large-batch analysis through improved task scheduling and cancellation behavior
- Reduced memory footprint during extended operations and long-running sessions
- Optimized log viewer loading and search performance for large log archives
- Improved responsiveness of progress indicators and list updates under heavy load

### Workflow Fixes and Options
- New batch options for all workflows:
  - Skip already processed items (already signed/notarized/stapled)
  - Auto-staple after notarization
  - Auto-create DMG after successful stapling with selectable layout presets
  - Validate-only dry run (no signing/notarization changes)
  - Per-run override for extended attributes cleaning
- Queue management improvements: reliable cancel, retry, and resume with accurate progress
- Fixed stuck states after cancellations, notarytool timeouts, or transient network errors
- Correct toolbar/state enablement during long-running workflows and after mode toggles
- Accurate progress totals and completion banners for mixed-success batches
- Working folders: consistent batch counts, proper aggregation across nested items, and safer re-run behavior

### Signing and Notarization Enhancements
- More resilient stapling and verification workflow with automatic fallback checks
- Hardened Runtime and entitlements checks provide clearer recommendations in validation results
- Better pre-flight extended attributes handling for higher notarization success rates
- Enhanced certificate trust validation with clearer indicators for untrusted/expired identities
- New Advanced Timing controls for notarization polling and stapler backoff with presets (Aggressive, Balanced, Conservative/CI) and a Reset Defaults option

### DMG Creation and Distribution
- Safer volume naming and path sanitation to prevent hdiutil edge-case failures
- Improved Applications alias handling and layout persistence for distribution DMGs
- Clearer guidance for encryption/segmentation options with validation before execution

### Logging and Diagnostics
- Richer metadata in operation logs, including aggregated batch summaries
- More robust credential masking and sensitive data scrubbing
- Additional validation and distribution analytics exposed in log summaries
- Documentation for common benign system log messages (e.g., RenderBox/ViewBridge) to reduce confusion

---

## Core Features

### Advanced Code Signing
- **Batch sign & unsign** multiple files simultaneously with progress tracking
- **Smart certificate selection** with trust status verification and compatibility checking
- **In-place signing** for .app bundles (preserves original location and metadata)
- **Copy-based signing** for .pkg and .mobileconfig files (maintains originals)
- **Extended attributes cleaning** automatically removes problematic quarantine attributes, resource forks, and Finder metadata
- **Real-time signature verification** with detailed status reporting and error diagnosis
- **Comprehensive error handling** with recovery suggestions and troubleshooting guidance

### Intelligent Notarization Requirements Validation
- **Pre-flight validation system** — Comprehensive analysis of files before notarization submission to identify potential issues
- **Dual validation modes** offering both Quick (basic checks) and Detailed (comprehensive analysis) validation options
- **Smart readiness assessment** with percentage-based scoring and clear visual indicators of notarization preparedness
- **Professional results presentation** with summary banners and expandable details for technical deep-dive analysis
- **Already-notarized detection** — Intelligent identification of files that have already been notarized with appropriate next-step recommendations
- **Certificate compatibility validation** ensuring selected certificates are appropriate for the specific file types being processed
- **Interactive requirements checking** with one-click validation and real-time results for immediate feedback
- **Educational validation assistance** — Built-in explanations of Apple's notarization requirements and validation mode differences
- **Batch validation support** for simultaneous assessment of multiple files with aggregated project-level results
- **Validation analytics and logging** — Complete tracking of validation operations with detailed metrics and professional reporting

### Working Folders Project Management
- **Project-based file organization** — Group related files into working folders for unified management and batch operations
- **Persistent project storage** with automatic status tracking and progress monitoring across all files in each project
- **Smart organization tools** with auto-suggested folder names based on file locations and naming patterns
- **Batch operations** perform signing, notarization, analysis, and DMG creation on entire working folders
- **Project status aggregation** showing overall readiness, completion status, and workflow progress for entire projects
- **Export configuration** functionality for sharing working folder setups and project organization with team members
- **Comprehensive audit trails** with all working folder operations automatically logged for enterprise compliance
- **Flexible workflow modes** easily toggle between individual file management and project-based organization
- **Working folder creation wizard** with intelligent file grouping suggestions and project naming recommendations
- New in 2.6:
  - Skip already processed items
  - Auto-staple and optional auto-create DMG with preset layouts
  - Validate-only dry run mode
  - Per-run override for extended attributes cleaning

### Professional DMG Creation
- **Complete DMG creation workflow** supporting multiple formats (compressed, read/write, read-only, highly compressed)
- **Advanced DMG customization** including AES encryption (128/256-bit), file segmentation, and custom background images
- **App distribution DMGs** with professional drag-and-drop layouts including Applications folder aliases
- **Multi-file DMG creation** from selected file collections with automatic staging and intelligent organization
- **Blank DMG creation** for working disk images with user-specified sizes and format options
- **Professional layout customization** with custom icon positioning, window sizing, and branding elements
- **Intelligent volume naming** with automatic name generation based on source content and distribution requirements
- **Size optimization** with automatic size estimation, compression recommendations, and space efficiency analysis
- **Complete operation logging** with detailed DMG creation logs including command-line calls, processing times, and file sizes
- **Enterprise encryption support** with password protection using industry-standard AES encryption for sensitive distributions
- New in 2.6:
  - Selectable layout presets for auto-created DMGs

### App Distribution Workflow
- **End-to-end distribution preparation** from source app to ready-to-distribute DMG with complete signing and notarization
- **Automated workflow orchestration** combining signing, notarization, stapling, and professional DMG creation in a single streamlined process
- **Distribution readiness validation** ensuring apps meet all Apple distribution requirements before final packaging
- **Professional DMG layouts** with Applications folder shortcuts, custom backgrounds, and enterprise branding support
- **Workflow templates** for common distribution scenarios (Mac App Store preparation, direct distribution, enterprise deployment)
- **Comprehensive progress tracking** with detailed status updates throughout the entire distribution preparation process
- **Multi-stage validation** checking code signing status, notarization completion, and distribution compliance requirements
- **One-click distribution preparation** from unsigned app to distribution-ready DMG with complete audit logging and professional presentation
- New in 2.6:
  - Skip already notarized/stapled items
  - Auto-staple and optional auto-create DMG with preset layouts
  - Validate-only dry run mode
  - Per-run extended attributes cleaning override

### Universal Extended Attributes Management
- **Unified cleaning system** — Works seamlessly with individual files, working folders, and DMG creation workflows
- **Automatic cleaning** of extended attributes before signing operations to prevent common failure scenarios
- **Pre-notarization cleaning** removes problematic attributes before submission to Apple's servers for higher success rates
- **DMG preparation cleaning** ensures clean file metadata before disk image creation to prevent mounting issues
- **User preference control** to enable/disable cleaning with transparent operation logging and detailed reporting
- **Advanced diagnostic capabilities** to analyze files and working folders for potentially problematic attributes
- **Manual cleaning tools** for troubleshooting and specialized workflow preparation requirements
- **Support for quarantine attributes** (`com.apple.quarantine`) commonly added to downloaded files and archives
- **Resource fork cleanup** removing legacy Mac file system artifacts that interfere with modern operations
- **Finder metadata removal** eliminating extended attributes that can cause signing, notarization, and DMG creation conflicts
- **Enhanced success rates** for individual files, working folder projects, and complete distribution workflows
- New in 2.6:
  - Per-run override for extended attributes cleaning

### Apple Notarization Integration
- **Seamless notarization workflow** with Apple's notary service and automated submission
- **Secure credential management** with validation, encrypted storage, and session handling
- **Extended attributes cleaning** automatically applied before notarization submission (when enabled)
- **Advanced status checking** for any notarization request ID with detailed progress monitoring
- **Automatic stapling** after successful notarization with integrity verification
- **Comprehensive notarization reporting** with detailed logs, timestamps, and audit trails
- **Batch notarization support** with queue management and progress tracking
- **Pre-flight validation** with extended attributes assessment and cleaning recommendations
- **Complete operation logging** with Apple responses, Request UUIDs, and comprehensive metadata automatically captured
- New in 2.6:
  - Skip already notarized items
  - Auto-staple and optional auto-create DMG with preset layouts
  - Validate-only dry run mode
  - Per-run extended attributes cleaning override

### Advanced Persistent Logging
- **Comprehensive operation tracking** — All signing, notarization, stapling, analysis, working folder, DMG creation, and validation operations automatically logged
- **Complete submission details** — Full command-line calls, Apple responses, Request UUIDs, exit codes, and processing times
- **Rich metadata collection** including file checksums, device information, operation context, working folder configurations, validation results, and detailed timestamps
- **Daily log rotation** with automatic cleanup and size management (30-day retention, 10MB file limit)
- **Advanced log viewer** with search functionality, filtering, and professional export capabilities
- **Security-focused design** with automatic credential masking and secure storage in ~/Documents/Signaro Logs/
- **Thread-safe logging** ensuring data integrity during concurrent operations and batch processing
- **Audit trail generation** perfect for enterprise compliance, debugging, and Apple Support communication
- **Export functionality** with professional formatting for sharing with teams or Apple Support
- **Log statistics dashboard** showing success rates, operation trends, historical analytics, validation metrics, and workflow performance
- **Instant access** via toolbar "View Logs" button for immediate troubleshooting and operation history

### Certificate Management
- **Automatic keychain discovery** of code signing certificates with trust validation
- **Trust status indicators** (trusted/untrusted certificates) with detailed information
- **Certificate type identification** (Developer ID, Apple Development, etc.) with usage guidance
- **Expiration date monitoring** with proactive alerts and working folder impact assessment
- **Certificate information dialogs** with comprehensive details and troubleshooting tips
- **Keychain integration** with secure access and permission handling

### File Analysis & Verification
- **Multi-format support**: `.pkg`, `.app`, `.mobileconfig`, `.dmg` with extensible architecture
- **Signature validation** with complete authority chain verification and trust assessment
- **Team ID extraction** and consistency checking across file collections
- **Notarization status verification** for all applicable file types with detailed reporting
- **Extended attributes analysis** with identification of potentially problematic metadata
- **File icon display** using system icons with type identification badges
- **Security posture assessment** with vulnerability detection and compliance checking

### Professional UI & UX
- **Modern SwiftUI interface** with native macOS design patterns and animations
- **Modular MVVM architecture** for enhanced performance and maintainability
- **Comprehensive accessibility support** with VoiceOver compatibility and keyboard navigation
- **Intelligent keyboard shortcuts** for efficient workflow and power user features
- **Responsive layout design** with resizable windows and adaptive content
- **Dark mode support** following system preferences with high contrast compatibility
- **Professional visual design** with consistent spacing, typography, and color schemes

---

## Screenshots & Interface Overview

### Main Interface
*[Screenshot: Main application window showing complete workflow with notarization validation]*
- Notarization requirements validation with dual-mode selection (Quick/Detailed)
- Professional validation results banner with expandable details and readiness scoring
- Working folders mode toggle for switching between individual and project-based workflows
- Certificate selection dropdown with intelligent recommendations and extended attributes cleaning toggle
- File list with drag & drop support, working folder organization, and sorting options
- Operation buttons (Sign, Unsign, Analyze, Notarize, Check Requirements, Create DMG) with status indicators
- Real-time file signature analysis and validation results
- **"Check Requirements" button in toolbar** for instant notarization readiness assessment
- **Working folder status banner** showing project-level progress and completion status
- **"View Logs" button in toolbar** for instant access to comprehensive operation history

### Notarization Requirements Validation
*[Screenshot: Validation interface showing comprehensive requirements analysis]*
- **Validation mode selector** with Quick (minimal, fast) and Detailed (comprehensive) options
- **Professional results banner** with visual readiness indicators and percentage scoring
- **Expandable details interface** showing critical issues, recommendations, and technical analysis
- **Already-notarized detection** with appropriate next-step guidance
- **Certificate compatibility assessment** with specific recommendations for file types
- **Interactive help system** explaining validation modes and Apple requirements
- **Progress indicators** during validation operations with real-time status updates

### Working Folders Management
*[Screenshot: Working folders interface showing project organization]*
- **Working folder creation dialog** with intelligent naming suggestions and file grouping
- **Project status dashboard** showing overall progress across all working folders
- **Batch operation controls** for performing operations on entire working folders
- **Working folder file list** with project-specific file organization and status tracking
- **Export configuration** options for sharing working folder setups with team members
- **Working folder statistics** showing completion rates, file counts, and project health

### DMG Creation Interface
*[Screenshot: DMG creation dialog with advanced options]*
- **DMG format selection** with detailed descriptions and use case recommendations
- **Advanced options panel** including encryption, segmentation, and custom backgrounds
- **Volume naming** with intelligent suggestions based on source content
- **App distribution options** with Applications folder alias and layout customization
- **Progress tracking** during DMG creation with detailed status updates
- **Professional customization** options for enterprise branding and presentation
- New in 2.6:
  - Selectable layout presets for auto-created DMGs

### App Distribution Workflow
*[Screenshot: Complete distribution workflow from app to DMG]*
- **End-to-end workflow orchestration** showing all steps from signing to DMG creation
- **Distribution readiness validation** with comprehensive requirement checking
- **Workflow progress tracking** with detailed status updates at each stage
- **Professional DMG preview** showing final distribution layout and presentation
- **One-click distribution preparation** with complete automation and logging
- New in 2.6:
  - Skip already notarized/stapled items
  - Auto-staple and optional auto-create DMG with preset layouts
  - Validate-only dry run mode
  - Per-run extended attributes cleaning override

### Enhanced Logging System
*[Screenshot: Advanced log viewer showing comprehensive operation history with validation logging]*
- **Log file browser** with chronological organization, operation type filtering, and metadata display
- **Rich log content** showing complete command-line calls, Apple responses, Request UUIDs, validation results, and working folder operations
- **Enhanced search and filtering** capabilities for finding specific operations, projects, validation results, or troubleshooting issues
- **Export functionality** for sharing logs with Apple Support or team collaboration
- **Statistics dashboard** showing success rates, operation counts, working folder analytics, validation metrics, and DMG creation statistics
- **Professional formatting** suitable for enterprise documentation and compliance reporting

### Extended Attributes Management
*[Screenshot: Extended attributes diagnostic with working folder support]*
- **Individual and batch file analysis** with working folder-level extended attributes assessment
- **Problematic attribute identification** with warning indicators and batch cleaning recommendations
- **Manual cleaning capabilities** with working folder batch processing options
- **Before/after comparison** showing cleaning operation results across multiple files
- **Universal integration** — Same cleaning system works for signing, notarization, and DMG creation
- New in 2.6:
  - Per-run override for extended attributes cleaning

### Certificate Management
*[Screenshot: Certificate selection with working folder context]*
- **Available certificates** from keychain with trust status indicators and working folder compatibility
- **Certificate type identification** (Developer ID, Apple Development, etc.) with project-specific recommendations
- **Extended attributes cleaning preference** with explanatory text and working folder integration
- **Expiration date monitoring** and renewal reminders with project impact assessment
- **Intelligent certificate recommendations** based on selected files and working folder requirements

### Advanced Timing (Polling & Stapling)

Fine‑tune how Signaro polls Apple for notarization results and how it retries stapling when tickets are still propagating (e.g., stapler error 65). Use the presets below or customize per your environment.

### Recommended Presets

- Balanced (default; good for most users)
  - Poll Interval: 30s
  - Max Wait: 10m
  - Grace Wait (after Acceptance): 60s
  - Stapling Backoff: Base 30s, Jitter 10s, Max Attempts 8, Max Delay 180s

- Aggressive (faster feedback for single items/small batches on stable networks)
  - Poll Interval: 15s
  - Max Wait: 8m
  - Grace Wait: 30s
  - Stapling Backoff: Base 15s, Jitter 5s, Max Attempts 5, Max Delay 90s

- Conservative / CI (large batches, flaky networks, or peak Apple load)
  - Poll Interval: 45s
  - Max Wait: 20m
  - Grace Wait: 120s
  - Stapling Backoff: Base 45s, Jitter 15s, Max Attempts 10, Max Delay 240s

### What Each Setting Does

- Poll Interval
  - How often Signaro queries Apple for status of a notarization Request UUID
  - Lower = faster detection but more API calls; higher = gentler on rate limits

- Max Wait
  - Upper bound for waiting on “Accepted” before timing out the polling cycle

- Grace Wait (after Acceptance)
  - Extra delay after “Accepted” to let Apple’s ticket propagate before stapling
  - Helps avoid intermittent stapler error 65 (“Ticket not found yet”)

- Stapling Backoff Policy (applies to retryable errors like 65)
  - Base Delay: initial retry wait; grows exponentially each attempt
  - Jitter: randomization added to spreads out retries in batches/CI
  - Max Attempts: total retry tries before failing
  - Max Delay: cap on any single retry sleep

### When To Adjust

- Frequent stapler 65s after “Accepted”: increase Grace to 90–120s and/or raise Max Attempts to 10 and Max Delay to 240s
- Very large CI batches: lengthen Poll Interval (45s), increase Jitter (15s), raise Attempts (10)
- Need quick feedback for single items: use the Aggressive preset

### Notes

- Reset Defaults restores the Balanced preset values
- Short Poll Intervals across large batches can hit rate limits—prefer Balanced/Conservative for team/CI workloads
- These settings affect behavior only after a notarization submission; they don’t change Apple’s actual processing time

---

## Troubleshooting: Common Log Messages

This section explains frequent messages you may see in Signaro’s debug logs or macOS Console, what they mean, and typical actions.

- "code object is not signed at all", "Authority lines: []", or "codesign --verify result: valid=false, terminationStatus=1"
  - Meaning: The app/bundle is currently unsigned, or the signature is missing/invalid.
  - Action: Sign with a Developer ID Application certificate (for .app). For installers (.pkg), use a Developer ID Installer certificate. Consider enabling extended attributes cleaning before signing, then re-verify and proceed to notarization.

- "In architecture: x86_64" (or arm64)
  - Meaning: codesign reported which architecture it checked during verification.
  - Action: Informational only. For universal binaries you may see multiple architecture notes.

- "IdentityManager: Filtering out certificate: Package Signing … (include: false, exclude: false)"
  - Meaning: A certificate type not applicable to the current operation was ignored (e.g., an Installer/Package certificate when working with an .app).
  - Action: Choose an appropriate certificate type for the file you are processing (Developer ID Application for .app, Developer ID Installer for .pkg).

- "Unable to open mach-O at path: … RenderBox.framework … default.metallib Error:2"
  - Meaning: A benign system message from Apple’s internal RenderBox/metal tooling while macOS renders or previews views.
  - Action: Safe to ignore. It does not affect signing or notarization outcomes.

- "ViewBridge to RemoteViewService Terminated … NSViewBridgeErrorCanceled"
  - Meaning: A remote view (e.g., a system panel or sheet) was dismissed/canceled; macOS logs the disconnect.
  - Action: Benign and expected when closing dialogs; no action required.

- Frequent stapler error 65 ("ticket not found yet") after notarytool reports "Accepted"
  - Meaning: Apple’s ticket may not have fully propagated to all CDNs when stapling first runs.
  - Action: Increase Grace Wait to 90–120s and/or raise Stapling Backoff Max Attempts and Max Delay. See Advanced Timing (Polling & Stapling) for recommended presets.

---

## System Requirements

- **macOS 13.5** or later (Universal Binary support for Apple Silicon and Intel)
- **Xcode Command Line Tools** (for codesign, pkgutil, notarytool, xattr, hdiutil, and stapler integration)
- **Apple Developer Account** (for notarization and distribution certificates)
- **Keychain Access** to code signing certificates with proper permissions
- **Internet Connection** (for notarization operations and certificate validation)

---

## Version Information

- **Current Version**: 2.6 Build 1.1
- **Release**: 2025
- **Platform**: macOS (Universal Binary - Apple Silicon & Intel)
- **License**: Free
- **Support**: Community-driven with comprehensive documentation
- **Architecture**: Modern SwiftUI with modular MVVM architecture, enterprise-grade code documentation, advanced logging system, working folders, professional DMG creation, and intelligent validation systems

---

## Alternative Tools

For command-line workflows, consider:
- [profilesigner](https://github.com/nmcspadden/ProfileSigner) — CLI alternative for configuration profiles
- Apple's native `codesign`, `pkgutil`, `xattr`, `hdiutil`, and `notarytool` commands

---

Choose Signaro for modern, intelligent, and secure code signing, notarization, and distribution on macOS!

---

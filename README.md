# Still testing - Let me know of issues.

# Signaro: Advanced macOS Code Signing & Notarization Utility

**Signaro** is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, **notarizing**, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, **comprehensive extended attributes cleaning**, **advanced persistent logging system**, **working folders project management**, **professional DMG creation**, and **comprehensive code documentation**, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

> **Current Status**: Feature-complete core functionality with ongoing enhancements. See [roadmap](#roadmap) for upcoming features.

---

## What's New in Version 2.5 Build 1.1

### Bug Fixes & Improvements
- **Fixed file removal functionality** — Red X buttons now properly show confirmation dialogs and remove files from selection
- **Enhanced alert system** — Added comprehensive alert handling for file removal confirmations and general notifications
- **Improved state management** — File removal now properly cleans up all associated data including signatures, analysis results, and UI state
- **Better user feedback** — Clear confirmation dialogs with file count display and proper cancel/confirm options
- **Automatic UI updates** — File analysis and status information automatically refreshes after file removal operations

### Technical Improvements
- **Complete alert system implementation** — Added missing alert modifiers for file removal and general notifications
- **Enhanced data cleanup** — File removal operations now properly clean up fileSignatures, fileExpandedState, and selectedForRemoval collections
- **Improved error handling** — Better user feedback and state management throughout the application
- **Consistent UI behavior** — File removal operations now maintain consistent behavior across all interface components

### Working Folders System
- **Project-based organization** — Group related files together into working folders for batch operations and project management
- **Persistent folder management** with automatic status tracking across all files in each folder
- **Smart file organization** with auto-suggested folder names based on file locations and naming patterns
- **Batch operations** perform signing, notarization, and analysis operations on entire working folders
- **Working folder status aggregation** showing overall project readiness and completion status
- **Export configuration** functionality for sharing working folder setups with team members
- **Comprehensive logging integration** with all working folder operations automatically logged for audit trails
- **Toggle working folder mode** to switch between individual file management and project-based workflows
- **Working folder creation wizard** with intelligent file grouping suggestions and naming recommendations

### Professional DMG Creation System
- **Complete DMG creation workflow** with support for multiple formats (compressed, read/write, read-only, highly compressed)
- **Advanced DMG options** including encryption (AES-128/256), segmentation for large files, and custom background images
- **App distribution DMGs** with Applications folder aliases and professional drag-and-drop installation layouts
- **Multi-file DMG creation** from selected file collections with automatic staging and organization
- **Blank DMG creation** for working disk images with user-specified sizes
- **Professional layout customization** with custom icon positioning, window sizing, and background images
- **Volume naming intelligence** with automatic name generation based on source content
- **Size calculation and optimization** with automatic size estimation and compression recommendations
- **Complete operation logging** with detailed DMG creation logs including command-line calls and processing times
- **Encryption support** with password protection using industry-standard AES encryption

### App Distribution Workflow
- **End-to-end distribution preparation** from source app to ready-to-distribute DMG with signing and notarization
- **Automated workflow orchestration** combining signing, notarization, stapling, and DMG creation in a single process
- **Distribution readiness validation** ensuring apps meet all Apple distribution requirements before DMG creation
- **Professional DMG layouts** with Applications folder shortcuts and custom branding support
- **Workflow templates** for common distribution scenarios (Mac App Store, direct distribution, enterprise deployment)
- **Progress tracking** with detailed status updates throughout the entire distribution preparation process
- **Comprehensive validation** checking code signing, notarization status, and distribution requirements
- **One-click distribution preparation** from unsigned app to distribution-ready DMG with full logging

### Enhanced Logging System
- **Working folder operation logging** with detailed project-level audit trails and batch operation tracking
- **DMG creation logging** capturing all disk image creation operations with command-line details and processing metrics
- **Distribution workflow logging** tracking complete end-to-end distribution preparation processes
- **Enhanced metadata capture** including working folder configurations, DMG creation parameters, and workflow selections
- **Advanced log organization** with operation-type filtering and project-based log grouping
- **Export improvements** with workflow-specific log exports and enhanced formatting for professional documentation
- **Statistics enhancements** showing distribution workflow success rates, DMG creation metrics, and working folder analytics

### Universal Extended Attributes Management
- **Working folder integration** — Extended attributes cleaning now works seamlessly with working folder batch operations
- **DMG preparation cleaning** — Automatic extended attributes cleaning before DMG creation to prevent common issues
- **Distribution workflow integration** — Extended attributes cleaning integrated into complete app distribution workflows
- **Enhanced diagnostic tools** with working folder-level extended attributes analysis and batch cleaning capabilities
- **Improved success rates** for both individual files and entire working folder projects through consistent attribute management

### Advanced File Analysis
- **Working folder analysis** with project-level status aggregation and batch file analysis capabilities
- **Distribution readiness assessment** evaluating complete app distribution workflows including signing, notarization, and packaging requirements
- **Project-level recommendations** providing guidance for entire working folders rather than just individual files
- **Enhanced certificate analysis** with working folder-level certificate consistency checking and project recommendations
- **Comprehensive workflow validation** ensuring complete distribution workflows meet all Apple requirements

### Intelligent Workflow Management
- **Context-aware recommendations** based on selected files, working folder configurations, and intended distribution methods
- **Workflow templates** for common scenarios (individual file signing, project-based distribution, enterprise deployment)
- **Smart defaults** that adapt based on file types, certificate availability, and project organization
- **Progressive workflow assistance** guiding users through complex multi-step distribution preparation processes
- **Educational workflow modes** helping users learn Apple's distribution requirements and best practices

### Enhanced User Interface
- **Working folders mode toggle** seamlessly switching between individual file management and project-based workflows
- **DMG creation interface** with advanced options, format selection, and professional customization settings
- **Distribution workflow dialogs** providing step-by-step guidance through complete distribution preparation
- **Enhanced file management** with working folder organization, drag-and-drop batch operations, and project status visualization
- **Professional workflow indicators** showing distribution readiness, project completion status, and workflow progress
- **Advanced help system** covering working folders, DMG creation, and complete distribution workflows

---

## What's in Version 2.0 Build 1.7 *(Previous Features)*

### Advanced Persistent Logging System
- **Complete operation history** — Persistent logging of all signing, analysis, and notarization operations with Apple responses
- **Comprehensive submission details** — Full command-line calls, Apple responses, Request UUIDs, and processing times automatically logged
- **Rich metadata capture** including file checksums, device information (macOS version, Xcode version, CPU architecture), and operation context
- **Daily log rotation** with automatic cleanup (30-day retention) and file size management to prevent storage issues
- **Advanced log viewer** with search functionality, export capabilities, and professional formatting for enterprise documentation
- **Secure credential handling** with automatic password masking in all log entries for privacy and security
- **Thread-safe logging** with queue-based operations ensuring data integrity during concurrent operations
- **Comprehensive operation coverage** — Logs signing, unsigning, notarization, stapling, status checks, and Smart Analysis operations
- **Audit trail generation** with detailed timestamps, session IDs, and complete workflow documentation for compliance and debugging
- **Export functionality** for sharing logs with Apple Support or team members with professional formatting
- **Log statistics and analytics** showing success rates, operation counts, and historical trends

### Comprehensive Extended Attributes Cleaning System
- **Universal cleaning integration** — Extended attributes cleaning now works for **both Code Signing AND Apple Notarization** operations
- **Automatic cleaning** of problematic extended attributes before signing and notarization operations
- **User-controlled preference** to enable/disable extended attributes cleaning (enabled by default)
- **Consistent workflow experience** — Same cleaning behavior applies to both local signing and Apple notarization
- **Diagnostic tools** to check and analyze extended attributes that may interfere with signing or notarization
- **Manual cleaning capabilities** for troubleshooting and preparation workflows
- **Comprehensive logging** showing what attributes were cleaned and operation results
- **Prevention of common failures** caused by quarantine attributes, resource forks, and Finder metadata in both signing and notarization workflows
- **Enhanced notarization success rates** by removing problematic attributes before submission to Apple

### Enhanced Operation Results Management
- **Smart results reset** — Operation results automatically clear when files are added or removed from selection
- **Clean UI state** — Results always match current file selection, eliminating confusion
- **Consistent experience** — Results reset applies to both signing operations and notarization workflows
- **Better workflow management** — Encourages users to review results immediately after operations

### Enhanced Smart Analysis System
- **Manual Smart Analysis Button** with immediate comprehensive results (bypasses 3-second auto-analysis delay)
- **Force Refresh Capabilities** that clear all cached data for completely fresh analysis
- **Network-dependent validation** with certificate revocation checking and OCSP verification
- **Certificate compatibility re-analysis** when certificate selection changes
- **Deep debugging support** with enhanced error reporting and diagnostics
- **Educational exploration mode** for learning code signing concepts

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

### App Distribution Workflow
- **End-to-end distribution preparation** from source app to ready-to-distribute DMG with complete signing and notarization
- **Automated workflow orchestration** combining signing, notarization, stapling, and professional DMG creation in a single streamlined process
- **Distribution readiness validation** ensuring apps meet all Apple distribution requirements before final packaging
- **Professional DMG layouts** with Applications folder shortcuts, custom backgrounds, and enterprise branding support
- **Workflow templates** for common distribution scenarios (Mac App Store preparation, direct distribution, enterprise deployment)
- **Comprehensive progress tracking** with detailed status updates throughout the entire distribution preparation process
- **Multi-stage validation** checking code signing status, notarization completion, and distribution compliance requirements
- **One-click distribution** preparation from unsigned app to distribution-ready DMG with complete audit logging and professional presentation

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

### Advanced Persistent Logging
- **Comprehensive operation tracking** — All signing, notarization, stapling, analysis, working folder, and DMG creation operations automatically logged
- **Complete submission details** — Full command-line calls, Apple responses, Request UUIDs, exit codes, and processing times
- **Rich metadata capture** including file checksums, device information, operation context, working folder configurations, and detailed timestamps
- **Daily log rotation** with automatic cleanup and size management (30-day retention, 10MB file limit)
- **Advanced log viewer** with search functionality, filtering, and professional export capabilities
- **Security-focused design** with automatic credential masking and secure storage in ~/Documents/Signaro Logs/
- **Thread-safe logging** ensuring data integrity during concurrent operations and batch processing
- **Audit trail generation** perfect for enterprise compliance, debugging, and Apple Support communication
- **Export functionality** with professional formatting for sharing with teams or Apple Support
- **Log statistics dashboard** showing success rates, operation trends, historical analytics, and workflow performance
- **Instant access** via toolbar "View Logs" button for immediate troubleshooting and operation history

### Certificate Management
- **Automatic keychain discovery** of code signing certificates with trust validation
- **Trust status indicators** (trusted/untrusted certificates) with detailed information
- **Certificate type identification** (Developer ID, Apple Development, etc.) with usage guidance
- **Expiration date monitoring** with renewal reminders and proactive notifications
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
- **Comprehensive accessibility support** with VoiceOver compatibility and keyboard navigation
- **Intelligent keyboard shortcuts** for efficient workflow and power user features
- **Responsive layout design** with resizable windows and adaptive content
- **Dark mode support** following system preferences with high contrast compatibility
- **Professional visual design** with consistent spacing, typography, and color schemes

---

## Screenshots & Interface Overview

### Main Interface
*[Screenshot: Main application window showing complete workflow with working folders]*
- Working folders mode toggle for switching between individual and project-based workflows
- Certificate selection dropdown with intelligent recommendations and extended attributes cleaning toggle
- File list with drag & drop support, working folder organization, and sorting options
- Operation buttons (Sign, Unsign, Analyze, Notarize, Create DMG, Check Attributes) with status indicators
- Real-time file signature analysis and validation results
- **Working folder status banner** showing project-level progress and completion status
- **"View Logs" button in toolbar** for instant access to comprehensive operation history

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

### App Distribution Workflow
*[Screenshot: Complete distribution workflow from app to DMG]*
- **End-to-end workflow orchestration** showing all steps from signing to DMG creation
- **Distribution readiness validation** with comprehensive requirement checking
- **Workflow progress tracking** with detailed status updates at each stage
- **Professional DMG preview** showing final distribution layout and presentation
- **One-click distribution** preparation with complete automation and logging

### Enhanced Logging System
*[Screenshot: Advanced log viewer showing comprehensive operation history with new features]*
- **Log file browser** with chronological organization, operation type filtering, and metadata display
- **Rich log content** showing complete command-line calls, Apple responses, Request UUIDs, and working folder operations
- **Enhanced search and filtering** capabilities for finding specific operations, projects, or troubleshooting issues
- **Export functionality** for sharing logs with Apple Support or team collaboration
- **Statistics dashboard** showing success rates, operation counts, working folder analytics, and DMG creation metrics
- **Professional formatting** suitable for enterprise documentation and compliance reporting

### Extended Attributes Management
*[Screenshot: Extended attributes diagnostic with working folder support]*
- **Individual and batch file analysis** with working folder-level extended attributes assessment
- **Problematic attribute identification** with warning indicators and batch cleaning recommendations
- **Manual cleaning capabilities** with working folder batch processing options
- **Before/after comparison** showing cleaning operation results across multiple files
- **Universal integration** — Same cleaning system works for signing, notarization, and DMG creation

### Certificate Management
*[Screenshot: Certificate selection with working folder context]*
- **Available certificates** from keychain with trust status indicators and working folder compatibility
- **Certificate type identification** (Developer ID, Apple Development, etc.) with project-specific recommendations
- **Extended attributes cleaning preference** with explanatory text and working folder integration
- **Expiration date monitoring** and renewal reminders with project impact assessment
- **Intelligent certificate recommendations** based on selected files and working folder requirements

---

## System Requirements

- **macOS 13.5** or later (Universal Binary support for Apple Silicon and Intel)
- **Xcode Command Line Tools** (for codesign, pkgutil, notarytool, xattr, hdiutil, and stapler integration)
- **Apple Developer Account** (for notarization and distribution certificates)
- **Keychain Access** to code signing certificates with proper permissions
- **Internet Connection** (for notarization operations and certificate validation)

---

## Security & Privacy

Signaro maintains the highest security standards with enterprise-grade protection:
- **Local operations only** — All signing, analysis, extended attributes, working folder, and DMG operations happen on your Mac with no external data transmission
- **Secure keychain integration** — Certificate access through macOS Keychain with proper permissions
- **No data collection** — Your files, working folders, and certificates never leave your device (except for Apple notarization)
- **Encrypted credential handling** — Notarization credentials stored in memory only with secure cleanup
- **Apple tool integration** — Uses official Apple command-line tools (codesign, pkgutil, notarytool, stapler, xattr, hdiutil)
- **Safe extended attributes handling** — Only removes problematic attributes with user consent and comprehensive logging
- **Universal cleaning security** — Extended attributes cleaning uses the same secure methods for all workflows
- **Comprehensive audit logging** — Detailed operation logs for security compliance and debugging
- **Privacy-first design** — No analytics, tracking, or data collection of any kind
- **Secure log storage** — Operation logs stored locally with automatic credential masking for privacy protection
- **Thread-safe logging** — Concurrent operations safely logged without data corruption or race conditions
- **Working folder security** — Project configurations stored locally with no external synchronization
- **DMG creation security** — All disk image operations performed locally using Apple's secure hdiutil framework

---

## Perfect For

- **iOS/macOS Developers** — Complete development workflow from signing to distribution-ready DMGs with working folders project organization, automatic extended attributes cleaning, and comprehensive audit logging
- **IT Administrators** — Manage configuration profiles and installer packages with enterprise features, working folder organization, professional DMG creation, and detailed operation logs for compliance
- **Enterprise Teams** — Ensure consistent signing and notarization across projects with team coordination tools, working folder sharing, unified extended attributes management, and comprehensive audit trails
- **Security Professionals** — Verify and analyze file signatures with detailed security assessment, metadata analysis, working folder project tracking, and complete operation logging for forensic analysis
- **Independent Developers** — Professional-grade signing, notarization, and DMG creation without complexity, with working folder organization, automatic handling of common file issues, and detailed logs for Apple Support communication
- **DevOps Engineers** — Integrate signing, notarization, and distribution operations into automated workflows and CI/CD pipelines with working folder batch processing, reliable extended attributes handling, and comprehensive logging for build process tracking

---

## Credits & Acknowledgments

- **Jesus Ayala**, Ayala Solutions (2025)
- **Inspiration**: Open MacAdmins community and feedback from real-world usage

---

## Version Information

- **Current Version**: 3.0 Build 1.0
- **Release**: 2025
- **Platform**: macOS (Universal Binary - Apple Silicon & Intel)
- **License**: Free
- **Support**: Community-driven with comprehensive documentation
- **Architecture**: Modern SwiftUI with enterprise-grade code documentation, advanced logging system, working folders, and professional DMG creation

## Alternative Tools

For command-line workflows, consider:
- [profilesigner](https://github.com/nmcspadden/ProfileSigner) — CLI alternative for configuration profiles
- Apple's native `codesign`, `pkgutil`, `xattr`, `hdiutil`, and `notarytool` commands

---

Choose Signaro for modern, intelligent, and secure code signing, notarization, and distribution on macOS!

---

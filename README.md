# Still testing - Let me know of issues.

# Signaro: Advanced macOS Code Signing & Notarization Utility

**Signaro** is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, **notarizing**, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, **comprehensive extended attributes cleaning**, **advanced persistent logging system**, **working folders project management**, **professional DMG creation**, **intelligent notarization requirements validation**, **modular MVVM architecture for enhanced performance**, and **comprehensive code documentation**, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

> **Current Status**: Feature-complete core functionality with ongoing enhancements. See [roadmap](#roadmap) for upcoming features.

---

## What's New in Version 2.5 Build 1.4

### Critical Bug Fixes & Stability Improvements
- **Fixed file removal functionality** — Resolved issues with red X buttons not properly showing confirmation dialogs and removing files from selection
- **Enhanced alert system reliability** — Improved alert handling for file removal confirmations and general notifications with better error recovery
- **Improved state management consistency** — File removal now properly cleans up all associated data including signatures, analysis results, and UI state
- **Better user feedback mechanisms** — Clear confirmation dialogs with accurate file count display and proper cancel/confirm options
- **Automatic UI updates after operations** — File analysis and status information automatically refreshes after file removal operations

### Workflow Optimization Updates
- **Enhanced file management workflow** — Streamlined file selection and removal process with better visual feedback
- **Improved batch operation reliability** — More consistent behavior when working with multiple files in working folders
- **Better error handling and recovery** — Enhanced error messages and recovery suggestions for common workflow issues
- **Optimized memory management** — Reduced memory usage during large file operations and extended analysis sessions
- **Performance improvements** — Faster UI updates and more responsive user interactions across all workflows

### User Experience Enhancements
- **Refined confirmation dialogs** — More intuitive and accessible confirmation flows for destructive operations
- **Better visual state indicators** — Clearer visual feedback for file states, operation progress, and completion status
- **Enhanced accessibility support** — Improved VoiceOver compatibility and keyboard navigation for all new features
- **Streamlined working folder operations** — More consistent behavior when adding/removing files from working folder projects
- **Professional error messaging** — Clear, actionable error messages with specific troubleshooting guidance

### Technical Infrastructure Improvements
- **Robust state synchronization** — Enhanced state management ensuring UI consistency across all operations
- **Memory leak prevention** — Improved memory management patterns preventing accumulation during extended usage
- **Thread safety enhancements** — Better concurrent operation handling for file management and analysis tasks
- **Data integrity validation** — Enhanced validation to prevent inconsistent states during file operations
- **Logging system improvements** — More detailed operation logging for better debugging and support

---

## What's New in Version 2.5 Build 1.3

### Major Architecture Refactoring for Enhanced Performance
- **Modular MVVM Architecture** — Complete refactoring of MainContentView (~1000+ lines) into focused, maintainable components
- **Dramatically Improved Build Times** — Modular architecture means only changed files need recompilation, reducing development iteration time
- **Enhanced Code Maintainability** — Clear separation of concerns makes code easier to understand, modify, and debug
- **Better Team Collaboration** — Multiple developers can work on different components without conflicts
- **Improved Testability** — ViewModels and Managers can be unit tested independently for better code quality
- **Future-Proof Architecture** — Scalable foundation for new features and enhancements

### New Architectural Components
- **MainContentViewModel** — Centralized state management with all UI state and computed properties
- **FileOperationsManager** — Dedicated manager for file signature analysis and smart analysis logic
- **SigningOperationsManager** — Specialized manager for code signing and unsigning operations
- **ValidationManager** — Focused manager for notarization requirements validation logic
- **Modular UI Components** — Reusable view components (ValidationModePickerView, FileListContainerView, MainContentToolbar, NotarizationValidationBannerView)

### Performance & Development Improvements
- **Faster Compilation** — Reduced MainContentView from ~1000+ lines to ~400 lines with distributed logic
- **Optimized Memory Usage** — Better state management and object lifecycle handling
- **Enhanced Error Handling** — Centralized error management with consistent user feedback
- **Improved Code Reusability** — Managers can be reused across different views and contexts
- **Better Debugging Experience** — Modular architecture makes issues easier to isolate and fix
- **Professional Code Organization** — Enterprise-grade architecture patterns for long-term maintainability

### Technical Enhancements
- **Async/Await Pattern Consistency** — Improved async operations handling across all managers
- **State Management Optimization** — Centralized @Published properties with efficient updates
- **Dependency Injection** — Proper dependency management for better testing and flexibility
- **Task-Based Concurrency** — Modern Swift concurrency patterns for file operations
- **Enhanced Code Documentation** — Comprehensive inline documentation for all new components

### Bug Fixes & Improvements 
- **Fixed file removal functionality** — Red X buttons now properly show confirmation dialogs and remove files from selection
- **Enhanced alert system** — Added comprehensive alert handling for file removal confirmations and general notifications
- **Improved state management** — File removal now properly cleans up all associated data including signatures, analysis results, and UI state
- **Better user feedback** — Clear confirmation dialogs with file count display and proper cancel/confirm options
- **Automatic UI updates** — File analysis and status information automatically refreshes after file removal operations

### Working Folders System
- **Project-based organization** — Group related files together into working folders for batch operations and project management
- **Persistent folder management** with automatic status tracking across all files in each folder
- **Smart file organization** with auto-suggested folder names based on file locations and naming patterns
- **Batch operations** perform signing, notarization, analysis, and DMG creation on entire working folders
- **Working folder status aggregation** showing overall project readiness and completion status
- **Export configuration** functionality for sharing working folder setups with team members
- **Comprehensive logging integration** with all working folder operations automatically logged for audit trails
- **Toggle working folder mode** to switch between individual file management and project-based workflows
- **Working folder creation wizard** with intelligent file grouping suggestions and naming recommendations

### Professional DMG Creation System
- **Complete DMG creation workflow** with support for multiple formats (compressed, read/write, read-only, highly compressed)
- **Advanced DMG options** including encryption (AES-128/256), segmentation for large files, and custom background images
- **App distribution DMGs** with professional drag-and-drop layouts including Applications folder aliases
- **Multi-file DMG creation** from selected file collections with automatic staging and organization
- **Blank DMG creation** for working disk images with user-specified sizes
- **Professional layout customization** with custom icon positioning, window sizing, and background images
- **Volume naming intelligence** with automatic name generation based on source content
- **Size calculation and optimization** with automatic size estimation, compression recommendations, and space efficiency analysis
- **Complete operation logging** with detailed DMG creation logs including command-line calls and processing times
- **Encryption support** with password protection using industry-standard AES encryption

### App Distribution Workflow
- **End-to-end distribution preparation** from source app to ready-to-distribute DMG with signing and notarization
- **Automated workflow orchestration** combining signing, notarization, stapling, and professional DMG creation in a single process
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
- **One-click distribution preparation** from unsigned app to distribution-ready DMG with complete audit logging and professional presentation

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

### App Distribution Workflow
*[Screenshot: Complete distribution workflow from app to DMG]*
- **End-to-end workflow orchestration** showing all steps from signing to DMG creation
- **Distribution readiness validation** with comprehensive requirement checking
- **Workflow progress tracking** with detailed status updates at each stage
- **Professional DMG preview** showing final distribution layout and presentation
- **One-click distribution preparation** with complete automation and logging

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
- **Local operations only** — All signing, analysis, extended attributes, working folder, validation, and DMG operations happen on your Mac with no external data transmission
- **Secure keychain integration** — Certificate access through macOS Keychain with proper permissions
- **No data collection** — Your files, working folders, validation results, and certificates never leave your device (except for Apple notarization)
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
- **Validation privacy** — All requirements validation performed locally with no external service communication
- **Modular architecture security** — Enhanced security through separation of concerns and proper access control

---

## Perfect For

- **iOS/macOS Developers** — Complete development workflow from signing to distribution-ready DMGs with working folders project organization, intelligent notarization validation, automatic extended attributes cleaning, modular architecture for customization, and comprehensive audit logging
- **IT Administrators** — Manage configuration profiles and installer packages with enterprise features, working folder organization, pre-flight validation, professional DMG creation, enhanced performance, and detailed operation logs for compliance
- **Enterprise Teams** — Ensure consistent signing and notarization across projects with team coordination tools, working folder sharing, validation standardization, unified extended attributes management, improved build times, and comprehensive audit trails
- **Security Professionals** — Verify and analyze file signatures with detailed security assessment, validation requirements checking, metadata analysis, working folder project tracking, enhanced debugging capabilities, and complete operation logging for forensic analysis
- **Independent Developers** — Professional-grade signing, notarization, and DMG creation without complexity, with working folder organization, intelligent validation, automatic handling of common file issues, faster development iteration, and detailed logs for Apple Support communication
- **DevOps Engineers** — Integrate signing, notarization, and distribution operations into automated workflows and CI/CD pipelines with working folder batch processing, validation automation, reliable extended attributes handling, modular architecture for integration, and comprehensive logging for build process tracking

---

## Version Information

- **Current Version**: 2.5 Build 1.4
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

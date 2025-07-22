# (testing to confirm fixed) -Let me know

# Signaro: Advanced macOS Code Signing & Notarization Utility

**Signaro** is a completely rewritten, modern, privacy-focused macOS app for signing, unsigning, **notarizing**, and comprehensively analyzing `.pkg`, `.app`, and `.mobileconfig` files. With an intuitive drag & drop interface, advanced signature analysis, smart certificate recommendations, seamless Apple notarization integration, and **comprehensive code documentation**, Signaro is the ultimate tool for developers, IT administrators, and security professionals.

> **Current Status**: Feature-complete core functionality with ongoing enhancements. See [roadmap](#roadmap) for upcoming features.


---

## What's in Version 2.0 Build 1.7

Added the option to **Remove Extended Attributes
- Fix DMG support

### **🧠 Enhanced Smart Analysis System**
- **Manual Smart Analysis Button** with immediate comprehensive results (bypasses 3-second auto-analysis delay)
- **Force Refresh Capabilities** that clear all cached data for completely fresh analysis
- **Network-dependent validation** with certificate revocation checking and OCSP verification
- **Certificate compatibility re-analysis** when certificate selection changes
- **Deep debugging support** with enhanced error reporting and diagnostics
- **Educational exploration mode** for learning code signing concepts

### **🔍 Comprehensive Notarization Preflight Validation**
- **Pre-submission certificate validation** ensuring files have correct Developer ID certificates
- **Code signing status verification** for .app bundles before notarization submission
- **Certificate type compatibility checking** (Developer ID vs Mac App Store certificates)
- **Hardened Runtime validation** with warnings for missing security features
- **File format validation** ensuring only notarizable file types (.app, .pkg, .dmg) are submitted
- **Comprehensive validation feedback** with specific recommendations for fixes

### **📋 Comprehensive Code Documentation**
- **Inline documentation** with architectural diagrams and implementation guides: SignaroApp, MainContentView, DataModels, IdentityManager, SignatureChecker, FileAnalysisEngine, CodeSigningOperations, NotarizationOperations
- **Professional commenting standards** throughout codebase: SignaroApp, MainContentView, DataModels, IdentityManager, SignatureChecker, FileAnalysisEngine, CodeSigningOperations, NotarizationOperations
- **Technical implementation guides** with Apple framework integration details: Apple framework integration, codesign, pkgutil, notarytool, stapler
- **Performance optimization** strategies and troubleshooting workflows: Optimization strategies, troubleshooting workflows
- **Accessibility guidelines** and user experience design patterns: Accessibility guidelines, user experience design patterns

### **🔍 Advanced File Analysis & Intelligence**
- **Comprehensive file analysis** with intelligent recommendations and security insights
- **Certificate type analysis** with specific suggestions for each file type and workflow
- **Team ID consistency checking** across multiple files with conflict detection
- **Notarization status verification** for all applicable files with detailed reporting
- **Apple best practices guidance** based on your specific use case and file types
- **Signature issue diagnosis** with actionable resolution strategies

### **💡 Intelligent Certificate Recommendations**
- **Context-aware suggestions** based on selected file types and distribution requirements
- **Auto-detection of appropriate certificates** from your keychain with trust validation
- **Real-time validation** of certificate suitability for specific files and workflows
- **Visual indicators** for optimal certificate matches and compatibility warnings
- **Certificate acquisition guidance** for missing certificate types

### **📁 Enhanced File Management & UI**
- **Advanced file sorting** by name, extension, or signature status with persistent preferences
- **Expandable file details** with comprehensive signature information and metadata
- **File type badges** for easy identification (.app, .pkg, .mobileconfig) with system icons
- **Drag & drop support** with duplicate detection and batch processing capabilities
- **Responsive layout design** with accessibility support and keyboard navigation

### **🔐 Deep App Bundle Analysis**
- **Enhanced entitlements viewer** with search functionality and export capabilities
- **Bundle structure verification** and validation with detailed reporting
- **Signature diagnosis** with specific issue identification and resolution guidance
- **Automated fix suggestions** for common signing problems and certificate issues
- **Comprehensive security analysis** with vulnerability detection

### **📊 Professional Analysis Export & Reporting**
- **Copy individual sections** (valid signatures, recommendations, analysis results)
- **Export complete analysis** with timestamps, formatting, and detailed metadata
- **Clipboard integration** for documentation, reporting, and team collaboration
- **Professional formatting** for enterprise documentation and compliance reporting
- **Audit trail generation** for security compliance and debugging

### **🎓 Comprehensive Help System**
- **Interactive Help Sheet** with 12 detailed sections covering all application features
- **9-Section Notarization Guide** with step-by-step instructions for Apple notarization
- **Certificate Information Dialog** with detailed explanations of certificate types
- **Contextual Help Integration** with tooltips and guidance throughout the interface
- **Progressive Disclosure** managing information complexity without overwhelming users
- **Professional Documentation** suitable for enterprise environments and technical users

### **📋 Best Practices**
- Apple-specific guidance for each file type with detailed explanations
- Security recommendations based on your workflow and requirements
- Industry standard compliance suggestions with actionable steps
- Performance optimization recommendations for large-scale operations

---

## Core Features

### **✍️ Advanced Code Signing**
- **Batch sign & unsign** multiple files simultaneously with progress tracking
- **Smart certificate selection** with trust status verification and compatibility checking
- **In-place signing** for .app bundles (preserves original location and metadata)
- **Copy-based signing** for .pkg and .mobileconfig files (maintains originals)
- **Real-time signature verification** with detailed status reporting and error diagnosis
- **Comprehensive error handling** with recovery suggestions and troubleshooting guidance

### **🍎 Apple Notarization Integration**
- **Seamless notarization workflow** with Apple's notary service and automated submission
- **Secure credential management** with validation, encrypted storage, and session handling
- **Advanced status checking** for any notarization request ID with detailed progress monitoring
- **Automatic stapling** after successful notarization with integrity verification
- **Comprehensive notarization reporting** with detailed logs, timestamps, and audit trails
- **Batch notarization support** with queue management and progress tracking

### **🔑 Certificate Management**
- **Automatic keychain discovery** of code signing certificates with trust validation
- **Trust status indicators** (trusted/untrusted certificates) with detailed information
- **Certificate type identification** (Developer ID, Apple Development, etc.) with usage guidance
- **Expiration date monitoring** with renewal reminders and proactive notifications
- **Certificate information dialogs** with comprehensive details and troubleshooting tips
- **Keychain integration** with secure access and permission handling

### **🔍 File Analysis & Verification**
- **Multi-format support**: `.pkg`, `.app`, `.mobileconfig` with extensible architecture
- **Signature validation** with complete authority chain verification and trust assessment
- **Team ID extraction** and consistency checking across file collections
- **Notarization status verification** for all applicable file types with detailed reporting
- **File icon display** using system icons with type identification badges
- **Security posture assessment** with vulnerability detection and compliance checking

### **🎨 Professional UI & UX**
- **Modern SwiftUI interface** with native macOS design patterns and animations
- **Comprehensive accessibility support** with VoiceOver compatibility and keyboard navigation
- **Intelligent keyboard shortcuts** for efficient workflow and power user features
- **Responsive layout design** with resizable windows and adaptive content
- **Dark mode support** following system preferences with high contrast compatibility
- **Professional visual design** with consistent spacing, typography, and color schemes

---

## Screenshots & Interface Overview

### **🏠 Main Interface**
*[Screenshot: Main application window showing complete workflow]*
- Certificate selection dropdown with intelligent recommendations
- File list with drag & drop support and sorting options
- Operation buttons (Sign, Unsign, Analyze, Notarize) with status indicators
- Real-time file signature analysis and validation results

### **🔐 Certificate Management**
*[Screenshot: Certificate selection and management interface]*
- Available certificates from keychain with trust status indicators
- Certificate type identification (Developer ID, Apple Development, etc.)
- Expiration date monitoring and renewal reminders
- Intelligent certificate recommendations based on selected files

### **📁 File Analysis & Details**
*[Screenshot: Expanded file details showing comprehensive analysis]*
- Individual file signature status with detailed information
- Expandable file details with entitlements viewer
- Signature diagnosis with specific issue identification
- Terminal command generation for advanced users

### **🧠 Smart Analysis Results**
*[Screenshot: File Analysis banner showing intelligent recommendations]*
- Comprehensive security analysis and recommendations
- Certificate compatibility assessment and suggestions
- Apple best practices guidance for distribution readiness
- Team ID consistency checking across multiple files

### **🔍 Enhanced Entitlements Viewer**
*[Screenshot: Detailed entitlements viewer with search functionality]*
- Complete entitlements display with syntax highlighting
- Search functionality for finding specific entitlements
- Export capabilities for documentation and sharing
- Copy individual entitlements or complete sections

### **🍎 Notarization Workflow**
*[Screenshot: Notarization credential dialog and progress monitoring]*
- Secure credential input with validation
- Real-time notarization progress tracking
- Detailed status updates and Apple API responses
- Automatic stapling workflow management
  


### **⚙️ Advanced Features**

- File sorting and filtering options
- Batch operation progress tracking
- Comprehensive error reporting and diagnostics
- Professional analysis export and clipboard integration

---

## System Requirements

- **macOS 13.5** or later (Universal Binary support for Apple Silicon and Intel)
- **Xcode Command Line Tools** (for codesign, pkgutil, notarytool integration)
- **Apple Developer Account** (for notarization and distribution certificates)
- **Keychain Access** to code signing certificates with proper permissions
- **Internet Connection** (for notarization operations and certificate validation)

---

## Security & Privacy

Signaro maintains the highest security standards with enterprise-grade protection:
- **Local operations only** — All signing and verification happens on your Mac with no external data transmission
- **Secure keychain integration** — Certificate access through macOS Keychain with proper permissions
- **No data collection** — Your files and certificates never leave your device (except for Apple notarization)
- **Encrypted credential handling** — Notarization credentials stored in memory only with secure cleanup
- **Apple tool integration** — Uses official Apple command-line tools (codesign, pkgutil, notarytool, stapler)
- **Comprehensive audit logging** — Detailed operation logs for security compliance and debugging
- **Privacy-first design** — No analytics, tracking, or data collection of any kind

---

## Perfect For

- **iOS/macOS Developers** — Sign and notarize apps for distribution with comprehensive workflow support
- **IT Administrators** — Manage configuration profiles and installer packages with enterprise features
- **Enterprise Teams** — Ensure consistent signing across projects with team coordination tools
- **Security Professionals** — Verify and analyze file signatures with detailed security assessment
- **Independent Developers** — Professional-grade signing without complexity or learning curve
- **DevOps Engineers** — Integrate signing operations into automated workflows and CI/CD pipelines

---

## Credits & Acknowledgments

- **Author*: Jesus M. Ayala, Ayala Solutions (2025)
- **Inspiration**: Open MacAdmins community and feedback from real-world usage

---

## Version Information

- **Current Version**: 2.0 Build 1.5
- **Release**: 2025
- **Platform**: macOS (Universal Binary - Apple Silicon & Intel)
- **Architecture**: Modern SwiftUI with enterprise-grade code documentation

---

## Alternative Tools

For command-line workflows, consider:
- [profilesigner](https://github.com/nmcspadden/ProfileSigner) — CLI alternative for configuration profiles
- Apple's native `codesign`, `pkgutil`, and `notarytool` commands

---

Choose Signaro for modern, intelligent, and secure code signing on macOS!

### **🧠 Smart Analysis Workflow**
1. **Automatic Analysis** — Files are automatically analyzed 3 seconds after selection for convenience
2. **Manual Smart Analysis** — Click "Smart Analysis" for immediate, comprehensive results without delay
3. **Force Refresh** — Use Smart Analysis to clear cached data and perform fresh analysis
4. **Enhanced Validation** — Manual analysis includes network-dependent checks for maximum accuracy

---

lication bootstrap<br>• Runtime initialization<br>• System integration setup<br>• Process lifecycle management | • AppKit framework<br>• Application lifecycle<br>• System bootstrap<br>• Runtime environment |

### **🔗 Integration Dependencies**

| Category | External Dependencies | Internal Coordination |
|----------|----------------------|----------------------|
| **Apple Frameworks** | Security.framework, AppKit, SwiftUI, UniformTypeIdentifiers | Certificate management, UI rendering, file type handling |
| **Command Line Tools** | codesign, pkgutil, notarytool, stapler, spctl | File operations, signature verification, Apple services |
| **System Services** | Keychain Services, File System, Network APIs | Certificate storage, file access, Apple API communication |

---

## Roadmap

### **🚀 Planned Features (Next Releases)**
- **Actionable Smart Recommendations System** with one-click problem resolution
- **Settings & Preferences Panel** with workflow customization
- **Certificate Expiration Monitoring** with proactive alerts
- **Enhanced Batch Operations** with queue management
- **Operation History & Analytics** with detailed reporting
- **Command Line Interface (SignaroCLI)** for automation
- **Enhanced Error Recovery** with automatic retry mechanisms

### **💡 Future Enhancements**
- **Team Collaboration Features** for enterprise workflows
- **Advanced Analytics Dashboard** with usage insights
- **Compliance Reporting** for enterprise security requirements
- **Integration APIs** for CI/CD pipeline automation

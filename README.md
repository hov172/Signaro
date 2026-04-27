# Signaro: Advanced macOS Code Signing & Notarization Utility

Signaro is a professional-grade, privacy-first macOS application for code signing, notarization, stapling, and distribution of `.app`, `.pkg`, `.dmg`, and `.mobileconfig` files. Built with SwiftUI and a strict MVVM architecture, it shares a single operations layer between the GUI and a native companion CLI, so every guarantee that holds in the app holds in automation as well. All processing is local; no credentials, file contents, or metadata leave the device except as required by Apple's notarization service.

**Current version: 5.0 Build 1.3 (2026-04-27)**

## Table of Contents

- [What's New](#whats-new-in-version-50-build-13)
- [Core Features](#core-features)
  - [Code Signing](#code-signing)
  - [Notarization](#notarization)
  - [DMG Creation and Customization](#dmg-creation-and-customization)
  - [Distribution Workflows](#distribution-workflows)
  - [Certificate Management](#certificate-management)
- [Command-Line Interface](#command-line-interface)
  - [CLI Commands](#commands)
  - [End-to-End Example (Profile-Based)](#end-to-end-example-profile-based)
- [Notarization Credential Modes](#notarization-credential-modes)
- [System Requirements](#system-requirements)
- [Troubleshooting](#troubleshooting)
- [Architecture Overview](#architecture-overview)
- [Version Information](#version-information)

---

## What's New in Version 5.0 Build 1.3

DMG distribution workflow reliability and diagnostics release.

- **Fixed: DMG layout persistence mismatch against preview/background size.** Finder customization now explicitly hides the path bar during layout scripting so window bounds and visible background rendering stay consistent with the preview.
- **Fixed: Workflow checkpoint discard consistency.** "Discard All" now clears pending workflow checkpoints through an awaited single-path call, preventing stale checkpoint banners after reopen/relaunch.
- **Added: DMG layout debug instrumentation.** App and PKG distribution flows now emit step-by-step DMG layout debug logs, including AppleScript generation/execution markers and script capture in operation output for troubleshooting.
- **Build metadata bump.** `CURRENT_PROJECT_VERSION` is now `1.3` (`MARKETING_VERSION` remains `5.0`), and the CLI version string is now `SignaroCLI 5.0.1.3`.

---

## What's New in Version 5.0 Build 1.1

Maintenance release addressing ten confirmed defects from a senior-developer audit after the 5.0 Build 1.0 launch.

- **Fixed: App Distribution workflow failed when Skip Notarize & Staple was enabled.** The `.waitForAppNotarization` step was calling `waitForNotarization(requestId: nil, …)` and receiving a hard failure instead of a skip-success result. The guard now recognizes `nil` as a skip signal and short-circuits correctly.
- **Fixed: Batch resume silently skipped the failed file.** Both `BatchSigningCoordinator` and `BatchDistributionCoordinator` saved a checkpoint — storing `completedCount = index + 1` — before evaluating whether the file had failed. On resume, the stored count caused the coordinator to start at the next file, leaving the failed file permanently skipped. The checkpoint is now written only after a successful result.
- **Fixed: Distribute All PKG with "Create DMG" enabled silently skipped DMG signing.** The `PkgDistributionConfiguration` built inside the batch distribution path was missing `dmgSigningIdentity`, causing `signDMG()` to return `⏭ Skipped` unconditionally even when the user had enabled signing. The Developer ID Application identity is now passed explicitly.
- **Fixed: Working Folder per-file DMG volume icon picker never fired.** Two stacked `.fileImporter` modifiers on `DMGFileSettingsView` — a known SwiftUI limitation where only the last-registered importer fires — caused the icon picker to appear to open but immediately dismiss with no file selected. Both pickers are replaced with direct `NSOpenPanel` calls.
- **Fixed: Certificate auto-select blocked distribution when the only certificate was expiring imminently.** `bestIdentity(for:)` was treating `.expiringImminently` identically to `.expired`, excluding still-valid certificates from the candidate pool and presenting a false "no certificate available" state. Only `.expired` certificates are now excluded; expiring-imminently certificates remain available with the existing amber-banner advisory.
- **Fixed: PKG Distribution advanced DMG layout was silently ignored.** The `buildAndCustomizeDMG()` internal `appName` derivation returned `nil` for non-`.app` sources, preventing the `buildCustomizationScript()` call from executing. PKG, DMG, and `.mobileconfig` sources now pass their full `lastPathComponent` so layout customization runs for all source types.
- **Fixed: PKG DMG AppleScript item reference appended `.app` unconditionally.** After fixing the above, the Finder `set position of item` script was generating references like `"MyInstaller.pkg.app"` — a nonexistent item. The item name now includes `.app` only when the source name carries no extension (i.e., bare `.app` bundle names passed by the App Distribution path). Files with explicit extensions — `.pkg`, `.dmg`, `.mobileconfig` — are referenced verbatim.
- **Fixed: PKG Distribution notarization polling loop swallowed Task cancellation.** Three `try? await Task.sleep` calls in the notarization polling loop discarded `CancellationError`, causing the workflow to continue polling for up to 15 minutes after the user pressed Cancel. All three sites now use `do { try await } catch { return cancellation result }`.
- **Fixed: Auto-clear after Distribute All fired unconditionally on total failure.** The file list cleared even when every file in the batch had failed, removing failure evidence before the user could review it. The clear is now guarded by `success > 0`.
- **Fixed: `BatchSigningCoordinator.cancel()` left state as `.running`.** Calling `cancel()` tore down the underlying `Task` but left `state = .running`. Observers continued to see a running coordinator indefinitely. `cancel()` now sets `state = .idle`.

---

## What's New in Version 5.0 Build 1.0

Batch distribution engine, full Create DMG layout preview parity, Working Folders toolbar integration, and automatic file-list clearing on workflow completion.

- **Distribute All with per-file DMG customization.** The Distribute action now fans out through `BatchDistributionCoordinator`, a sequential engine that processes every file in the selection in order. Each file carries its own `DMGFileSettings` record — independent volume name, background image, window size, icon positions, and icon size — so a mixed-app batch produces correctly customized DMGs for each file without re-entering the workflow dialog. A live progress row beneath the file list displays the current file name, index, and total count. The engine inherits the same checkpoint-resume infrastructure as the batch signing engine: a checkpoint is saved after each successfully distributed file, and the `PendingWorkflowCheckpointsBanner` at next launch offers a Resume button that restarts from the exact failure point.

- **Create DMG dialog: full inline layout preview parity.** The standalone Create DMG dialog now provides the same live layout preview as the App Distribution and PKG Distribution workflow dialogs. All customization is exposed inline in a `"Layout, Appearance & Security"` disclosure group — no separate sheet. Controls include: volume icon picker (`.icns`), background image picker (PNG/JPEG/TIFF), custom window size (width 400–1200 px, height 300–800 px, step 20), icon size (16–128 px, step 8), text size (10–16 pt, step 1), and per-file icon position (x/y steppers with out-of-bounds detection). The `DMGLayoutPreviewView` renders inline when custom layout is enabled, with grid overlay, rulers, snap-to-guides, drag-to-position editing, a hide/show toggle, and a "Preferences" action that resets saved preview defaults for all DMG surfaces. Dragging a tile beyond the current window bounds auto-expands the editable layout dimensions. Encryption (AES-128 and AES-256 with password confirmation and minimum-length enforcement) and segmentation controls are also inline. `DMGAdvancedOptions` properties were changed from `let` to `var` to support direct SwiftUI `@State` binding mutation without full struct reconstruction. All three DMG surfaces — App Distribution, PKG Distribution, and Create DMG — now have identical inline preview capability.

- **Working Folders toolbar button.** The Working Folders sidebar toggle is now exposed as a direct toolbar button (SF Symbol `sidebar.left`, accent-colored when active) rather than an item buried in the `•••` overflow menu.

- **Auto-clear file list on workflow completion.** After a Sign, Unsign, or Distribute workflow completes with at least one success, the file list clears automatically after a 2-second delay, returning the UI to a clean ready state. The delay is surfaced as a "Clearing selection…" suffix on the completion status message. Working folders are intentionally not cleared.

---

## What's New in Version 4.8 Build 1.0

Batch signing engine with per-file progress and checkpoint resume, Working Folder Sign All with automatic certificate routing, and a pre-flight UX improvement.

- **Batch signing engine with per-file progress and checkpoint resume.** The Sign action now fans out through `BatchSigningCoordinator`, which processes files sequentially and publishes live progress (file name, index/total) in a dedicated row beneath the file controls. A checkpoint is saved to disk after each successfully signed file. If signing fails mid-batch, the coordinator pauses and preserves the checkpoint so the run can resume from the exact point of failure at the next launch — previously signed files are not re-signed. Cancellation is clean: the underlying `Task` is cancelled, `currentProgress` is cleared, and the checkpoint is removed only on successful completion so interrupted runs are always resumable. `BatchSigningCoordinator` and `BatchDistributionCoordinator` share a common checkpoint store (`WorkflowCheckpointStore`) and a `PendingWorkflowCheckpointsBanner` that appears at launch when a saved checkpoint exists.

- **Working Folder Sign All.** The "Sign All" button and "Sign All Files" context-menu item in the Working Folder Manager now have a full implementation. Each unsigned file in the folder is signed in sequence, automatically routed to the correct certificate class: `.app` and `.dmg` use Developer ID Application; `.pkg` and `.mobileconfig` use Developer ID Installer. Files with no compatible certificate in the active identity list are skipped without error. The button displays a spinner while signing runs and briefly shows "Done" on completion. It is disabled when all files in the folder are already validly signed. Signature status pills update in place after the operation completes.

- **Pre-flight alert: "Continue Anyway" → "Open Working Folder Manager."** When the Distribute action detects that the selection contains extra non-target files (for example, a `.dmg` alongside the target `.app`), the secondary alert button previously offered "Continue Anyway," implying all files would be processed. That button is now "Open Working Folder Manager," which opens the folder manager where users can organize files into named working folders and use Sign All for multi-file batch workflows.

- **Eliminated "Publishing changes from within view updates" runtime warnings.** The `.onChange` handlers observing `batchCoordinator.state` and `batchCoordinator.currentProgress` were mutating `@Published` ViewModel properties synchronously during SwiftUI's layout pass, triggering repeated runtime warnings and undefined behavior in edge cases. Both handlers now wrap mutations in `Task { @MainActor in }` to defer to the next run-loop turn.

---

## What's New in Version 4.7

Certificate selection is now workflow-aware and the DMG layout preview is fully accurate.

- **Workflow-aware certificate auto-select.** The App Distribution and PKG Distribution dialogs open with the best matching Developer ID certificate already selected. App distribution remembers the last-used Developer ID Application identity separately from PKG distribution's Developer ID Installer identity, so switching between workflow types does not clobber the previously selected certificate.
- **Expired-certificate fallback is visible.** If the workflow-specific last-used certificate has expired, Signaro falls back to the best valid replacement, shows an inline amber banner explaining the fallback, and posts a macOS User Notification once per session so the fallback is noticed even if the dialog is dismissed quickly.
- **Split-aware signing.** The main Sign action routes each file to the certificate class that matches its type: `.app` and `.dmg` use Developer ID Application; `.pkg` and `.mobileconfig` use Developer ID Installer. Distribution workflows require a homogeneous selection — mixed app/install files must be run in separate distribution passes.
- **Accurate DMG layout preview with auto-sizing and bounded auto-expand.** The live preview renders the background image at native 1:1 pixel resolution, matching exactly what Finder shows when the DMG is opened. Picking a background image automatically sets the window dimensions to the image's native size. Auto-expand is capped at the background image's native dimensions (or 3840×2160 when no background is set) so dragging an icon cannot produce runaway window geometry. Reset Layout restores to the background image's native dimensions when one is set and to the standard defaults otherwise.
- **Build 1.5:** Fixed ruler and grid visibility on dark DMG backgrounds. Build 1.6 corrected two compounding icon-position bugs — an incorrect Y-axis flip and double-scaling in the preview ZStack — and replaced `NSImage` (AppKit-only) with `CGImageSourceCopyPropertiesAtIndex` (ImageIO) in the CLI target for background image size detection.

---

## What's New in Version 4.6

Submission History Browser, Entitlement & Profile Inspector, two new CLI commands, and stability patches.

- **Submission History Browser.** Every notarization, signing, stapling, and distribution operation is stored as a structured `SubmissionLogEntry` and browsable from a dedicated view opened via the History toolbar button in the Submission Log window. Filter by operation type, search by filename or UUID, toggle a failures-only view, and copy the request ID to the clipboard with one click. The detail panel shows the full command line, Apple response, processing time, pre/post signature state, and output path.
- **Entitlement & Profile Inspector.** Side-by-side comparison of the entitlements embedded in a signed `.app` (via `codesign --display --entitlements - --xml`) against a `.mobileprovision` profile (via `security cms -D -i`). Keys present in one side but absent from the other are highlighted in orange with a mismatch count banner. Single-side inspection is supported. Risky entitlements — `get-task-allow`, `disable-library-validation`, `allow-unsigned-executable-memory`, `disable-executable-page-protection`, `allow-dyld-environment-variables` — are flagged with advisory text. Access via More (···) → Entitlement Inspector…
- **Entitlements in Smart Analysis.** When Smart Analysis runs on `.app` bundles, entitlements are extracted asynchronously and surfaced inside each file's analysis card. An Open Inspector… button opens the full inspector pre-loaded with that app.
- **`staple --uuid <id> <path>` (v4.6+).** Poll a known notarization submission UUID and staple the target file once Apple marks it Accepted. Enables deferred-staple pipelines where submission and stapling are separate CI steps.
- **`xcode-phase <path.xcodeproj>` (v4.6+).** Read an Xcode project's active build settings (`xcodebuild -showBuildSettings`) and emit a ready-to-paste Run Script Build Phase that calls `signaro distribute app` with the correct `PRODUCT_NAME`, `DEVELOPMENT_TEAM`, `CODE_SIGN_IDENTITY`, and `PRODUCT_BUNDLE_IDENTIFIER` values.
- **Build 1.2 patch:** Fixed notarization polling false rejection — the polling loop was evaluating `contains("invalid")` before `contains("in progress")`, matching the word "Invalid" in Apple's in-progress help text and exiting with a failure result while the submission was still active. Status checks are now evaluated in the correct precedence order.
- **Build 1.3 patch:** PKG Distribution Full DMG layout parity (see 4.6 Build 1.3 entry in Release Notes). Fixed DMG window size lost in segmented/encrypted pipelines. Fixed Help menu routing.

---

## What's New in Version 4.5

Full CLI feature parity and distribution pipeline tooling.

- **`SignaroCLI` reaches feature parity with the GUI.** `distribute app`, `distribute pkg`, `dmg create`, `sign`, `notarize`, `staple`, `validate`, `analyze`, `identities list` all share the same underlying operation modules as the GUI application. No duplication; no shell-script wrappers.
- **Structured identity output.** `identities list --json` returns a full object array including name, SHA-1, Team ID, serial number, expiry date, and trust indicators.
- **Quick validation mode.** `validate --mode quick` performs a high-speed preflight check for CI gatekeeping.
- **Build 1.1 parser hardening.** Boolean flags no longer consume positional arguments. `--key=value` inline assignment is parsed correctly. The `--` end-of-options terminator is honored. UUID pre-validation in `notarize wait` and `notarize log`.
- **Build 1.2 tooling.** `Signaro (Embedded CLI)` scheme, `Release-Embedded` configuration, `scripts/release.sh` and `scripts/create-installer.sh`.

## What's New in Version 4.1 Build 1.1

DMG customization that works. Rebuilds the customization pipeline to apply icons, backgrounds, and window layout on the writable mount post-creation.

- **Custom volume icons now actually appear on mounted DMGs.** The previous implementation wrote the `kHasCustomIcon` flag on the DMG source folder, which `hdiutil` silently discarded — the icon file was there but the flag wasn't. Fixed by applying icons, backgrounds, and window layout on the writable mount post-creation.
- **Advanced DMG options in the notarization workflow.** The Sign → Notarize → Staple → Distribute dialog gains a collapsed "Advanced DMG Options" section with controls for custom volume icon (`.icns`), background image (PNG/TIFF/JPG), window width/height, icon size (16–128 pt), text size (10–16 pt), and icon positions for the app bundle and Applications alias.
- **Clearer primary action.** Button text is now "Sign, Notarize, Staple & Distribute" so what the workflow does is visible at a glance.
- **Explicit success confirmation.** After a DMG is created you'll see a dialog with the filename, size, and save location.
- **Fixed: Finder layout actually persists.** Background, window size, and icon positions now reliably stick.
- **Fixed: Volume icon pickers open reliably.** Replaced stacked file importers with direct `NSOpenPanel` calls to avoid silent shadowing.
- **Fixed: AppleScript errors surface in logs.** Errors in the Finder layout script are now captured and surfaced instead of disappearing.
- **Note on first use:** macOS will prompt *"Signaro would like to control Finder"* the first time you create a DMG with a custom layout. This permission is required for layout customization and can be managed in System Settings → Privacy & Security → Automation.

---

## Core Features

### Code Signing

- **In-place and copy-based signing** using `codesign` with hardened-runtime entitlements (`--options=runtime`) for notarization compatibility. Supports Developer ID Application and Developer ID Installer certificate classes.
- **Split-aware signing for mixed selections.** When the file list contains both app-type (`.app`, `.dmg`) and installer-type (`.pkg`, `.mobileconfig`) files, each file is signed with the certificate class that matches its type.

| File Type | Extension | Certificate Class |
|:---|:---|:---|
| **App Bundles** | `.app` | Developer ID Application |
| **Disk Images** | `.dmg` | Developer ID Application |
| **Installers** | `.pkg` | Developer ID Installer |
| **Config Profiles** | `.mobileconfig` | Developer ID Installer |

- **Extended attributes cleaning** (`xattr -cr`) before signing, ensuring no quarantine flags or third-party metadata interferes with notarization assessment.
- **Batch signing engine with checkpoint resume (v4.8+).** `BatchSigningCoordinator` processes files sequentially, publishes per-file live progress, saves a checkpoint after each success, and pauses on failure so the run is resumable from the exact failure point at next launch.

### Notarization

- **Full Apple notarization pipeline** via `notarytool submit` + polling loop + `stapler staple`. Supports all three Apple credential modes: Apple ID + app-specific password, Keychain Profile (`notarytool store-credentials`), and App Store Connect API Key (`.p8`).
- **Notarization requirements validation.** Pre-submission static analysis checks the signed binary's hardened runtime flag, entitlement safety, code signature validity, minimum OS version, bundle structure, and `Info.plist` completeness — surfacing issues before they cause Apple to reject the submission.
- **Entitlement & Profile Inspector (v4.6+).** Side-by-side diff between the entitlements embedded in a signed `.app` and any `.mobileprovision` profile, with orange highlighting on mismatches and risky-entitlement advisory text.

### DMG Creation and Customization

Professional disk image creation with full Finder layout customization via a mount-customize-convert pipeline (`hdiutil create -type UDIF` → R/W mount → Finder AppleScript layout → `hdiutil convert`).

- **Volume icon** (`.icns`): written to `<mount>/.VolumeIcon.icns`; `kHasCustomIcon` set via the `com.apple.FinderInfo` xattr.
- **Background image**: staged to `<mount>/.background/` and referenced via Finder AppleScript `set background picture of opts`.
- **Window geometry**: bounds, icon size (16–128 px), text size (10–16 pt), and per-file icon positions via AppleScript `set position of item`.
- **Encryption**: AES-128 and AES-256, with password piped via stdin to avoid shell-history exposure.
- **Segmentation**: `hdiutil convert -segmentSize` for split DMG sets.
- **Inline live preview**: Drag-to-position editing of icon placements with grid overlay, rulers, snap-to-guides, and auto-expand when icons are dragged beyond the current window bounds. Persistent per-surface presentation preferences via `@AppStorage`. A Preferences action resets presentation defaults for all three DMG surfaces simultaneously.
- **All three DMG surfaces at full parity (v5.0+).** App Distribution workflow dialog, PKG Distribution workflow dialog, and the standalone Create DMG dialog all expose the same inline preview and layout controls. No separate sheet.

### Distribution Workflows

- **App Distribution Workflow**: sign → notarize → staple → create DMG → sign DMG → notarize DMG → staple DMG. Full step-by-step progress with per-step result detail. The workflow supports `skipNotarizeAndStaple` for offline or pre-notarized scenarios, and `cleanExtendedAttributes` for files that carry quarantine or third-party xattrs.
- **PKG Distribution Workflow**: sign `.pkg` with `productsign` → notarize → staple → optionally create a distribution DMG → sign DMG → notarize DMG → staple DMG. The DMG created for a PKG uses the signed `.pkg` path as its single source, with all advanced layout options available.
- **Distribute All with per-file DMG customization (v5.0+).** `BatchDistributionCoordinator` fans out through the full file list, routing each file to the appropriate workflow (`AppDistributionService` or `PkgDistributionService`) with its own `DMGFileSettings`. Checkpoint/resume mirrors the batch signing engine.
- **Workflow checkpoint resume (v4.0+).** `WorkflowCheckpointStore` persists completed step IDs and execution context (credential snapshot, output paths) after every step. At next launch, `PendingWorkflowCheckpointsBanner` appears with a Resume button that restarts from the last completed step without re-executing already-finished work.

### Certificate Management

- **Workflow-aware auto-select (v4.7+).** Distribution dialog openings trigger `bestIdentity(for:)`, which filters identities by workflow type, excludes expired certificates (but not expiring-imminently ones), and prefers the identity last used for that specific workflow.
- **Per-workflow identity history.** App distribution and PKG distribution each maintain an independent last-used identity key, preventing cross-workflow history pollution.
- **Certificate Status Pill.** The selected Developer ID identity displays a days-until-expiry pill with escalating color: neutral ≥ 90 days; advisory 30–89; warning 7–29; error < 7 or expired.
- **Expiry monitoring and notifications.** `CertificateLifecycleMonitor` evaluates every discovered Developer ID identity on launch and on a daily schedule. Approaching-expiry conditions post macOS User Notifications (permission requested on first launch) and surface inline banners in the distribution dialogs.

### Working Folders

- **Named project folders** that group related files together for batch operations. Persistent across launches via `WorkingFolderManager`.
- **Sign All (v4.8+).** Signs every unsigned file in the folder, routing each to the correct certificate class automatically. Spinner feedback during signing; status pills update in place after completion.
- **Sidebar integration.** When Working Folders mode is active, the main view adopts a two-column layout with folders in the leading sidebar. A direct toolbar button (v5.0+) toggles the sidebar without navigating to the overflow menu.

### Submission History & Analytics

- **Submission History Browser (v4.6+).** Structured, searchable record of every operation, browsable from the Submission Log window. Filter by operation type, search by filename or UUID, toggle failures-only, copy request IDs to the clipboard.
- **Distribution Analytics.** On-device metrics store (`DistributionMetricsStore`) backed by `SubmissionLogger`. Aggregates submission counts, notarization durations, and failure classes. Exportable as CSV or JSON from the Analytics tab in Preferences. Strictly local — no data leaves the device.
- **Retry Policy.** Bounded exponential backoff with jitter around `notarytool submit`, with classification of retryable (5xx, DNS, timeout) versus fatal (4xx, authentication) conditions. Stapler error 65 is retried.

---

## Command-Line Interface

`SignaroCLI` is a native macOS executable built from the same Xcode project as the GUI application. It shares `CodeSigningOperations`, `NotarizationOperations`, `DMGCreationOperations`, `AppDistributionWorkflow`, and `PkgDistributionWorkflow` directly — no separate implementation, no shell-script wrappers. All commands are non-interactive by default and suitable for CI and automated build pipelines.

### Build

```bash
xcodebuild build \
  -project Signaro.xcodeproj \
  -scheme SignaroCLI \
  -destination 'platform=macOS'
```

Verify the build:

```bash
SignaroCLI --version    # → SignaroCLI 5.0.1.3
SignaroCLI --help
```

The embedded variant (CLI binary inside `Signaro.app/Contents/Helpers/`) is built with the `Signaro (Embedded CLI)` scheme using the `Release-Embedded` configuration.

---

### Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Emit a single structured JSON object to `stdout` instead of human-readable text. All commands support this flag. |
| `--help`, `-h` | Print usage with examples and exit 0. |
| `--version` | Print `SignaroCLI 5.0.1.3` and exit 0. |

---

### Commands

| Command | Purpose | Example |
|:---|:---|:---|
| `analyze` | Check signature, notarization, & entitlements | `SignaroCLI analyze MyApp.app --smart` |
| `validate` | Pre-submission readiness check | `SignaroCLI validate MyApp.app --mode quick` |
| `sign` | Sign files (supports split identities) | `SignaroCLI sign MyApp.app --identity-name "..."` |
| `notarize` | Submit, wait for, or log notarization | `SignaroCLI notarize submit MyApp.zip --wait` |
| `staple` | Attach notarization ticket to files | `SignaroCLI staple MyApp.app` |
| `dmg create` | Create customized disk images | `SignaroCLI dmg create --source App.app --icon-size 96` |
| `distribute` | Full E2E pipeline (Sign → Notarize → DMG) | `SignaroCLI distribute app --app MyApp.app` |
| `xcode-phase` | Generate Xcode Build Phase script | `SignaroCLI xcode-phase MyApp.xcodeproj` |

#### `analyze <path> [<path> ...]`

Report the code signature status of one or more files. Evaluates notarization state, signature validity, certificate expiry, and hardened-runtime flags. Supports `.app`, `.dmg`, `.pkg`, and `.mobileconfig`.

```bash
SignaroCLI analyze MyApp.app --smart
SignaroCLI analyze MyApp.app --json
SignaroCLI analyze MyApp.app MyInstaller.pkg --smart --json
```

#### `validate <path> [<path> ...]`

Run a pre-submission notarization-readiness check. Exits `65` if any file fails validation. Pass `--mode quick` for a fast preflight suitable for CI gatekeeping.

```bash
SignaroCLI validate MyApp.app --identity-sha1 ABC123 --json
SignaroCLI validate MyApp.app --mode quick
SignaroCLI validate MyApp.app MyInstaller.pkg --mode quick --json
```

#### `sign <path> [<path> ...]`

Sign one or more files. When the selection mixes `.app`/`.dmg` and `.pkg`/`.mobileconfig` files, each file is signed with the certificate class that matches its type. Pass `--clean-attributes` to strip extended attributes before signing.

```bash
SignaroCLI sign MyApp.app --identity-name "Developer ID Application: Acme (TEAMID)"
SignaroCLI sign MyApp.app MyInstaller.pkg \
  --app-identity-name "Developer ID Application: Acme (TEAMID)" \
  --pkg-identity-name "Developer ID Installer: Acme (TEAMID)" \
  --clean-attributes
```

#### `staple <path> [<path> ...]`

Attach a notarization ticket to one or more previously notarized files.

```bash
SignaroCLI staple MyApp.app MyInstaller.pkg
SignaroCLI staple MyApp.app --json
```

**Deferred stapling by UUID (v4.6+):** Polls a known notarization submission UUID until Accepted, then staples. Exits `65` on rejection; `69` on credential or service failure; `64` on malformed UUID.

```bash
SignaroCLI staple --uuid <request-id> MyApp.app --keychain-profile MyProfile
SignaroCLI staple --uuid <request-id> MyInstaller.pkg --keychain-profile Jay_SIGNARO --timeout 20 --poll-interval 20
```

#### `xcode-phase <path.xcodeproj>` (v4.6+)

Read an Xcode project's active build settings via `xcodebuild -showBuildSettings` and emit a ready-to-paste Run Script Build Phase that calls `signaro distribute app` with the correct `PRODUCT_NAME`, `DEVELOPMENT_TEAM`, `CODE_SIGN_IDENTITY`, and `PRODUCT_BUNDLE_IDENTIFIER` values. Pass `--json` to receive `productName`, `teamID`, `identity`, and `bundleID` as structured output.

```bash
SignaroCLI xcode-phase MyApp.xcodeproj
SignaroCLI xcode-phase MyApp.xcodeproj --json
```

#### `notarize submit <path>`

Submit a file to Apple's notarization service and print the request ID. Supports Apple ID + app-specific password, Keychain Profile, and App Store Connect API Key credential modes. Pass `--wait` to block until Apple returns a verdict.

```bash
SignaroCLI notarize submit MyApp.zip --keychain-profile MyProfile --wait
SignaroCLI notarize submit MyApp.app --keychain-profile Jay_SIGNARO --wait
SignaroCLI notarize submit MyApp.zip \
  --key-id KEYID \
  --issuer-id ISSUERID \
  --key-path ~/.private_keys/AuthKey_KEYID.p8
```

#### `notarize wait <request-id>`

Poll a previously submitted notarization request ID and exit when Apple returns a verdict. Exits `65` on rejection.

```bash
SignaroCLI notarize wait <request-id> --keychain-profile MyProfile
SignaroCLI notarize wait <request-id> --keychain-profile Jay_SIGNARO --timeout 20 --poll-interval 20
```

#### `notarize log <request-id>`

Retrieve Apple's notarization log for a completed submission. Useful for diagnosing rejection reasons.

```bash
SignaroCLI notarize log <request-id> --keychain-profile MyProfile
SignaroCLI notarize log <request-id> --keychain-profile Jay_SIGNARO --json
```

#### `dmg create`

Create a compressed DMG from a source file or directory. Supports the full advanced customization pipeline: background image, volume icon, window size, icon positions, encryption, and segmentation.

```bash
SignaroCLI dmg create \
  --source MyApp.app \
  --output ~/Desktop/MyApp.dmg \
  --volume-name "My App 2.0" \
  --background Resources/background.png \
  --volume-icon Resources/AppVolume.icns \
  --icon-size 96 \
  --text-size 12 \
  --icon-x 180 \
  --icon-y 170 \
  --window-width 560 \
  --window-height 380
```

#### `distribute app`

Full App Distribution workflow: sign → notarize → staple → create DMG → sign DMG → notarize DMG → staple DMG. The input must be an `.app` bundle. Pass `--skip-notarize` to produce a signed, un-notarized DMG (useful for offline development workflows).

```bash
SignaroCLI distribute app \
  --app MyApp.app \
  --identity-name "Developer ID Application: Acme (TEAMID)" \
  --keychain-profile Jay_SIGNARO \
  --output-dir ~/Desktop \
  --volume-name "My App Installer" \
  --background Resources/background.png \
  --volume-icon Resources/AppVolume.icns
```

```bash
SignaroCLI distribute app \
  --app MyApp.app \
  --identity-sha1 ABC123 \
  --keychain-profile MyProfile \
  --output-dir ~/Desktop \
  --skip-notarize
```

#### `distribute pkg`

Full PKG Distribution workflow: sign `.pkg` with `productsign` → notarize → staple → (optionally) create a distribution DMG containing the signed package. The input must be a `.pkg` file. Use `--create-dmg` to enable the DMG step; `--icon-x` and `--icon-y` position the PKG icon in the DMG layout.

```bash
SignaroCLI distribute pkg \
  --pkg MyInstaller.pkg \
  --identity-name "Developer ID Installer: Acme (TEAMID)" \
  --keychain-profile Jay_SIGNARO \
  --create-dmg \
  --volume-name "My Installer" \
  --background Resources/pkg-bg.png \
  --icon-x 260 \
  --icon-y 170
```

### End-to-End Example (Profile-Based)

```bash
# 1) Store credentials once
xcrun notarytool store-credentials Jay_SIGNARO \
  --apple-id you@example.com \
  --team-id TEAMID

# 2) Sign
SignaroCLI sign MyApp.app \
  --identity-name "Developer ID Application: Acme (TEAMID)" \
  --clean-attributes

# 3) Submit (non-blocking)
SignaroCLI notarize submit MyApp.app --keychain-profile Jay_SIGNARO

# 4) Wait for verdict
SignaroCLI notarize wait <request-id> --keychain-profile Jay_SIGNARO

# 5) Staple and verify
SignaroCLI staple MyApp.app
xcrun stapler validate MyApp.app
spctl --assess --type exec --verbose MyApp.app
```

---

## Notarization Credential Modes

| Mode | Input Required | Recommended For |
|:---|:---|:---|
| **App-Specific Password** | Apple ID + Password | Local development |
| **Keychain Profile** | Profile Name | Local CI / Single agents |
| **API Key (.p8)** | Key ID, Issuer ID, .p8 file | Scalable CI / Headless servers |

**Apple ID + App-Specific Password**
Generate an app-specific password at [appleid.apple.com](https://appleid.apple.com). Suitable for personal development machines. Not recommended for CI environments where the Apple ID should not be persisted.

**Keychain Profile (`notarytool store-credentials`)**
Store credentials once: `xcrun notarytool store-credentials MyProfile --apple-id you@example.com --team-id TEAMID`. Signaro references the profile name at submission time. Suitable for local development and single-machine CI agents where the login keychain is available.

**App Store Connect API Key (`.p8`)**
Create a key in App Store Connect under Users and Access → Integrations → App Store Connect API. Download the `.p8` file and provide the Key ID and Issuer ID. Suitable for CI/CD agents, headless build servers, and restricted accounts where an Apple ID should not be stored.

---

## System Requirements

- **macOS 13.5 (Ventura)** or later. Universal Binary (Apple Silicon and Intel).
- **Xcode Command Line Tools** (`xcode-select --install`). Required for `codesign`, `productsign`, `notarytool`, `stapler`, and `hdiutil`.
- **Apple Developer Account** with a valid Developer ID Application and/or Developer ID Installer certificate. Code signing and notarization require a paid developer program membership.
- **Automation permission for Finder** (macOS Security & Privacy). Requested automatically on first DMG creation with custom layout. Required for the Finder AppleScript `set position of item` calls that apply icon positions and window geometry to DMG volumes.

---

## Troubleshooting

**"Signaro would like to control Finder" prompt.**
Expected on first run of any DMG with custom layout options. Click OK. This permission allows Signaro to set icon positions and window geometry via Finder AppleScript after mounting the DMG. If you previously denied the request, re-enable it in System Settings → Privacy & Security → Automation.

**"File is already stapled."**
Handled as success in v3.5+. Re-running a workflow on a file that was previously notarized and stapled produces a clean ✅ for the staple step and continues to the next step.

**Entitlement or provisioning profile mismatch.**
Open More (···) → Entitlement Inspector… and drop both the signed `.app` and the `.mobileprovision` profile. Keys present in one but absent from the other are highlighted in orange. Common causes: entitlements declared in the profile but not added to the `.entitlements` file in Xcode, or vice versa.

**Mixed signing selections behave unexpectedly.**
Signaro signs each file with the certificate class that matches its type (`.app`/`.dmg` → Developer ID Application; `.pkg`/`.mobileconfig` → Developer ID Installer). Distribution workflows require a homogeneous selection — if you mix `.app` and `.pkg` files and then press Distribute, the preflight alert will offer to open the Working Folder Manager, where you can organize the files and run separate distribution passes.

**Notarization returns "in progress" for longer than expected.**
Apple's notarization service processing time varies. Signaro polls every 30 seconds for up to 30 attempts (15 minutes total). If the submission is still active after the timeout, the request ID is displayed so you can resume later with `SignaroCLI staple --uuid <id> <path> --keychain-profile MyProfile`.

**"No request ID available for status checking" on skip-notarize workflows.**
Fixed in 5.0 Build 1.1. Update to the latest build.

**Certificate auto-select shows no identity even though a valid certificate exists.**
If the certificate's `CertificateLifecycleStatus` evaluates to `.expiringImminently`, Signaro 5.0 Build 1.0 incorrectly excluded it from the candidate pool. Fixed in 5.0 Build 1.1. An amber banner confirms the expiring-soon state while still allowing the identity to be used.

---

## Architecture Overview

Signaro is structured around a strict separation between the operations layer (which both the GUI and CLI consume) and the presentation layer (which is GUI-only).

```
Signaro.app (SwiftUI + MVVM)          SignaroCLI (Foundation + CoreGraphics)
      │                                         │
      └──────────────┬──────────────────────────┘
                     │ Shared Operations Layer
     ┌───────────────┼────────────────────────────┐
     │               │                            │
CodeSigningOps  NotarizationOps          DMGCreationOps
ProcessRunner   AppDistributionWorkflow  PkgDistributionWorkflow
IdentityManager CertificateLifecycleMonitor  WorkflowCheckpointStore
SubmissionLogger BatchSigningCoordinator  BatchDistributionCoordinator
```

| Feature | GUI Application | Command-Line Interface |
|:---|:---:|:---:|
| **App Distribution Pipeline** | ✅ | ✅ |
| **PKG Distribution Pipeline** | ✅ | ✅ |
| **DMG Layout Customization** | ✅ (Interactive Preview) | ✅ (Flags & Scripts) |
| **Smart Entitlement Analysis** | ✅ | ✅ |
| **Batch Signing & Checkpoints** | ✅ | ❌ (Single-file focus) |
| **Working Folder Management** | ✅ | ❌ |
| **Submission History Browser** | ✅ | ❌ (`notarize log` only) |
| **Expiry Notifications** | ✅ | ❌ |

Key design constraints:
- All operations execute through `ProcessRunner`, an actor-serialized wrapper around `Process` that prevents concurrent invocations of tools that do not support it (`codesign`, `productsign`, `notarytool`).
- No shared mutable state between the GUI and CLI. Both link the same modules but each binary maintains its own log namespace and credential surface.
- `@MainActor` is applied at the ViewModel and Coordinator level. Operations are `async` functions on the operations layer — called with `await` from the actor.
- `Sendable` conformance is enforced throughout. Configuration structs and result types are value types with no shared mutable references.

---

## Version Information

| Field | Value |
|-------|-------|
| Current version | 5.0 Build 1.3 |
| Build date | 2026-04-27 |
| `MARKETING_VERSION` | 5.0 |
| `CURRENT_PROJECT_VERSION` | 1.3 |
| CLI version string | `SignaroCLI 5.0.1.3` |
| Platform | macOS 13.5+, Universal Binary |
| Architecture | SwiftUI + MVVM, shared operations layer, full CLI parity |
| Test suite | 64 tests across 10 classes in `SignaroTests` |

---

- [GitHub](https://github.com/hov172)
- Slack: **@Hov172** · Discord: **Jay172_**

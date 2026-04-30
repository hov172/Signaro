# Release Notes

## 5.0 Build 1.4 — 2026-04-27

CLI parity completion: four features that previously existed only in the GUI now have full CLI equivalents, along with the GUI wiring that was missing for certificate expiry notifications, distribution workflow logging, and batch signing cancellation.

### New CLI Commands

- **`folder sign <dir>`** Sign all signable files in a directory tree. Auto-routes each file to the correct certificate class (`.app`/`.dmg` → Developer ID Application; `.pkg`/`.mobileconfig` → Developer ID Installer). Flags: `--recursive` (default on), `--dry-run` (report without signing), `--identity <name>` (override auto-routing), `--clean-attributes` (strip xattrs before each sign). JSON output includes per-file status and a summary.

- **`history list`** Browse the local submission history written by `SubmissionLogger`. Returns the most recent entries by default. Flags: `--limit N` (cap result count), `--operation <type>` (filter by operation slug, e.g. `APP_DISTRIBUTION`, `SIGNING`). JSON output surfaces file path, operation, success flag, timestamp, processing time, and Apple response.

### New CLI Features

- **`identities list` expiry classification.** Each identity in the output now includes `expiryStatus` (`healthy` / `expiring_soon` / `expiring_imminently` / `expired`), `expiresInDays`, and `expiresAt` (ISO-8601 date). Human-readable output appends an `[expires in N days — STATUS]` suffix after the identity name. Enables CI pipelines to gate on certificate health before signing.

### GUI Wiring

- **Certificate expiry notifications wired end-to-end.** `notifyExpiringCertificates(evaluations:)` is now called from the daily certificate check callback in `MainContentView`. Delivers a macOS `UNUserNotification` once per certificate per day when a certificate reaches warning (≤ 30 days), imminent (≤ 7 days), or expired status. Gated by the `Signaro_CertExpiryNotificationsEnabled` preference (default on). The Notifications preferences tab toggle now correctly reads and writes this key.

- **Distribution workflow logging wired end-to-end.** `SubmissionLogger.logDistributionRun(...)` is now called at both success and failure exit points in `AppDistributionWorkflow` and `PkgDistributionWorkflow`. Distribution runs now appear in the Submission History Browser alongside signing and notarization entries.

- **Batch signing cancel button wired.** `FileControlsSectionView` now exposes an `onCancelBatch` callback. `MainContentView` passes `{ batchCoordinator.cancel() }` at the call site, so the Cancel button in the batch progress row correctly terminates the signing run.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.3` → `1.4`. `MARKETING_VERSION` remains `5.0`.
- CLI version string bumped `5.0.1.3` → `5.0.1.4`.

---

## 5.0 Build 1.3 — 2026-04-27

Targeted DMG distribution reliability and diagnostics release.

### Bug Fixes

- **DMG Finder layout could open with visible bounds not fully matching the preview/background sizing.** The Finder customization script now explicitly disables the path bar before applying bounds and icon placement (`set pathbar visible of container window to false`), reducing UI chrome variance that altered perceived layout size.
- **Pending workflow checkpoints could persist unexpectedly after discard flows.** Discard handling now routes through an awaited ViewModel API (`discardAllPendingCheckpoints`) so checkpoint state is cleared deterministically before next launch/run-state checks.
- **DMG layout troubleshooting lacked full visibility into script application stages.** DMG creation/distribution paths now emit structured debug markers for customization start, Finder script build, and AppleScript execution, and include script capture in operation output for reproducible diagnostics.

### Documentation

- **README/PRD release metadata refreshed.** Current-version markers, CLI version references, and build metadata fields now track Build 1.3.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.2` -> `1.3`. `MARKETING_VERSION` remains `5.0`.
- CLI version string bumped `5.0.1.2` -> `5.0.1.3`.

---

## 5.0 Build 1.1 — 2026-04-26

Maintenance release. Addresses ten confirmed defects identified during a senior-developer audit following the 5.0 Build 1.0 launch. No behavior changes outside the reported failure paths.

### Bug Fixes

- **App Distribution workflow failed when Skip Notarize & Staple was enabled.** When `skipNotarizeAndStaple = true`, the `.notarizeApp` step correctly returns success and sets `executionState.appRequestId = nil`. The subsequent `.waitForAppNotarization` step then called `waitForNotarization(requestId: nil, ...)` which, instead of short-circuiting, returned a hard failure: "No request ID available for status checking." The guard now returns an `⏭` skip-success result when `requestId` is `nil`, matching the behavior of the `.stapleApp` step under the same flag. File: `AppDistributionWorkflow.swift`.

- **Batch signing and distribution resumed from the wrong file after a failure.** Both `BatchSigningCoordinator` and `BatchDistributionCoordinator` called `saveCheckpoint(completedUpTo: index)` — which persists `completedCount = index + 1` — **before** the failure check. On the next launch, `resume(startingAt: completedCount)` started at `index + 1`, silently skipping the file that had actually failed. The fix moves `saveCheckpoint` to after the failure guard: a failed file does not produce a checkpoint, so the stored `completedCount` from the last successful file is used for the resume index. `pausedOnFailure(failedIndex:)` is corrected from `index + 1` to `index` for consistency. Files: `BatchSigningCoordinator.swift`, `BatchDistributionCoordinator.swift`.

- **Distribute All `.pkg` with "Create DMG" enabled silently skipped DMG signing.** `PkgDistributionWorkflow.signDMG()` requires `config.dmgSigningIdentity != nil` and a `"Developer ID Application"` certificate type — but the `PkgDistributionConfiguration` constructed in `MainContentView.distributeAll()` was assembled without the `dmgSigningIdentity` parameter, leaving it `nil`. The signDMG step returned `⏭ Skipped DMG signing per settings` even when DMG signing was expected. Fixed by passing `dmgSigningIdentity: viewModel.signingIdentity(for: .application)` at the `PkgDistributionConfiguration` construction site. File: `MainContentView.swift`.

- **Working Folder per-file DMG volume icon picker never fired.** `DMGFileSettingsView` registered two `.fileImporter` modifiers on the same view — one for the volume icon (`.icns`, `.png`, `.jpeg`) and one for the background image (`.png`, `.jpeg`, `.tiff`). SwiftUI silently honors only the last-registered `.fileImporter` on a given view, so the icon picker's `isPresented` binding would flip to `true` but immediately snap back with no panel appearing. Both pickers are now implemented as direct `NSOpenPanel.runModal()` calls from the button action, matching the pattern used by the three distribution dialog pickers since 4.1 Build 1.0. File: `DMGFileSettingsView.swift`.

- **Certificate auto-select blocked distribution when the only available certificate was expiring imminently.** `MainContentViewModel.bestIdentity(for:)` filtered the candidate identity list with `case .expired, .expiringImminently: return false`. An expiring-imminently certificate is still valid for signing and notarization; only `.expired` certificates must be excluded. Grouping the two cases introduced a silent failure path where a user with a single, soon-to-expire Developer ID had no identity available for distribution even though the certificate was fully functional. Fixed by separating the cases: `.expired` returns `false`; `.expiringImminently` returns `true` (keeping the identity in the candidate pool and allowing the existing amber-banner warning to display). File: `MainContentViewModel.swift`.

- **PKG Distribution advanced DMG layout was silently ignored.** `DMGCreationOperations.buildAndCustomizeDMG()` derives an `appName` at line 275 to gate the `buildCustomizationScript()` call: `let appName = sourceURL.pathExtension == "app" ? … : nil`. For `.pkg` sources, `appName` was always `nil`, so the `shouldApplyCustomLayout` guard (`if let appName = appName`) never passed and the AppleScript layout step — including icon positions, window size, icon size, text size, and background image placement — was silently skipped. Fixed by providing the full `lastPathComponent` for non-`.app` sources (e.g. `"MyInstaller.pkg"`) instead of `nil`, allowing the layout script to execute for any source type. File: `DMGCreationOperations.swift`.

- **PKG DMG AppleScript item reference appended `.app` unconditionally.** `buildCustomizationScript()` generated the Finder AppleScript item reference as `"\(appName).app"` regardless of the actual source file type. For a `.pkg` source whose `appName` was now `"MyInstaller.pkg"` (after the bug-6 fix), the script would reference `"MyInstaller.pkg.app"` — a nonexistent item — causing a silent Finder position-set failure. The `itemName` derivation now checks `appName.contains(".")`: if the name already carries an extension (as `.pkg`, `.dmg`, `.mobileconfig` do), it is used verbatim; if not (as stripped `.app` bundle names are), `.app` is re-appended. The `iconPositions` key lookup already tried both forms (`appName` and `appName + ".app"`) so no change was needed there. File: `DMGCreationOperations.swift`.

- **PKG Distribution notarization polling loop swallowed Task cancellation.** The `waitForNotarization()` polling loop in `PkgDistributionWorkflow.swift` used `try? await Task.sleep(...)` at all three sleep sites. `try?` discards the `CancellationError` thrown when the enclosing `Task` is cancelled, so a user pressing Cancel during the notarization wait would cancel the coordinator task but the workflow would continue polling for up to 15 minutes (30 attempts × 30 s). Each `try? Task.sleep` is replaced with a `do { try await Task.sleep(...) } catch { return … }` block that returns a cancellation result immediately. File: `PkgDistributionWorkflow.swift`.

- **Auto-clear after Distribute All fired unconditionally, including on total failure.** The `.completed` branch of the `batchDistributionCoordinator.state` observer called `viewModel.clearFiles()` after a 2-second delay regardless of how many files had been successfully distributed. If every file in the batch failed, the file list still cleared — removing the user's selection and the failure evidence before they had a chance to review it. The clear is now guarded by `if success > 0`, so the file list is preserved when the entire batch fails and the user can inspect the results, retry, or investigate. File: `MainContentView.swift`.

- **`BatchSigningCoordinator.cancel()` left the coordinator in the `.running` state.** Calling `cancel()` cancelled the underlying `Task` and nilled `activeTask` and `currentProgress`, but `state` remained `.running`. Any observer checking `batchCoordinator.state` after cancellation would see `.running` indefinitely. The `cancel()` method now sets `state = .idle` after tearing down the task, matching the already-correct behavior in `BatchDistributionCoordinator.cancel()`. File: `BatchSigningCoordinator.swift`.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.0` → `1.1`. `MARKETING_VERSION` remains `5.0`.

---

## 5.0 Build 1.0 — 2026-04-26

Batch distribution engine with per-file DMG customization, Create DMG live preview parity, and toolbar improvements.

### New Features

- **Distribute All with per-file DMG settings.** The Distribute action now fans out through a new `BatchDistributionCoordinator` that processes each file in the selection in sequence. Each file carries its own `DMGFileSettings` record — independent volume name, background image, window size, icon positions, and icon size — so a mixed-app batch produces correctly customized DMGs without re-entering the workflow per file. A live progress row beneath the file list shows the current file name, index, and total count during the run.

- **Checkpoint resume for distribution.** Mirrors the batch signing engine introduced in 4.8: a checkpoint is saved after each successfully distributed file. If the batch pauses on a failure or is cancelled, the `PendingWorkflowCheckpointsBanner` appears at next launch with a Resume button. Resuming restarts from the exact failure point — already-distributed files are not re-distributed.

- **DMGFileSettingsView.** Per-file DMG settings sheet surfaced from the Distribute All pre-flight UI. Gives access to the full Advanced DMG Options panel (background image, window size, icon positions, volume icon) on a per-file basis before the distribution run begins.

- **Create DMG dialog: full layout preview parity.** The standalone Create DMG dialog now exposes the same live preview as the App and PKG workflow dialogs. Background image, volume icon, window size, icon size, text size, custom icon positions, and drag-to-position are all inline in a "Layout, Appearance & Security" disclosure group — no separate sheet. Dragging a tile beyond the window bounds auto-expands the layout. Encryption (AES-128 / AES-256) and segmentation are also inline. `DMGAdvancedOptions` properties changed from `let` to `var` to support direct state binding.

- **Working Folders toolbar button.** The Working Folders sidebar toggle is now a direct toolbar button (sidebar.left icon, accent-colored when active) rather than buried in the ••• overflow menu.

- **Auto-clear file list on completion.** After a sign, unsign, or distribution workflow completes successfully, the file list clears automatically after a 2-second delay, returning the UI to a clean ready state. The delay is visible as a "Clearing selection…" suffix on the completion status message. Working folders are intentionally not cleared.

### Build

- `MARKETING_VERSION` bumped `4.8` → `5.0`. `CURRENT_PROJECT_VERSION` reset to `1.0`.

---

## 4.8 Build 1.0 — 2026-04-25

Batch signing engine, working-folder Sign All, and a pre-flight alert improvement.

### New Features

- **Batch signing engine with per-file progress and checkpoint resume.** The Sign action now fans out through a new `BatchSigningCoordinator` that processes files one at a time, reports live progress (file name, index, total) in a dedicated row beneath the file controls, and saves a checkpoint after each successful file. If signing fails mid-batch, the coordinator pauses on the failed file and records a checkpoint so the run can be resumed from the exact point of failure — files already signed are not re-signed. A `PendingWorkflowCheckpointsBanner` appears at app launch when a saved checkpoint exists, with a Resume button to restart from where the previous run stopped. Cancellation is clean: the Task is cancelled, progress is cleared, and the checkpoint is removed only on successful completion (not on cancel), so interrupted runs are always resumable.

- **Working Folder Sign All.** The "Sign All" button and "Sign All Files" context menu item in the Working Folder Manager and Working Folder Status View now have a full implementation. Clicking Sign All signs every unsigned file in the folder in sequence, automatically routing each file to the correct certificate class: `.app` and `.dmg` files use Developer ID Application, `.pkg` and `.mobileconfig` files use Developer ID Installer. Files with no compatible certificate are skipped without crashing. The button shows a spinner with "Signing…" while the operation runs and briefly shows "Done" on completion; it is disabled when all files are already signed. File signature status pills update in place after the run completes.

- **Pre-flight alert now points to Working Folder Manager.** When the Distribute action detects that the selection contains extra non-target files (e.g., a `.dmg` alongside the target `.app`), the alert previously offered a "Continue Anyway" button. That button implied the extra files would also be processed, which was incorrect — only the primary target file is distributed. The button is now "Open Working Folder Manager," which opens the folder manager where users can organize files into named working folders and run Sign All across a full project.

### Stability

- **Eliminated "Publishing changes from within view updates" runtime warnings.** The `.onChange` handlers observing `batchCoordinator.state` and `batchCoordinator.currentProgress` were mutating `@Published` ViewModel properties synchronously during SwiftUI's render cycle, triggering repeated warnings and undefined behavior. Both handlers now wrap their mutations in `Task { @MainActor in }` to defer to the next run-loop iteration.

---

## 4.7 Build 1.6 — 2026-04-25

Three fixes. Two of them correct the icon-position fix that shipped in Build 1.5.

- **Fixed: DMG icon positions now correctly match the preview (two compounding bugs).** Build 1.5 introduced a Y-axis flip (`windowHeight − y`) based on the incorrect assumption that Finder AppleScript uses Quartz bottom-left origin. Finder's `set position of item` uses top-left origin (Y increases down), the same convention as SwiftUI — no flip is needed. Separately, icon positions inside the preview were being pre-multiplied by `scale` before being placed in a ZStack that already applied `.scaleEffect(scale)`. This double-scaling caused every icon to render at `point × scale²` instead of `point × scale`, so icons appeared at half their intended proportional position in the preview but at the full native coordinate in Finder. Both bugs together meant that an icon dragged to 30 % from the left in the preview would land at 60 % in the DMG, with the Y position also mirrored for non-center placements. Both are now fixed: positions are written to native ZStack coordinates directly and the AppleScript receives them unmodified.
- **Fixed: CLI background image size detection.** `dmg create`, `distribute app`, and `distribute pkg` in the CLI target used `NSImage` (AppKit) to read a background image's pixel dimensions. `NSImage` is not available in the CLI target, so the auto-seed of window size from a background image silently failed. Replaced with `CGImageSourceCopyPropertiesAtIndex` (ImageIO), which is available in both targets and reads the same pixel dimensions.

---

## 4.7 Build 1.5 — 2026-04-25

Two DMG layout preview bug fixes.

- **Fixed: Icon positions now match the layout preview.** The preview stores positions in SwiftUI top-left origin; Finder AppleScript uses bottom-left origin (Quartz). The Y coordinate was being written raw, so every icon landed mirrored vertically relative to where it appeared in the preview. Fixed by flipping Y (`windowHeight − y`) before emitting the AppleScript `set position` command.
- **Fixed: Rulers and grid invisible on dark background images.** Ruler band, tick marks, and grid lines used near-black colors at very low opacity, making them invisible over dark backgrounds. Switched to white-based colors (ruler band `0.18`, ticks `0.55`, grid `0.12`) that remain subtle on light backgrounds and clearly visible on dark ones.

---

## 4.7 Build 1.0 — 2026-04-24

Workflow-aware certificate selection lands in the distribution sheets.

### New Feature

- **App and PKG workflow auto-select.** The App Distribution and PKG Distribution dialogs now open with the best matching Developer ID certificate already selected for that workflow type. Signaro remembers the last-used Developer ID Application certificate separately from the last-used Developer ID Installer certificate, so the two workflows no longer compete for a single global cert history.
- **Visible expired fallback.** If the workflow-specific last-used certificate has expired, Signaro falls back to the best valid replacement and shows an inline amber banner explaining what happened. A macOS User Notification is also posted once per session so operators can notice the fallback even if the dialog is dismissed quickly.
- **Manual override remains in-dialog.** The dialogs keep a compact certificate picker filtered to the compatible certificate type, and the user's explicit selection writes back the workflow-specific history on change and again when the workflow starts.
- **Launch-time notification permission.** Signaro requests notification permission on launch so the expired-fallback notification can be delivered immediately when needed.
- **Split-aware signing and distribution.** The main Sign action now signs each selected file with the certificate class that matches its file type. `.app` and `.dmg` use Developer ID Application, while `.pkg` and `.mobileconfig` use Developer ID Installer. Distribution workflows now require a homogeneous selection so mixed app/install batches are not forced through one certificate path. DMG creation still accepts mixed file sets, but it explicitly calls out that signing is split by workflow type.
- **Interactive DMG preview with accurate background rendering, auto-sizing, and capped auto-expand.** The live DMG layout preview now includes grid, rulers, snap-to-guides, drag-to-position editing, and a hide/show toggle. The preview renders the background image at native 1:1 resolution — matching exactly what Finder shows — rather than scaling it to fill. When a background image is selected, the window dimensions automatically snap to the image's native size so the layout is correct from the first open. Auto-expand is now capped at the background image's native dimensions (or 3840×2160 when no background is set) so dragging an icon cannot produce runaway window sizes. Reset Layout restores to the background image's native dimensions when one is set, and to the standard defaults otherwise. Removing a background image also clears the stored size so the cap and reset always reflect the current state.

### Build

- `MainContentView` now requests notification authorization on first appearance, and the app target continues to build successfully with the embedded CLI run-script sandbox disabled in this environment.

## 4.6 Build 1.3 — 2026-04-22

Three bug fixes and one new feature.

### New Feature

- **PKG Distribution: Full DMG layout parity.** The PKG Distribution workflow now exposes the same DMG visual presentation controls as the App Distribution workflow. An **Advanced DMG Options** disclosure group (collapsed by default, matching the App workflow's UI) contains: volume icon picker, background image picker, custom window layout toggle (window size, icon size, text size), and a custom icon position toggle with x/y controls for the `.pkg` file icon. The shared preview editor also auto-expands the editable bounds when you drag beyond the default window size, so PKG layout tuning no longer requires repeated resize cycles. The CLI `distribute pkg` command also gains `--icon-x` and `--icon-y` flags to position the PKG icon; `--icon-x`/`--icon-y` now work for both app and PKG workflows.

### Bug Fixes

- **DMG window size lost in segmented and encrypted pipelines.** `createSegmentedDMG` and `createEncryptedDMG` both rebuilt `DMGAdvancedOptions` from scratch without forwarding `customWindowSize`, `includeApplicationsAlias`, or `iconPositions`. Any DMG going through those paths silently reverted to the default 540×360 window, lost the Applications alias, and reset icon positions. All three fields are now forwarded in both rebuild sites.

- **Help menu showed "Help isn't available for Signaro."** The macOS Help menu and Cmd+? triggered the system help book lookup, which found nothing registered. Fixed by overriding the help command in `SignaroApp` with a `CommandGroup(replacing: .help)` that posts a notification; `MainContentView` catches it and opens the existing in-app `HelpSheet`.

- **Notarization polling false rejection.** Documented in 4.6 Build 1.2 — carried forward.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.2` → `1.3`. `MARKETING_VERSION` remains `4.6`.

---

## 4.6 Build 1.2 — 2026-04-22

Patch release. One bug fix; no new features or behavior changes.

### Bug Fix

- **Notarization polling false rejection.** The App Distribution Workflow was incorrectly reporting "DMG notarization rejected by Apple" while the submission was still "In Progress". Root cause: `checkNotarizationStatus` appends user-facing help text that includes the word "Invalid" to the status message when Apple's detailed log is not yet available (normal during active processing). The polling loop evaluated `contains("invalid")` before `contains("in progress")`, so it matched the word in the help text and exited with a failure result instead of continuing to poll. Fixed by reordering the status checks — in-progress is now evaluated first, so active submissions continue polling correctly regardless of what the accompanying help text says.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.0` → `1.2` (Build 1.1 was an unreleased internal build). `MARKETING_VERSION` remains `4.6`.

---

## 4.6 Build 1.0 — 2026-04-21

Four new features: two extending the CLI's automation surface and two closing high-frequency workflow gaps in the app.

### New CLI Commands

- **`staple --uuid <id> <path>`** Poll for a known notarization submission UUID and staple the target file once Apple marks it Accepted. Accepts the same three credential modes as `notarize submit` (Apple ID, Keychain Profile, ASC API Key). The intended use case is deferred stapling: submit with `notarize submit` (without `--wait`), let the pipeline continue, then staple in a later step using the request ID. Exits 65 if the submission is rejected; exits 69 on credential or service failure.

- **`xcode-phase <path.xcodeproj>`** Read an Xcode project's active build settings (`xcodebuild -showBuildSettings`) and emit a ready-to-paste Run Script Build Phase that calls `signaro distribute app` with the correct `PRODUCT_NAME`, `DEVELOPMENT_TEAM`, `CODE_SIGN_IDENTITY`, and `PRODUCT_BUNDLE_IDENTIFIER` values. Eliminates hand-authored CI scripts and keeps the invocation in sync with the project's own config. Pass `--project <path>` as an alternative to a positional argument.

### New App Features

- **Submission History Browser** Every notarization, signing, stapling, and distribution operation is now stored as a structured record (in addition to the existing raw text logs) and browsable from a dedicated view. Open it from the Submission Logs window via the new **History** toolbar button. Filter by operation type, free-text search by filename or UUID, and toggle a failures-only view. The detail panel shows the full command line, Apple response, processing time, pre/post signature state, and a one-click UUID copy — useful for feeding into `notarize log` or `staple --uuid`.

- **Entitlement & Profile Inspector** Drop a signed `.app` and a `.mobileprovision` provisioning profile onto the two zones to compare their entitlements side by side. Keys present in one but absent from the other are highlighted in orange; a banner shows the total mismatch count. Single-side inspection also works — drop just the `.app` to enumerate its embedded entitlements, or just a profile to read its Entitlements dictionary. Copy the full entitlements XML from either panel. Access from the More (···) menu as **Entitlement Inspector…**

- **Entitlements in Smart Analysis** When Smart Analysis runs on a selection that includes `.app` bundles, entitlements are extracted asynchronously and surfaced directly inside each file's analysis card. Risky entitlements (get-task-allow, disable-library-validation, allow-unsigned-executable-memory, disable-executable-page-protection, allow-dyld-environment-variables) are flagged in orange with an advisory description. Each card includes an **Open Inspector…** button that launches the full Entitlement Inspector pre-loaded with that `.app`, making the profile-comparison workflow one click away from the analysis results.

### Build

- `MARKETING_VERSION` bumped `4.5` → `4.6`. `CURRENT_PROJECT_VERSION` reset to `1.0`.

---

## 4.5 Build 1.2 — 2026-04-21

Build tooling and distribution pipeline additions. No changes to signing, notarization, or CLI behaviour.

### Xcode Project

- **`Signaro (Embedded CLI)` scheme.** New scheme that builds both Signaro and SignaroCLI and embeds the CLI binary into `Signaro.app/Contents/Helpers/` at build time via a Run Script phase. Uses the new `Release-Embedded` build configuration for Archive/Run so the Organizer's "Direct Distribution" option remains intact on the standard `Signaro` scheme.
- **`Release-Embedded` build configuration.** Identical to Release for all targets; triggers the embed Run Script in the Signaro target.
- **`Signaro (All)` scheme archive fix.** `buildForArchiving` set to `NO` for the SignaroCLI entry so archiving the `Signaro (All)` scheme no longer restricts the Organizer to "Custom" only.
- **`--json validate` exit code fix.** `CLICommandRunner.run()` now propagates `response.exitCode` to the process exit status. Previously `validate --json` always exited 0 even when the payload reported `exitCode: 65`.

### Release Scripts

- **`scripts/release.sh`.** Full release pipeline: embed CLI into exported app, re-sign, notarize, staple, create signed + notarized + stapled distribution DMG.
- **`scripts/create-installer.sh`.** Builds a signed, notarized `.pkg` installer — `Signaro.app` to `/Applications/` and `signarocli` to `/usr/local/bin/`.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.1` → `1.2`. `MARKETING_VERSION` remains `4.5`.

---

## 4.5 Build 1.1 — 2026-04-20

Patch on top of 4.5 Build 1.0. Closes a cluster of `CLIParser` correctness bugs and two UUID pre-validation gaps in the notarize sub-commands; no changes to signing or distribution logic.

### CLI Bug Fixes

- **Boolean flags no longer consume positional arguments.** `knownBooleanFlags` (`--smart`, `--applications-alias`, `--wait`, `--json`, and 14 others) now route through a fast-path that records the flag and advances the token cursor without inspecting the next token. Previously a command like `analyze --smart /path/to/app` silently treated the path as the value of `--smart`, producing an opaque failure.
- **`--key=value` inline assignment is now parsed correctly.** Option flags carrying their value via `=` (e.g. `--identity=ABC123`) are now split on the first `=` before lookup. Previously the whole `--identity=ABC123` token was treated as an unknown flag and discarded.
- **`--` end-of-options terminator is now honored.** All tokens appearing after a bare `--` are passed through as positionals unconditionally. Previously they were still subject to option parsing.
- **`notarize wait`: UUID pre-validated before any network or credential access.** A malformed request ID now exits immediately with code 64 and the message `"notarize wait requires a valid request UUID, got '<value>'"`. Previously any non-empty string bypassed validation and produced an opaque service error.
- **`notarize log`: same UUID pre-validation added for consistency.** An invalid request ID previously fell through to a credentials error, masking the actual problem. It now exits 64 with a clear diagnostic before touching credentials or the network.

### Internal / Code Quality

- **`CLIEntrypoint.swift` → `main.swift`.** Replaced `DispatchSemaphore` + `ExitCodeBox` boilerplate with Swift's native top-level `async` entry point. No behavior change; the file is now 4 lines.
- **`@discardableResult` on `AppDistributionService.execute` and `PKGDistributionService.execute`.** `executeWorkflow` call sites intentionally ignore the return value because state arrives through the event-handler closure. The annotation suppresses the compiler warning at the declaration rather than requiring `_ =` at every call site.

### Build

- `CURRENT_PROJECT_VERSION` bumped `1.0` → `1.1`. `MARKETING_VERSION` remains `4.5`.

---

## 4.5 Build 1.0 — 2026-04-20

Major milestone: **Full CLI Feature Parity**. This release marks the completion of the `SignaroCLI` parity track, ensuring that headless, CI, and server environments have access to the same professional distribution and signing engines as the GUI application.

### CLI Parity Features

- **App Distribution Workflow Parity.** `SignaroCLI distribute app` now handles the full sign → notarize → staple → DMG flow with support for all advanced customization options.
- **PKG Distribution Workflow Parity.** `SignaroCLI distribute pkg` mirror's the App's package distribution logic, including installer signing and optional DMG containers.
- **Intelligent Engine Routing.** The CLI now correctly selects between the simple and professional DMG creation engines based on provided customization flags (backgrounds, icons, etc.), matching the App's `MainContentView` logic.
- **Structured Identity Metadata.** The `identities list --json` output has been enhanced to a full object-based format including expiry dates, trust indicators, and team IDs, enabling detailed certificate auditing.
- **Identity Filter Bypass.** Added `--show-all` flag to `identities list` to allow manual override of certificate filtering.
- **Quick Validation Mode.** Added `--mode quick` to the `validate` command for high-speed CI preflights.

### Engine Improvements & Bug Fixes

- **Multi-File Staging Fix.** Resolved a major regression in the advanced DMG engine where selecting multiple files for a customized DMG would result in a staging failure. The engine now correctly handles and stages multiple source paths.
- **Credential Snapshot Initialization.** Fixed a build regression in the App and PKG distribution dialogs caused by an incorrect `CredentialSnapshot` initializer mismatch.
- **CLI Help Overhaul.** Significantly expanded the CLI help output with detailed examples for every command and comprehensive descriptions for all flags.

### Build

- `MARKETING_VERSION` bumped `4.1` → `4.5`.
- `CURRENT_PROJECT_VERSION` reset `1.1` → `1.0`.

---

## 4.1 Build 1.1 — 2026-04-19

DMG customization that works. Rebuilds the customization pipeline to apply icons, backgrounds, and window layout on the writable mount post-creation.

### What changed

- **Custom volume icons now actually appear on mounted DMGs.** The previous implementation wrote the `kHasCustomIcon` flag on the DMG source folder, which `hdiutil` silently discarded — the icon file was there but the flag wasn't. Fixed by rebuilding the customization pipeline to apply icons, backgrounds, and window layout on the writable mount post-creation.

- **Advanced DMG options in the notarization workflow.** The Sign → Notarize → Staple → Distribute dialog gains a collapsed "Advanced DMG Options" section with controls for:
  - Custom volume icon (.icns)
  - Background image (PNG/TIFF/JPG)
  - Window width/height
  - Icon size (16–128 pt) and text size (10–16 pt)
  - Icon positions for the app bundle and Applications alias

- **Clearer primary action.** Button text is now "Sign, Notarize, Staple & Distribute" so what the workflow does is visible at a glance.

- **Explicit success confirmation.** After a DMG is created you'll see a dialog with the filename, size, and save location.

### Bug Fixes

- **Finder layout (background, window size, icon positions) actually persists.**
- **Volume icon pickers open reliably.** Previous stacked file importers silently shadowed each other; fixed by using direct `NSOpenPanel` calls.
- **AppleScript errors surface in logs instead of disappearing.**
- **Custom App icon position fixed.** The builder now correctly handles both bare names and `.app`-suffixed forms for icon position lookup.
- **Text size is now emitted into the Finder layout script.**

### Note on first use

macOS will prompt *"Signaro would like to control Finder"* the first time you create a DMG with a custom layout. Click OK. You can also manage this later in System Settings → Privacy & Security → Automation.

### Architecture & Tests

- **New pipeline:** `DMGCreationOperations.buildAndCustomizeDMG` orchestrates the UDRW → mount R/W → stamp icon + run AppleScript layout → unmount → `hdiutil convert` flow.
- **Test coverage:** 35 tests total across `SignaroTests`. New regression tests in `AppDistributionWorkflowPresentationTests` and `DMGCustomizationPipelineTests`.

### Build

- `MARKETING_VERSION` bumped `4.0` → `4.1`. `CURRENT_PROJECT_VERSION` bumped to `1.1`.
- Xcode target gains `Signaro/Signaro.entitlements` for Apple Events automation.

---

## 4.0 Build 1.1 — 2026-04-18

Follow-up to 4.0 Build 1.0. Ships the first deferred sub-item of PRD Item 4 (Advanced DMG Customization & Automation) — the `DMGLayoutScript.volumeIconPath` field is now wired end-to-end.

### New Features

- **Custom Volume Icon for DMGs.** `AdvancedDMGOptionsView` → Appearance gains a "Custom Volume Icon" toggle with an `.icns` file picker (filtered to `com.apple.icns`). The selected icon is staged into the DMG source as `.VolumeIcon.icns`, and `kHasCustomIcon` is set on the staging root via `SetFile -a C` (with an `xattr`/`com.apple.FinderInfo` fallback when Xcode Command Line Tools are absent). The flag is baked into the source folder so the custom icon survives a read-only UDZO output. Fully round-trips through `DMGLayoutScript` JSON — load/save/reset apply it alongside window bounds and background.

### Architecture

- `DMGAdvancedOptions` gains `volumeIconPath: URL?`; existing call sites continue to compile because the parameter has a default of `nil`.
- `DMGLayoutScript.toCreationOptions(baseURL:)` now resolves `volumeIconPath` relative to the script's base directory and forwards it through the adapter. The previous `TODO(CR-3600-04)` for volume-icon support is closed; the remaining TODOs (viewStyle, licenseResources, hfsPlus filesystem, deterministicBuild) are still open.
- New staging helper `DMGCreationOperations.stageVolumeIcon(from:in:)` encapsulates the copy + Finder-flag work; failures log under `#if DEBUG` and do not abort DMG creation.
- Legacy `roundTripVolumeIconPath` state in `AdvancedDMGOptionsView` deleted — the real picker replaces it.

### Tests

- New `SignaroTests/DMGLayoutScriptAdapterTests.swift` covers Codable round-trip, adapter forwarding of `volumeIconPath`, a `nil`-omit case, and a background-image regression fence. Executing the suite still requires the one-time Xcode test-target setup documented in `scripts/add-tests-target.md`.

### Build

- `CURRENT_PROJECT_VERSION` bumped from `1.0` → `1.1` (Debug + Release configurations). `MARKETING_VERSION` remains `4.0`.

---

## 4.0 Build 1.0 — 2026-04-18

Major architecture and UI overhaul. 13 of 16 PRD items shipped. Build clean on macOS 13.5+, zero errors, zero warnings.

### UI

- **Sign on default row** alongside Select Files, Analyze, Create DMG, Clear.
- **Distribute as primary accent button** — leads the end-to-end release flow.
- **Advanced disclosure** collapses Unsign, Check Attributes, Notarize, Check Status, Staple Tickets.
- **Toolbar** reduced to three primary verbs (Check Requirements · Distribute · Create DMG) + a `•••` overflow menu (Credentials, Working Folders, Logs, Preferences).
- **Working Folders sidebar** via `NavigationSplitView` when the mode is enabled.
- **Certificate Status Pill** surfaces days-to-expiration with escalating color.
- **Preferences** expanded to seven tabs: General, Logging, Certificates, DMG Defaults, Analytics, Notifications, Advanced Timing.

### New Features

- **Certificate Expiration Monitoring** — local, on launch + daily, no network.
- **Working Folder Templates** — save/import/export JSON, secrets scrubbed.
- **Distribution Analytics** — NDJSON-backed metrics, 90-day retention, CSV/JSON export.
- **Retry Policy** around `notarytool submit` — exponential backoff + jitter, retries 5xx/DNS/timeout.
- **Workflow Checkpoints + Resume Banner** — interrupted App/PKG workflows resurface on next launch.
- **DMG Layout Authoring** — declarative `DMGLayoutScript` JSON; Load/Save in Advanced DMG Options, with a shared live preview that can auto-expand the editable bounds and a preview reset now exposed in Preferences.
- **`os_signpost` instrumentation** on every `ProcessRunner.run` invocation and the notarization wait.
- **`SignaroTests` scaffold** with initial tests for `MainContentViewModel`, `FileOperationsManager`, `RetryPolicy`.
- **`Localizable.xcstrings` catalog** — ~940 strings auto-extracted from SwiftUI literals.

### Architecture

- **`SignaroTheme`** design tokens (Spacing, Radius, Palette, Typography, Stroke) replace inline literals.
- **Four oversized view files decomposed** under `Signaro/Views/`: `NotarizationDialogs.swift` (1,906 → 7), `BasicViews.swift` (1,471 → 6), `FileStatusRow.swift` (1,230 → 4), `SectionViews.swift` (1,157 → 5). Each original reduced to an 8-10 line shim.
- **Service-layer modules** under `Signaro/Services/`: `CertificateLifecycleMonitor`, `WorkingFolderTemplateStore`, `DistributionMetricsStore`, `WorkflowCheckpointStore`, `DMGLayoutScript`, `DMGLayoutLoader`. Plus `RetryPolicy` in Utilities. All `Sendable`-correct under Swift 6 strict concurrency. Zero network calls.

### Bug Fixes

- **Inspector crash revert** — App/PKG distribution dialogs reverted from `.inspector` to `.sheet` so `@Environment(\.dismiss)` behaves reliably again.
- **Concurrency & cast warnings** — redundant `any Error → NSError?` cast in `NotarizationOperations` removed; Swift 6 actor-isolation fix in `WorkingFolderManager` (`nonisolated static let`); `@Sendable` data race in `DistributionMetricsStore` NDJSON reader rewritten.
- **Adapter fixes** in `DMGLayoutScript.toCreationOptions(baseURL:)` — visibility demoted to `internal`, `DMGAdvancedOptions.EncryptionOptions.none` fully-qualified, optional unwrapping for `windowPosition`/`customWindowSize`/`iconPositions`, full struct rebuild on `applyScript(_:)`.

### Deferred to 4.1

- **`SignaroCLI`** — headless command-line peer.
- **Full localization sweep** — wrap remaining `String`-typed API parameters and `NSAlert` usages in `String(localized:)`.
- **Inline Workflow Panel** — rewrite dialogs without `@Environment(\.dismiss)` so they can live inside a native `.inspector` column on macOS 14+.

---

*Full scorecard and per-item commits documented in `PRD.md` → Implementation Status.*

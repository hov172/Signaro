# Release Notes

## 5.5 Build 1.7.7 — 2026-07-05

### New

- **iOS re-sign tab: certificate-lifecycle parity with the macOS tab.** The signing-identity row gains the urgency-only status pill, the consolidated *type · trust · exact expiry* metadata line, the Renew… menu (built-in CSR generation + portal deep link), and the orphaned-certificate hint — all via the shared components (`CertificateStatusPill`, `CertificateMetadata`, new shared `CertificateRenewal`), retiring the tab's old 14-day text tags for the selected identity in favor of the 90-day lifecycle model. Files: `IPAResignView.swift`, `CertificateViews.swift`.
- **Auto mode describes the identity that will actually sign.** New `IPAResignViewModel.autoResolvedIdentity`: when every queued IPA resolves to the same identity, the picker row shows its pill/metadata/Renew… (explicit selection wins; mixed resolutions defer to per-card rows; computed from analyses directly so card expansion cannot hide it). Unit-tested (5 cases incl. the expansion regression). File: `IPAResignView.swift`.
- **The detected `.mobileprovision` gets a picker-level banner.** New `autoResolvedProfileExpiry` (common profile across the queue, surfacing the earliest expiry) drives a severity-tinted banner: `.mobileprovision` capsule chip, profile name (full name on hover), "Expires *date* — N days" via new pure `CertificateMetadata.profileExpiryText`, and an inline "Regenerate in portal…" link inside the 90-day window. Redundant "Profile: …"/"Identity: …" hint rows stand down when the banner/metadata line carry the same fact. Files: `IPAResignView.swift`, `CertificateViews.swift`.
- **Expired provisioning profiles are a hard stop.** The auto-matcher always filtered expired profiles, but a drag-in `.mobileprovision` override or cached analysis could carry one through to a re-sign that verified locally and died at install ("provisioning profile has expired") — the last member of the signs-cleanly-dies-later family. New pure `IPAResignService.expiredProfileBlockReason` (profile name + expiry date + full remediation in the message) enforced in `analyze()`'s resolution, `evaluateProvisioningBundle` (nested overrides), and `signTarget`; `SignaroCLI ios analyze`/`ios resign` inherit it. Files: `IPAResignService.swift`.

### Changed

- **Profile "Regenerate in portal…" trigger aligned to the 90-day model** (was a 14-day window that hid the link while the pill beside it warned). New pure `CertificateMetadata.profileRegenerationPrompt` maps the shared lifecycle thresholds. Files: `CertificateViews.swift`, `IPAAnalysisCard.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.7`. `MARKETING_VERSION` `5.5`. CLI version string `SignaroCLI 5.5 Build 1.7.7`.
- Suite now 218 tests across 27 classes (all green). Strings catalog synced (adds diagnostic strings; removes the unused "Selected: %@ certificate" key).

---

## 5.5 Build 1.7.6 — 2026-07-05

### New

- **Orphaned-certificate detection.** A signing certificate whose private key is missing is invisible to every identity enumeration in the app (`kSecClassIdentity` only returns cert+key pairs) while Keychain Access shows it plainly — the classic multi-Mac case, and the failure mode of the Renew… flow when the issued `.cer` is opened on a different Mac than the one that generated the CSR. New `KeychainOrphanScanner` (app target): enumerates raw `kSecClassCertificate` entries (metadata only), filters to Apple signing-cert subject prefixes (intermediates/roots such as "Developer ID Certification Authority" excluded), and diffs DER SHA-1s against the identity list. Expired orphans are dropped — a key-less expired cert is cleanup clutter, not a signing blocker. UI: an orange one-line hint under the certificate picker when orphans exist, and a "Certificates Missing Their Private Key" section in the stethoscope diagnostic with export-`.p12`/renew guidance. Classification is pure and unit-tested (9 cases). Files: `KeychainOrphanScanner.swift`, `CertificateViews.swift`.
- **Renewal keys tracked to completion.** The diagnostic's new "Renewal Keys Awaiting a Certificate" section lists Renew…-generated private keys whose public key matches no certificate in the keychain — an in-flight renewal is visible, an abandoned one is identifiable and safe to delete. The Open Keychain Access button now appears for orphan findings too. Files: `KeychainOrphanScanner.swift`, `CertificateViews.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.6`. `MARKETING_VERSION` `5.5`. CLI version string `SignaroCLI 5.5 Build 1.7.6`.
- New files: `Signaro/Services/KeychainOrphanScanner.swift` (Signaro target only), `SignaroTests/KeychainOrphanScannerTests.swift`. Suite now 199 tests across 26 classes (all green).

---

## 5.5 Build 1.7.5 — 2026-07-05

### New

- **Consolidated certificate metadata line.** The picker's "Selected: \<type\> certificate" and "This certificate is trusted." rows are replaced by one line: *type · trust · exact expiry* — "Developer ID Application · Trusted · Valid until Jul 5, 2027". Severity tracks the worst of trust and expiry: amber with date + countdown inside the 90-day window ("Expires Aug 6, 2026 — 32 days"), red when expired or expiry is unknown, and "NOT trusted" escalates even a date-healthy certificate — the summary line can never look healthier than the status pill. New pure `CertificateMetadata.line(certificateType:trusted:evaluation:)`, unit-tested (6 cases). Files: `CertificateViews.swift`.
- **Renew… with built-in CSR generation.** When the selected certificate leaves the healthy window, a Renew… menu appears next to the picker. *Generate CSR & Open Portal…* generates a 2048-bit RSA key pair permanently in the login keychain (labeled "Signaro renewal key — \<cert\>"; permanence is load-bearing — the certificate Apple issues against the CSR pairs with this key to form a working identity), writes a PEM `.certSigningRequest` via save panel, reveals it in Finder, and opens the developer portal's create-certificate page. *Open Developer Portal* is the plain deep link. New `CSRGenerator` (app target): hand-rolled PKCS#10 DER (macOS has no public CSR API) — SHA256withRSA self-signature, CN-only subject (Apple replaces it on issuance). DER primitives tested against known byte vectors; the full CSR is verified end-to-end by `openssl req -verify` in tests. Files: `CSRGenerator.swift`, `CertificateViews.swift`.

### Fixed

- **Certificate status pill never rendered.** The countdown pill (added in 1.7.1's lifecycle work) was placed inside the certificate picker's `Menu` label; the AppKit-backed menu button flattens its label to image + text and silently drops styled subviews, so the capsule never drew — while the adjacent trust icon and name survived, masking the bug. The pill now sits in the picker row after the menu, where SwiftUI owns the rendering, and shows from the advisory window onward ("32 days" … "Expired") — deliberately quiet when healthy now that the metadata line always carries the date. Verified live in the running app. File: `CertificateViews.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.5`. `MARKETING_VERSION` `5.5`. CLI version string `SignaroCLI 5.5 Build 1.7.5`.
- New files: `Signaro/Services/CSRGenerator.swift` (Signaro target only), `SignaroTests/CSRGeneratorTests.swift`, `SignaroTests/CertificateStatusPillTests.swift`. Suite now 190 tests across 25 classes (all green).
- Superseded 1.7.2/1.7.3 artifacts removed from `dist/` (still on their GitHub releases and in git history).

---

## 5.5 Build 1.7.4 — 2026-07-05

### New

- **Expired-certificate hard stop for iOS re-signing.** An expired signing certificate signs cleanly and passes local `codesign --verify`, but the resulting IPA fails to install on every device — a silent-until-runtime failure. Re-signing with one is now **Blocked** at analysis (predicted outcome, with the expiry date and remediation in the message), for nested `.appex`/Watch bundles in the per-bundle prediction, and again at sign time (covers precomputed or stale analyses). New pure decision `IPAResignService.expiredCertBlockReason`: blocks when either the keychain identity's `notAfter` **or** the authoritative DER copy of the certificate embedded in the provisioning profile is past due — the embedded copy catches a stale profile even when keychain metadata is missing. Unknown expiry does not block (matches the macOS auto-select policy); "expires soon" stays advisory (⚠ badge). `ProvisioningProfile` now parses `notAfter` from each `DeveloperCertificates` DER (`developerCertNotAfters`, `certNotAfter(forSHA1:)` — case-insensitive; an undecodable DER still contributes its SHA-1 to the cert-in-profile pre-flight but claims no date). Files: `IPAResignService.swift`, `ProvisioningProfile.swift`.
- **Nested-bundle entitlement preservation (macOS signing).** `signNestedBundles` previously signed helpers/XPC services/extensions without `--entitlements` whenever extraction failed — silently stripping entitlements that existed (the signature verifies; the app breaks at runtime). Extraction is now fail-closed (`nestedEntitlementDecision`: an extraction failure on a bundle that reports entitlements is a hard stop), and after signing each nested bundle the written entitlements are re-read and every intended key verified present. File: `CodeSigningOperations.swift`.
- **Per-bundle post-sign entitlement verification (iOS re-signing).** The intended-vs-written entitlement check (`writtenEntitlementNotes`) now runs for every signed provisioning bundle — nested `.appex` and Watch apps included, not just the main app. Dropped keys or an unverifiable re-read surface as **Degraded** instead of passing silently. File: `IPAResignService.swift`.
- **Certificate revocation blocking in the signing flows.** The OCSP revocation checker (shipped 1.7.3 as an opt-in CLI flag) is now consulted automatically: iOS re-sign analysis runs it concurrently with the per-bundle codesign work and escalates to **Blocked** on an affirmative revocation; `resign()` re-enforces it at sign time (covers precomputed analyses and identity overrides); the notarization readiness checker treats a revoked selected identity as a critical issue, saving a doomed notarize round-trip. The soft-fail contract is preserved end-to-end via the new pure `Verdict.blockReason(identityName:)`: only an affirmative "revoked" verdict blocks — unconfirmed or unavailable revocation status never does. Files: `CertificateRevocationChecker.swift`, `IPAResignService.swift`, `DataModels.swift`.

### Fixed

- **`SignaroCLI validate` hang.** `NotarizationRequirementsChecker.runShellCommand` used a bespoke `Process` + `waitUntilExit` that deterministically lost the exit notification, hanging `validate` forever; it also had a latent pipe deadlock on outputs over 64 KB. It now delegates to `LocalProcessRunner` like the rest of the app. The GUI's comprehensive-validation flow shares this path and is protected by the same fix. File: `NotarizationRequirementsChecker.swift`.
- **Auth-mode picker binding (follow-up to 1.7.3).** `NotarizationCredentials.authModeBinding` now reads and writes the `authModeRaw` storage directly instead of routing through the computed `authMode` property. File: `DataModels.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.4`. `MARKETING_VERSION` `5.5`. CLI version string `SignaroCLI 5.5 Build 1.7.4`.
- New test files `SignaroTests/NestedEntitlementPreservationTests.swift`, `SignaroTests/CertificateRevocationCheckerTests.swift`; suite now 178 tests across 24 classes (all green).

---

## 5.5 Build 1.7.3 — 2026-07-05

### New

- **Connected-device integration (`devicectl`).** New shared service `ConnectedDeviceService.swift` wraps `xcrun devicectl` (Xcode 15+, iOS 17+ devices paired via Xcode/Finder). Two GUI features build on it:
  - **Check This Mac's Devices** — in the UDID coverage section, one click fills the coverage field with the UDID of every device paired with this Mac (deduplicated, preserving hand-typed entries). Rows for known devices are annotated with device name and connection state. Manual pasting remains fully supported. File: `IPAAnalysisCard.swift`.
  - **Install on Device…** — after a successful re-sign of an Ad Hoc/Development/Enterprise IPA, installs the output IPA directly onto a connected device: the ground-truth verification that a re-sign worked. Blocked installs are explained up front (device not connected, UDID not in profile, Developer Mode off). File: `IPAAnalysisCard.swift` (DeviceInstallSheet).
- **CLI `devices list` and `ios install`.** `SignaroCLI devices list` prints every device known to this Mac (name, model, OS, UDID, connection state, Developer Mode). `SignaroCLI ios install <ipa|app> --device <udid-or-name>` installs onto a device via devicectl; exits 69 with devicectl's error when unreachable/rejected. File: `CLICommandRunner.swift`.
- **Read-only App Store Connect device registry (`devices registered`).** New shared service `ASCClient.swift` (ES256 JWT via CryptoKit, no dependencies; GET endpoints only — device registration and profile regeneration deliberately out of scope). `SignaroCLI devices registered --key-id <K> --issuer-id <I> --key-path <p8>` lists the team's registered devices; `--udid <u1,u2>` distinguishes "registered but profile predates it — regenerate profile" from "never registered — registering consumes annual quota". Requires an ASC API key with App Manager/Admin role (403 explained for notarization-only Developer keys). File: `CLICommandRunner.swift`.
- **Certificate revocation check (`identities list --check-revocation`).** New shared service `CertificateRevocationChecker.swift`: one OCSP-enabled SecTrust evaluation per identity. A revoked certificate has valid dates and signs cleanly but fails later at Gatekeeper/notarization — this catches it up front. Soft-fail: only an affirmative "revoked" verdict reports as revoked; network trouble reports "unconfirmed", never a false alarm. Opt-in flag (network operation). Files: `CLICommandRunner.swift`, `CLIModels.swift` (new `revocation` field in identity JSON payload), `CLIParser.swift`.
- **In-app Help additions.** New sections: "Connected Devices & Install on Device" and "Team Device Registry (App Store Connect)" (including step-by-step ASC API key creation with the App Manager role caveat); CLI Identities section documents `--check-revocation`. File: `HelpSheet.swift`.

### Fixed

- **"Publishing changes from within view updates" from the auth-mode picker.** The credential dialogs re-identify their subtree on auth-mode change (`.id(credentials.authMode)`), so the AppKit-backed picker is torn down and rebuilt mid-update — and the new picker re-writes its unchanged selection into the binding while the view update is still in flight, republishing through `authMode`'s `@AppStorage`-backed setter. The five duplicated `Binding(get:set:)` constructions are replaced by one shared `NotarizationCredentials.authModeBinding` whose setter drops no-op writes, breaking the re-entrant publish. Files: `DataModels.swift`, `NotarizationCredentialDialog.swift`, `StatusCredentialDialog.swift`, `NotarizationWorkflowDialog.swift`, `AppDistributionWorkflowDialog.swift`, `PkgDistributionWorkflowDialog.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.3`. `MARKETING_VERSION` `5.5`.
- CLI version string `5.5.1.7.3` (also corrects the stale `--version` output that still reported 1.7.1 in the 1.7.2 binary).
- New files: `Signaro/Services/ConnectedDeviceService.swift`, `Signaro/Services/ASCClient.swift`, `Signaro/Services/CertificateRevocationChecker.swift` (Foundation/Security/CryptoKit-only shared services; included in both Signaro and SignaroCLI targets).
- `CertificateRevocationChecker.swift` uses `@preconcurrency import Security` (framework lacks Sendable annotations; SecTrust is confined to a single evaluation queue).

---

## 5.5 Build 1.7.2 — 2026-06-29

### Fixed

- **Ad Hoc profile misclassification when device list is empty.** Provisioning profiles that carry a `ProvisionedDevices` key but list zero devices were incorrectly classified as App Store. `ProvisioningProfile.distributionType` now checks `hasProvisionedDevicesKey` (whether the key exists in the plist) rather than `!provisionedDevices.isEmpty`, so a zero-device Ad Hoc profile resolves to `.adHoc` correctly. File: `ProvisioningProfile.swift`.
- **Empty-device Ad Hoc/Development warning in analysis card.** When an Ad Hoc or Development profile has no registered devices, the analysis card now shows a warning banner: "No devices registered — this profile has an empty device list. The re-signed IPA cannot be installed on any device." Previously the UDID coverage section was silently suppressed and `codesign --verify` still reported success, meaning Signaro could produce a "Re-signed successfully" IPA that would predictably fail at install time with no user-visible explanation. File: `IPAAnalysisCard.swift`.
- **Debugger attachment blocked with Hardened Runtime + Manual signing.** With `ENABLE_HARDENED_RUNTIME = YES` and Manual code signing, Xcode does not auto-inject `com.apple.security.get-task-allow` for Debug builds. Added `Signaro-Debug.entitlements` containing `get-task-allow = true` and pointed the Debug build configuration at it. The Release configuration retains `Signaro.entitlements` (without `get-task-allow`) to pass notarization. Files: `Signaro/Signaro-Debug.entitlements` (new), `Signaro.xcodeproj/project.pbxproj`.
- **`ProvisionedDevices` type-mismatch false positive.** A malformed profile where the `ProvisionedDevices` key holds a non-`[String]` value (e.g. raw data or a dict) previously set `hasProvisionedDevicesKey = true`, causing the profile to be silently classified as Ad Hoc with an empty device list. The parser now requires a successful `as? [String]` cast before setting the flag. File: `ProvisioningProfile.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.2`. `MARKETING_VERSION` `5.5`.
- CLI version string `5.5.1.7.2`.
- New file: `Signaro/Signaro-Debug.entitlements` (Debug-only entitlements; included in Signaro target, Debug configuration only).

---

## 5.5 Build 1.7.1 — 2026-06-28

### New

- **OTA manifest generation (GUI).** The analysis card for a successfully re-signed IPA now shows an **OTA Manifest…** button when the distribution type is Ad Hoc, Development, or Enterprise (not App Store). Enter the HTTPS URL where the re-signed `.ipa` will be hosted; Signaro generates `manifest.plist` (Apple's `itms-services://` plist format) and `install.html` (a tap-to-install web page) alongside the output file. The `itms-services://?action=download-manifest&url=…` install link is displayed in-app and can be copied. File: `IPAAnalysisCard.swift`; new service: `OTAManifestGenerator.swift`.
- **CLI `--ota-url` flag for `ios resign`.** `SignaroCLI ios resign <ipa> --ota-url <https://…/app.ipa>` generates `manifest.plist` and `install.html` alongside the output IPA and prints the `itms-services://` install link to stdout. Requires HTTPS; limited to a single input IPA per invocation. File: `CLICommandRunner.swift`.

### Fixed

- **IPA routing notification race eliminated.** Opening `.ipa` files via File ▸ Open… (⌘O) or drag-and-drop previously posted `signaroSwitchToIOSTab` and `signaroOpenIPAURLs` as two back-to-back notifications. The tab switch is asynchronous; `IPAResignView` was not yet subscribed when the second notification fired, silently dropping files. `signaroSwitchToIOSTab` is now removed entirely. Pending IPA URLs are stored in `@State private var pendingIPAURLs: [URL]` above the TabView in `SignaroApp`, delivered to `IPAResignView` via `@Binding`, and consumed on `.onAppear` and `.onChange(of:)` — no subscription timing window. Files: `SignaroApp.swift`, `IPAResignView.swift`, `MainContentView.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.1`. `MARKETING_VERSION` `5.5`.
- CLI version string `5.5.1.7.1`.
- New file: `Signaro/Services/OTAManifestGenerator.swift` (shared Foundation-only service; included in both Signaro and SignaroCLI targets).

---

## 5.5 Build 1.7.0 — 2026-06-27

iOS Re-sign UX and safety pass. All changes are to the **iOS Re-sign** tab and underlying service layer.

### New

- **Distribution type selector.** Development / Ad Hoc / Enterprise segmented picker above the queue. Filters profile matching to the chosen type; Auto (default) picks the best available. Files: `IPAResignView.swift`, `IPAResignService.swift`, `ProvisioningProfileStore.swift`.
- **Apple Distribution identities in picker.** Both `Apple Development` and `Apple Distribution` certificates now appear in the signing identity picker, covering Ad Hoc workflows that use a Distribution cert. File: `IPAResignView.swift`.
- **Per-nested-bundle profile status rows with drag-drop override.** App extensions and embedded Watch apps each show their own matched-profile status row in the analysis card. Drop a `.mobileprovision` file onto any row to override the profile for that bundle independently. Files: `IPAAnalysisCard.swift`, `IPAResignView.swift`.
- **Post-resign codesign verification.** After each re-sign, `codesign --verify --deep --strict` is run and the result appears in the analysis card as a green or red shield badge. Files: `IPAResignService.swift`, `IPAAnalysisCard.swift`.
- **Entitlement diff with delta values.** The entitlement diff view (changed/dropped/added keys) now shows the actual value change as a secondary italic line under each changed key (e.g., `false → true`, `removed com.example.group`). File: `IPAAnalysisCard.swift`.
- **Wildcard profile warning.** When a wildcard provisioning profile is auto-selected, a callout warns that capabilities requiring an explicit App ID — Push Notifications, Associated Domains, PassKit, HealthKit — may not be active at runtime. Files: `IPAResignService.swift`, `IPAAnalysisCard.swift`.
- **Expired profile surface.** The "no profile" guide now reports how many matching profiles were found but are expired, with a prompt to re-download fresh profiles via Xcode → Settings → Accounts → Download Manual Profiles. Files: `IPAResignService.swift`, `IPAAnalysisCard.swift`.
- **App Store profile detection.** The "no profile" guide now explains specifically when App Store profiles are installed (but cannot be used for local re-signing) and directs users to install a Development or Ad Hoc profile instead. Files: `IPAResignService.swift`, `ProvisioningProfileStore.swift`, `IPAAnalysisCard.swift`.
- **Ad Hoc UDID coverage check.** When an Ad Hoc profile is selected, a text field in the analysis card accepts one or more device UDIDs and highlights which are and are not covered by the profile's provisioned device list. File: `IPAAnalysisCard.swift`.
- **Entitlement write verification.** After each re-sign, Signaro reads back the entitlements embedded in the signed binary and flags any declared in the profile that did not survive the codesign step. Files: `IPAResignService.swift`, `IPAAnalysisCard.swift`.
- **Refresh Profiles button.** A "Refresh Profiles" button in the signing card forces a re-scan of `~/Library/…/Provisioning Profiles` and re-runs analysis on all queued IPAs — no app restart required. File: `IPAResignView.swift`.
- **Per-IPA resign progress indicator.** During a batch resign, the card of the IPA actively being re-signed shows a spinner, making it clear which file is in progress. Files: `IPAResignView.swift`, `IPAAnalysisCard.swift`.
- **Cert picker expiry warnings.** Signing identities that are expired or expiring within 14 days are labelled with `⚠ EXPIRED` or `⚠ expires soon` directly inside the picker. File: `IPAResignView.swift`.
- **Parallel analysis.** `reanalyzeAll()` now runs analysis for all queued IPAs concurrently via `withTaskGroup`, reducing wait time proportionally to queue length. File: `IPAResignView.swift`.
- **Batch resign summary.** After completing (or cancelling) a batch resign, a summary row above the cards reports the total Valid / Degraded / Failed counts. File: `IPAResignView.swift`.
- **Cancel button during batch resign.** During an active batch, the Clear button is replaced by a Cancel button that stops the batch cleanly after the current IPA finishes signing. File: `IPAResignView.swift`.
- **Multi-IPA profile hint shows aggregate.** When multiple IPAs are queued, the signing card header shows "N of M profiles resolved" (with a warning when any are missing) rather than only the first IPA's profile and identity. File: `IPAResignView.swift`.
- **`get-task-allow` Degraded explanation.** When the only reason for "Predicted Degraded" is `get-task-allow` changing `false → true` (re-signing a distribution build with a Development profile), a contextual callout in the analysis card explains this is expected and the app will install and run normally. File: `IPAAnalysisCard.swift`.
- **Expansion-aware header hint.** When a single analysis card is expanded, the global signing header shows that card's resolved profile + identity. Two or more expanded cards → aggregate of just those cards. Collapsing all cards returns to queue-wide aggregate. Files: `IPAResignView.swift`, `IPAAnalysisCard.swift`.
- **UDID coverage checker for Development profiles.** The Ad Hoc UDID coverage text field is now also shown for Development profiles, which also carry a provisioned device list. File: `IPAAnalysisCard.swift`.
- **FairPlay encrypted binary detection.** `analyze()` now runs `otool -l` on the main binary before reading entitlements. If `LC_ENCRYPTION_INFO`/`LC_ENCRYPTION_INFO_64` shows `cryptid != 0`, analysis is blocked immediately with an actionable message: "FairPlay-encrypted binary — use a Development or TestFlight build." File: `IPAResignService.swift`.
- **Apple TSA timestamp.** Provisioning-bundle codesign calls now use Apple's RFC 3161 timestamp server (`--timestamp`). Signed apps remain installable after the signing cert expires. Falls back to `--timestamp=none` with a degraded note if the TSA is unreachable offline. File: `IPAResignService.swift`.
- **Profile discovery cached.** `IPAResignViewModel` caches discovered profiles (`cachedProfiles`) populated by Refresh Profiles and passed to all service calls. Eliminates N×2 redundant filesystem scans per batch. Files: `IPAResignView.swift`, `IPAResignService.swift`.
- **Cached resolved profile.** `IPAAnalysis.resolvedProfile` stores the matched `ProvisioningProfile`. `resign()` reuses it directly, ensuring the resign uses the same profile previewed in the parity check and skipping a redundant re-match. Files: `IPAResignService.swift`.
- **Sub-bundle signing progress.** During resign, the status text updates per-target: "Signing MyExtension.appex…". Files: `IPAResignService.swift`, `IPAResignView.swift`, `IPAAnalysisCard.swift`.
- **Output collision handling.** If the resigned `.ipa` path already exists, a numeric suffix (`-2`, `-3`, …) is appended instead of silently overwriting. File: `IPAResignService.swift`.
- **Work dir in Finder on failure.** Failed resign results carry a `workDirURL`; the analysis card shows a "Show work dir in Finder" button for direct access to the retained working directory. Files: `IPAResignService.swift`, `IPAAnalysisCard.swift`.
- **Codesign retry.** `runCodesign` retries once with a 500 ms backoff on transient failures (`amfid` timeout, security-framework contention). Retry outcome logged in `BundleReport` notes. File: `IPAResignService.swift`.
- **CLI `ios analyze` / `ios resign` updated.** Both commands now accept `--distribution development|adhoc|enterprise` to constrain profile + cert selection (Apple Development for development; Apple Distribution for ad hoc/enterprise). `ios analyze` output now includes the distribution type alongside the profile name. `ios resign` pre-analyzes each IPA before resigning (reuses cached result to skip redundant unzip), uses collision-safe output naming (`-2`/`-3` suffix), and prints `→ target.appex` per bundle signed. File: `CLICommandRunner.swift`.

### Fixed

- **"Publishing changes from within view updates" runtime warnings eliminated.** Migrated `IPAResignViewModel` from `ObservableObject` + `@Published` to the `@Observable` macro (macOS 14+). Closure properties use `@ObservationIgnored` to prevent spurious re-renders. File: `IPAResignView.swift`.
- **Team ID unreadable downgraded from blocked to degraded.** When the original team ID cannot be read from the embedded profile, the outcome is now `degraded` (re-sign proceeds with a warning) rather than `blocked`, matching the behavior for other non-fatal uncertainty cases. File: `TeamConsistency.swift`.
- **False-positive parity deltas on codesign-injected keys.** `application-identifier` and `com.apple.developer.team-identifier` are always rewritten by codesign from the profile; comparing them against the original produced noisy false deltas. Both keys are now excluded from `EntitlementParityChecker.diff()`. File: `EntitlementParityChecker.swift`.
- **TestFlight IPAs blocked by `beta-reports-active`.** `beta-reports-active` added to `cosmeticAllowList`; TestFlight IPAs no longer receive a false blocked verdict for this key. File: `EntitlementParityChecker.swift`.
- **Production apps blocked by `com.apple.developer.default-data-protection`.** Added to `cosmeticAllowList`; dropping the data protection class entitlement only affects the default class for newly created files — it does not affect installability. File: `EntitlementParityChecker.swift`.
- **Wrong cert type auto-selected for Ad Hoc / Enterprise.** `IdentityResolver` now prefers `Apple Distribution` certs when the profile's distribution type is Ad Hoc or Enterprise (was always preferring `Apple Development`, causing "Predicted Valid" that flipped to failure at resign time). File: `IdentityResolver.swift`.
- **Double unzip per resign.** `resign()` now accepts a `precomputedAnalysis` parameter and skips the internal `analyze()` call when the ViewModel's cached analysis is passed in. File: `IPAResignService.swift`.
- **`expandedURLs` not cleared on card removal.** `remove(url:)` and `clearQueue()` now clear `expandedURLs` entries, preventing the header hint from going blank after a card is removed. File: `IPAResignView.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.7.0`. `MARKETING_VERSION` `5.5`.
- CLI version string `5.5.1.7.0`.

---

## 5.0 Build 1.6.0 — 2026-06-26

### New

- **iOS App Re-signing (iOS Re-sign tab).** A dedicated **iOS Re-sign** tab lets you queue one or more `.ipa` files, auto-detect the correct provisioning profile and Apple Development certificate, preview the result of the re-sign operation before committing, and produce a re-signed `.ipa` ready for side-loading or TestFlight upload. Each queued file shows a collapsible analysis card that reports the matched profile, matched certificate, predicted outcome (Valid / Degraded / Blocked), and the concrete reason for any block — including cross-team conflicts, missing profiles, unauthorized certificates, and entitlement parity violations. Files: `IPAResignView.swift`, `IPAAnalysisCard.swift`, `IPAResignService.swift`, `IPAResignViewModel.swift`.

- **iOS Re-sign Analysis & Auto-Resolution.** On drop or file-pick, Signaro immediately runs a dry-run `analyze()` pass on each `.ipa`: unzips the archive in a temporary directory, reads the embedded provisioning profile and original entitlements, scans both `~/Library/MobileDevice/Provisioning Profiles` and `~/Library/Developer/Xcode/UserData/Provisioning Profiles` for a matching iOS profile (by bundle ID, platform, expiry, and team), resolves an authorized Apple Development certificate from the keychain, validates entitlement parity at the value level (deny-by-default), checks same-team membership for every provisioning bundle including App Extensions and Watch apps, and presents a `predictedStatus` card — all without writing any signed output. Files: `IPAResignService.swift`, `ProvisioningProfileStore.swift`, `IdentityResolver.swift`, `IPABundleClassifier.swift`, `EntitlementBuilder.swift`, `EntitlementParityChecker.swift`, `TeamConsistency.swift`.

- **iOS Re-sign expiry warnings.** The collapsed analysis card row shows an amber ⚠️ badge when either the matched profile or the matched certificate expires within 14 days, and a red 🛑 badge when either has already expired. The expanded card shows exact expiry dates with the same color coding. CLI `ios analyze` also surfaces expiry in its output. Files: `IPAAnalysisCard.swift`, `IPAResignService.swift`, `CLICommandRunner.swift`.

- **Queue management for iOS Re-signing.** Each queued `.ipa` card has an ✕ remove button (hidden during an active re-sign run). A **Clear** button beside the Re-sign button clears all cards and resets in-progress state. Duplicate URLs are silently deduplicated on drop or file-pick. Files: `IPAResignView.swift`, `IPAResignViewModel.swift`.

- **File picker for iOS Re-signing.** A **Choose…** link in the drop zone opens an `NSOpenPanel` filtered to `.ipa` files, as an alternative to drag-and-drop. Files: `IPAResignView.swift`.

- **Unified File ▸ Open… (⌘O) with auto-tab-switch.** A single **Open…** menu item accepts `.ipa`, `.app`, `.dmg`, `.pkg`, and `.mobileconfig` files. Files are automatically routed to the correct tab: `.ipa` files switch to the iOS Re-sign tab and are enqueued; all other supported types switch to the macOS tab and are added to the file list. Selecting a mixed batch switches to the tab for the first file type found. Files: `SignaroApp.swift`, `IPAResignView.swift`, `MainContentView.swift`.

- **`ios analyze` and `ios resign` CLI commands.** `SignaroCLI ios analyze <file.ipa>…` runs the same dry-run analysis as the GUI, printing the matched profile, matched certificate, expiry status, predicted outcome, and resolution guidance for any block. `SignaroCLI ios resign <file.ipa>…` re-signs and writes the output alongside the original (or to `--output <path>`). Both commands support `--identity-name` and `--identity-sha1` to pin a specific certificate instead of auto-resolving. Files: `CLICommandRunner.swift`, `CLIParser.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.6.0`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.6.0`.

---

## 5.0 Build 1.5.5 — 2026-06-08

### Changed

- **Recommendation banner now uses severity-based semantic colors instead of accent blue.** The certificate recommendation banner previously rendered every message on a solid accent-blue background — the same blue used by the primary "Sign, Notarize, Staple & Distribute" action button. With both on screen at once the informational banner competed with the call-to-action and read as if it were also clickable. The banner now derives its color and icon from the recommendation's severity: **red** (❌) for problems that will fail, **amber** (⚠️/🔧) for cautions to fix before signing — including wrong-certificate-for-file-type mismatches, **green** (✅) for good-to-go states, and neutral **gray** (🚀/🔒/💡) for optional next-step tips. All tints are muted (~13% fills) so accent blue is reserved exclusively for the primary action button. File: `CertificateViews.swift`.
- **Validation Mode segmented control uses a muted selected tint.** The selected segment ("Detailed (Recommended)") previously filled with full-saturation accent blue, adding to the blue noise immediately beside the primary button. It now uses a muted blue tint (`accentColor` at 50% opacity) so the bold accent blue stays unique to the call-to-action. File: `ValidationModePickerView.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5.5`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5.5`.

---

## 5.0 Build 1.5.4 — 2026-06-06

### Fixed

- **App archives are distributable again (Mac App Archive, not Generic Xcode Archive).** After Build 1.5.3 added the `Signaro → SignaroCLI` target dependency, every app archive also built the `SignaroCLI` command-line tool. The tool target had no `SKIP_INSTALL` setting and command-line tools default to `SKIP_INSTALL = NO`, so Xcode installed `SignaroCLI` into the archive at `Products/usr/local/bin/`. An archive that contains anything besides the `.app` is demoted from a **Mac App Archive** to a **Generic Xcode Archive**, so the Xcode Organizer offered only **"Distribute Content"** instead of the **"Distribute App"** flow (Developer ID export and notarization). `SKIP_INSTALL = YES` is now set on all three `SignaroCLI` configurations (Debug, Release, Release-Embedded) so the standalone tool no longer lands in the archive. The CLI is still built, and is still embedded and signed into `Signaro.app/Contents/Helpers` by the Release-Embedded phase. File: `Signaro.xcodeproj/project.pbxproj`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5.4`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5.4`.

---

## 5.0 Build 1.5.3 — 2026-06-06

### Fixed

- **Notarization submissions still "In Progress" are no longer reported as rejected.** In the GUI submit path (`submitForNotarization` with `waitForCompletion: true`), the result handler determined success with a single `output.localizedCaseInsensitiveContains("Accepted")` check and sent everything else — including Apple's interim `status: In Progress` — to a branch that appended "Notarization: Apple rejected the submission." Apple reports three distinct states (`Accepted`, `In Progress`, `Invalid`/`Rejected`); collapsing them into a binary caused still-processing submissions (e.g. immediately after `Successfully received submission info`) to be surfaced as rejections. The handler now performs an explicit three-way classification: `Accepted` marks success and fetches the audit log; `In Progress` is reported as still-processing (not a rejection, with the request ID and a re-check hint); only `Invalid`/`Rejected` is reported as a rejection. Classification prefers the authoritative parsed JSON `status` field and, when falling back to raw output, anchors on the `status:` prefix instead of bare `contains("invalid"/"rejected")` so appended audit-log or error text cannot trigger a false rejection. File: `NotarizationOperations.swift`.
- **Embedded-CLI archive no longer fails with "SignaroCLI not found … Build the SignaroCLI target first."** Archiving the *Signaro (Embedded CLI)* scheme runs a Release-Embedded build phase that copies `$BUILT_PRODUCTS_DIR/SignaroCLI` into `Signaro.app/Contents/Helpers` and signs it. The app target had no dependency on `SignaroCLI` and does not link it, so during archive Xcode never built the CLI into the shared products directory (`SignaroCLI` was `buildForArchiving = NO` in the scheme), and the embed phase aborted. Added an explicit **Signaro → SignaroCLI** target dependency so the CLI is always built before the embed phase — including during archive — regardless of scheme flags or parallel-build ordering. File: `Signaro.xcodeproj/project.pbxproj`.
- **Embed-phase code signing is guarded for unsigned builds.** The embed build phase always invoked `/usr/bin/codesign --sign "$EXPANDED_CODE_SIGN_IDENTITY"`, so CI/test builds run with `CODE_SIGNING_ALLOWED=NO` (or no identity) failed at `codesign --sign ""`. The phase now skips signing when signing is disabled or no identity is set, and signs normally for real Developer ID archives. File: `Signaro.xcodeproj/project.pbxproj`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5.3`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5.3`.

---

## 5.0 Build 1.5.2 — 2026-05-27

### Fixed

- **Resolved all "compiler unable to type-check expression" build errors.** A SwiftUI API modernization pass in Build 1.5.1 introduced `foregroundStyle()` throughout the codebase. Several call sites mixed `Color` values (`.red`, `.orange`, `.green`) with `HierarchicalShapeStyle` values (`.primary`, `.secondary`) in the same ternary expression — the two are different concrete types, so the type-checker failed to unify them. All such ternaries are now wrapped in `AnyShapeStyle()` to give the compiler a single erased type. Additionally, bare `.accentColor` dot-shorthand inside `foregroundStyle()` was replaced with the explicit `Color.accentColor` form since `ShapeStyle` has no `.accentColor` member. 45 occurrences fixed across 18 files.
- **Resolved "compiler unable to type-check expression in reasonable time" errors in large view bodies.** `MainContentView.body` (346 lines, 25+ chained modifiers) and `NotarizationViews.body` (443 lines) exceeded Swift's practical type-checking budget. Both were split into private layered computed properties (`coreContent` → `withSheets` → `withObservers` → `withAlerts`, etc.) capping each layer at ≤10 modifiers. Inline `Binding(get:set:)` expressions inside `.alert` modifier chains were extracted to named computed `Binding` properties.
- **Reduced type-checker load from complex ForEach row bodies and ternary clusters.** Eight files had `ForEach` closures or modifier chains where repeated inline ternary expressions (some triple-nested) or multi-argument view calls exceeded the type-checker's expression-complexity budget. Fixes: repeated ternary conditions extracted to named `let` bindings before the view, multi-argument views inside `Menu`/`ForEach` extracted to `@ViewBuilder` helper functions. Files: `NotarizationWorkflowDialog`, `AppDistributionWorkflowDialog`, `PkgDistributionWorkflowDialog`, `DMGCreationDialog`, `CertificateViews`, `FileListSectionView`, `LogViewerDialog`, `SubmissionLogViewer`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5.2`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5.2`.

---

## 5.0 Build 1.5.1 — 2026-05-27

### Fixed

- **Post-credential sheet action can no longer fire after dismiss.** The 0.5-second deferred dispatch that opens a distribution or notarization workflow after credentials are confirmed was a bare `DispatchQueue.asyncAfter` with no cancellation token. If the credential sheet was dismissed before the delay elapsed, the closure still fired and mutated view state — causing ghost sheets to appear. The dispatch is now a stored `DispatchWorkItem` that is explicitly cancelled via `onChange(of: showStatusCredentialDialog)` the moment the sheet closes. File: `MainContentView.swift`.
- **Corrupt resume checkpoints are now logged and discarded instead of silently dropped.** When a batch-signing or batch-distribution checkpoint blob failed JSON decoding (e.g. after a schema change between builds), the `guard let context = try? JSONDecoder()…else { return }` path returned silently — the resume prompt vanished with no indication that progress was lost. Both decode sites now use an explicit `do/catch` that prints the decoding error with the workflow kind and discards the corrupt store entry via `discardCheckpoint`. File: `MainContentView.swift`.
- **Batch signing coordinator logs task abandonment and flushes an in-progress checkpoint on cancellation.** If the `BatchSigningCoordinator` was deallocated before its active task ran, the `guard let self else { return }` exited silently. The workflow UUID is now captured before the guard so abandonment is logged. Separately, when the task is cancelled mid-batch the `execute` loop now saves a checkpoint for the current in-flight file index (not just the last completed one), so resume picks up at the right point. File: `BatchSigningCoordinator.swift`.
- **`codesign` validation no longer blocks the Swift cooperative thread pool.** `checkCodeSigning(appBundle:)` and `checkCodeSigningDMG(diskImage:)` called `process.waitUntilExit()` — a blocking call — directly inside `async` functions. Under batch notarization this starves the cooperative executor. All four `run()`/`waitUntilExit()` pairs are now wrapped in `withCheckedThrowingContinuation` dispatched onto `DispatchQueue.global(qos: .userInitiated)`, moving the blocking wait off the thread pool. File: `NotarizationOperations.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5.1`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5.1`.

---

## 5.0 Build 1.5 — 2026-05-26

### New

- **Step-based workflow progress view in Create DMG dialog.** When "Create DMG" is tapped the configuration form is replaced by a step-list view identical in style to the App Distribution and PKG Distribution dialogs. Each active step — Create DMG, Sign DMG, Notarize DMG, Staple DMG — is listed with a pending circle, live spinner, or checkmark/xmark as it progresses. A banner at the top shows the currently running step with detail text. A final result banner (green/red) appears on completion. The separate success alert popup is gone — result is shown inline with a Done button. Files: `DMGCreationDialog.swift`, `MainContentView.swift`.
- **Sign DMG after creation.** The standalone Create DMG dialog includes a "Post-Creation Actions" section. When a compatible Developer ID Application certificate is selected, a **Sign DMG after creation** toggle appears. Signing runs immediately after a successful `hdiutil` create using `codesign` in-place. File: `DMGCreationDialog.swift`, `MainContentView.swift`.
- **Notarize and staple DMG after creation.** When signing is enabled and notarization credentials are configured, a **Submit for notarization** sub-toggle appears with an optional **Staple ticket** toggle. Notarization submits to Apple with `waitForCompletion: true`; stapling embeds the ticket for offline Gatekeeper assessment. Files: `DMGCreationDialog.swift`, `MainContentView.swift`.
- **Live Apple notarization status and request ID in the step view.** The Notarize DMG step row shows Apple's submission request ID, per-step duration, and the final acceptance or rejection message as returned by `notarytool`. On failure, the first line of Apple's response appears directly in the step row for immediate troubleshooting without opening the Log Viewer. File: `MainContentView.swift`.
- **DMG creation logged to Operation Logs.** The Create DMG step is now recorded as a `DMG_CREATION` entry in `SubmissionLogger`, matching the log format of signing, notarization, and stapling. The full workflow — DMG creation, signing, notarization, stapling — is now traceable as four consecutive entries in the Operation Logs viewer. Files: `SubmissionLogger.swift`, `MainContentView.swift`.
- **Auto-selection of correct certificate when DMG dialog opens.** When the Create DMG dialog is opened, `bestIdentity(for: .application)` is called to pre-select the best available Developer ID Application certificate, regardless of which certificate is active in the main view. This prevents the "Incompatible certificate" warning appearing on open when an Installer or other certificate type is selected. Falls back to the currently selected identity if no application certificate is available. File: `MainContentView.swift`.

### Fixed

- **Multi-file DMG icon layout now positions all items.** `buildCustomizationScript` previously hardcoded `set position` for a single item (`appName`) plus the Applications alias, silently dropping all other entries in `iconPositions`. For multi-file DMGs with custom icon positions, files beyond the first landed wherever Finder placed them. The fix iterates all entries in `iconPositions` and emits a `set position` line for each, applied across all DMG creation paths. File: `DMGCreationOperations.swift`.
- **Create DMG custom name now controls the output filename.** Changing the Volume Name while leaving File Name empty now updates the generated `.dmg` filename instead of keeping the multi-file fallback `Archive.dmg`. File: `DMGCreationDialog.swift`.
- **DMG preview auto-expand no longer mutates layout synchronously.** Standalone Create DMG, App Distribution, and PKG Distribution now defer preview auto-expand size updates to the next main-loop tick, avoiding AppKit layout recursion warnings. Files: `DMGCreationDialog.swift`, `AppDistributionWorkflowDialog.swift`, `PkgDistributionWorkflowDialog.swift`.
- **Notarization pre-flight failures now appear in Operation Logs.** When `submitForNotarization` rejected a file during internal validation (unsigned binary, missing hardened runtime, etc.) it returned early without writing a log entry. The early-return path now calls `SubmissionLogger.logNotarizationSubmission` with `success: false`, so all notarization attempts — including blocked ones — appear in the Operation Logs viewer, consistent with App Distribution failure logging. File: `NotarizationOperations.swift`.

### Build

- `CURRENT_PROJECT_VERSION` `1.5`. `MARKETING_VERSION` `5.0`.
- CLI version string `5.0.1.5`.
- `release.sh`: fixed `cp -R` nesting bug — when `dist/Signaro.app` already existed the script would nest the app inside itself (`dist/Signaro.app/Signaro.app`); a `rm -rf` before the copy prevents this.
- `release.sh`: `SignaroCLI` is now re-signed using `Signaro/SignaroCLI.entitlements` (empty dict) to strip the `com.apple.security.get-task-allow` debug entitlement that Xcode embeds in Release builds and Apple's notarization service rejects.
- `create-installer.sh`: added `--skip-signing` flag to skip code signing, notarization, and stapling for local testing.
- `Signaro/SignaroCLI.entitlements` added — empty entitlements dict used when re-signing SignaroCLI for distribution.

---

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

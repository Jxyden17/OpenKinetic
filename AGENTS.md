# openKinetic — Agent instructions and strategic execution plan

## Project identity
Use openKinetic as the display name and openkinetic for repository and directory identifiers. Proposed npm scope: @openkinetic, with packages @openkinetic/core, @openkinetic/ui and @openkinetic/pose-provider. Verify availability before registering or publishing names; use private workspace packages until confirmed. Do not imply official affiliation with openGym.

## Mission
Build an independent, open-source, privacy-first exercise video analysis application and reusable SDK. Analyze recorded exercise clips locally, identify repetitions and measurable technique findings, and provide timestamped, actionable feedback. After the standalone MVP is released, propose an optional integration into openGym via a fork and upstream pull request. The standalone project must remain useful even if upstream declines.

## Working constraints
- Developer uses Windows and an iPhone 17; no local Mac.
- Target zero additional MVP infrastructure expenditure. Use public GitHub repository standard runners, subject to current quotas and policies. Avoid paid services unless explicitly approved.
- Develop web and shared code on Windows; compile iOS with remote macOS/Xcode GitHub Actions. Produce unsigned device IPA for Sideloadly testing. Free-account signing expires periodically; verify actual installation early. Do not assume CI can test Neural Engine performance.
- No mandatory backend, account, telemetry, cloud AI, or video upload. Bundle model assets for offline operation where licensing permits.
- Do not fork openGym until standalone MVP is complete. Read-only upstream research is allowed earlier.

## Strategic goals
| Goal | Evidence of success |
|---|---|
| Privacy | No network transmission of clips or pose data; offline analysis test passes |
| Sustainability | No mandatory recurring infrastructure or AI API cost |
| Cross-platform | Shared React UI and TypeScript engine; interchangeable pose providers |
| Reliability | Reproducible tests, coach-reviewed findings, explicit uncertainty |
| Adoption | Documented versioned npm SDK and standalone demo |
| Upstream integration | Optional working openGym fork, maintainer discussion and PR |

## Decisions and verification gates
Proposed stack: React, TypeScript (strict), Vite, Capacitor, Vitest, npm workspaces, Apple Vision/AVFoundation in Swift, MediaPipe web provider, optional Android Kotlin/MediaPipe provider. Use Zustand only if useful; avoid unnecessary dependencies. Match upstream versions only after inspection. Choose a permissive SDK license (MIT candidate) after auditing dependencies and model assets; obtain owner approval before final licensing. Do not copy unlicensed code.

Upstream candidate: https://github.com/DuarteSantos8/openGym (confirm this is the intended repository). Earlier research indicated React/Vite/Capacitor, AGPL-3.0 and set-linked video recording/import. These are leads, not a verified current source audit. Inspect package manifests, lockfiles, native plugin conventions, media storage, exercise IDs, contribution policy and license at a pinned commit. Record findings in docs/upstream-audit.md. Also verify project/package name availability and existing projects (PocketCoach, RepCounterSDK, FormCheck are unverified leads). Do not assume those repositories exist or are reusable until checked.

## Architecture
```text
apps/openkinetic/              # React/Vite/Capacitor standalone app
packages/core/              # Pure TypeScript; no DOM or native dependencies
packages/ui/                # Reusable React analysis components
packages/pose-provider/     # Capacitor plugin and web implementation
  src/                     # Typed provider contract
  ios/                     # Swift Vision + AVFoundation
  android/                 # Later Kotlin provider
  web/                     # MediaPipe implementation
tests/fixtures/             # Synthetic or consented landmark fixtures
docs/                      # ADRs, validation, integration, privacy
.github/workflows/         # CI and unsigned IPA build
```
Dependency direction: apps -> UI/core/providers; core never imports UI or native plugins. Provider converts native landmarks to shared schema. Keep video decoding and inference on the native side for iOS; batch timestamped landmarks across bridge, not raw frames. Use browser workers where supported; avoid blocking UI. Do not promise Neural Engine execution: Apple frameworks select hardware; benchmark on device. Keep 2D and estimated 3D distinct; never treat monocular 3D as ground truth.

### Proposed contracts (refine with tests)
```ts
interface Landmark { id: string; x: number; y: number; z?: number; confidence: number }
interface PoseFrame { timestampMs: number; landmarks: Landmark[]; space: 'image2d' | 'world3d'; provider: string }
interface PoseProvider {
  capabilities(): Promise<ProviderCapabilities>;
  estimate(input: VideoSource, options: EstimateOptions): Promise<PoseFrame[]>;
}
interface Finding {
  ruleId: string; status: 'observed' | 'review' | 'uncertain';
  startMs: number; endMs: number; repIndex?: number;
  message: string; evidence: Record<string, number>; confidence: number;
}
```
Define remaining types before implementing. Include cancellation, progress, schema version, coordinate orientation, aspect ratio, units, provider/model version and analysis rule version. Normalize left/right and timestamps. Distinguish missing data from zero. Do not equate provider confidence values without calibration. Separate video source adapters for browser File/Blob and native local URI. Never pass arbitrary remote URLs to native file access. Results should be serializable and contain per-rep measurements, findings and quality warnings; do not persist video by default.

## MVP scope
Import local video, select exercise manually, show recording guidance, analyze on device, display synchronized skeleton overlay, rep count, range of motion and tempo, timestamped findings, confidence and deletion controls. Start with side-view bodyweight squat; then bicep curl and push-up only after squat works. Provide accessible progress, cancellation and error states. Avoid universal good/bad judgments and overall 0–100 scores until validated. No injury prediction, diagnosis, live coaching, automatic exercise recognition, LLM, accounts, cloud sync or mandatory Android native provider in MVP.

## Execution roadmap (indicative 24 weeks at 10 hours/week; revise after spikes)
### M0 — Discovery and foundation (week 1)
- [ ] Confirm upstream repository and audit pinned source, licensing and media workflow.
- [ ] Verify competitor repositories, model licenses, package/name availability.
- [ ] Record architecture decisions, scope, privacy policy and initial README.
- [ ] Initialize npm workspaces, React/Vite app, TypeScript strict, lint and Vitest.
Exit: reproducible Windows setup, documented decisions, no unverified dependency reuse.

### M1 — Build and install spike (weeks 1–2)
- [ ] Add Capacitor and minimal iOS app; generate native project reproducibly.
- [ ] Configure GitHub Actions Windows tests and macOS Xcode build; pin tool versions and use least privilege.
- [ ] Build for iphoneos with signing disabled; package app under Payload/ into IPA; upload artifact.
- [ ] Download on Windows, sign/install with Sideloadly, enable Developer Mode if required, verify launch on iPhone 17.
- [ ] Test native bridge with a simple capability call; document signing/refresh steps and failures.
Exit: actual device launch and bridge call confirmed by owner. If blocked, continue browser work and report blocker; do not claim iOS success.

### M2 — Web pose and core engine (weeks 3–6)
- [ ] Bundle permitted MediaPipe model assets; import local clips and extract timestamped frames.
- [ ] Define canonical pose schema and provider adapter; draw correctly aligned skeleton.
- [ ] Implement vector angles, confidence filtering, smoothing, rep state machine, exercise definition schema.
- [ ] Unit test degenerate vectors, missing joints, orientation, irregular frame timing, incomplete reps and cancellation.
Exit: side-view squat clip produces repeatable landmarks and rep count offline on Windows.

### M3 — Native Apple provider (weeks 5–9)
- [ ] Implement Swift Capacitor plugin with AVFoundation and Vision 2D pose request.
- [ ] Handle rotation, frame timing, local file permissions, cancellation and memory limits.
- [ ] Batch pose results and compare normalized output against web fixtures.
- [ ] Explore Vision 3D separately; keep optional until validated.
- [ ] Benchmark real iPhone 17 throughput, memory, thermal behavior and battery impact.
Exit: local clip analysis on iPhone, no network required; native provider selected when supported, explicit fallback otherwise.

### M4 — Exercise rules and standalone MVP (weeks 8–14)
- [ ] Squat: phase detection, relative depth, tempo and visible torso movement; camera quality gate.
- [ ] Curl: elbow movement, range and tempo; push-up: alignment and depth where visible.
- [ ] Add rule-specific confidence and recording guidance; do not infer hidden joints or load.
- [ ] Build analysis results, playback seek to findings, local history (results only by default), deletion and accessible UI.
- [ ] Test offline, bad clips, multiple people, occlusion, unsupported formats and long clips; define supported clip limits.
- [ ] Release v0.1.0 standalone app with unsigned IPA and installation docs; document known limitations.
Exit: standalone app usable on web and tested iPhone; three exercises only if each passes validation, otherwise ship narrower scope.

### M5 — Validation and SDK (weeks 15–18)
- [ ] Collect consented clips or licensed datasets; never commit private clips or identifiable pose data without explicit permission.
- [ ] Separate development and held-out test subjects; obtain qualified coach labels where possible.
- [ ] Report rep count error, per-rule precision/recall, coverage/abstention and provider differences.
- [ ] Stabilize public API, version results, document integration and build sample consumer app.
- [ ] Publish npm packages after owner approval and name/license verification; contact openGym maintainers with demo before substantial fork work.
Exit: documented SDK and transparent validation report. If coach validation unavailable, label feedback experimental and report blocker.

### M6 — openGym fork (weeks 19–22; after standalone release)
- [ ] Confirm maintainer interest and current upstream commit; fork only with owner approval.
- [ ] Reuse existing set video workflow where verified; map exercise IDs and use existing local media permissions.
- [ ] Add optional Analyze form action, capability detection, analysis view and local result association with set.
- [ ] Do not change sync defaults or upload video/pose data; ensure unsupported devices degrade gracefully.
- [ ] Test upstream regressions and minimal diff; document dependency/version/maintenance impact.
Exit: demonstrable optional integration without mandatory backend or breaking existing workout flows.

### M7 — Upstream proposal (weeks 23–24+)
- [ ] Prepare demo, benchmarks, privacy/accuracy limitations, license compatibility and maintenance plan.
- [ ] Submit discussion/PR only after owner approval; follow contribution guidelines and respond to review.
- [ ] Keep standalone release cadence independent of upstream outcome.
Exit: proposal submitted; acceptance and timing controlled by maintainers, not guaranteed.

## Engineering targets (not measured claims)
- At least 15 sampled pose frames/sec on iPhone 17; 30-second clip processed in under 30 seconds, measured end-to-end.
- At least 95% exact rep-count agreement on supported held-out clips; report sample size and failure cases.
- Aim for 100 consented clips across users, body proportions, lighting and supported camera views; aim for 85% coach agreement per supported finding, with precision/recall and abstention separately reported. These are provisional targets, not clinical validation.
- Aim for 80% core coverage, including boundary tests; zero mandatory network requests during analysis.
- Do not gate success on arbitrary numerical scores or conceal uncertainty. Recalibrate targets after baseline measurements.

## CI, security and privacy
Use lockfile and reproducible npm ci; run lint/typecheck/tests/build on PRs. macOS job builds iOS device app and native tests where possible. Use explicit workflow permissions, trusted pinned actions, no signing credentials in repo or logs, short artifact retention. No paid runners without approval. Keep video/pose data out of logs and analytics. Clean temporary files on success/error/cancel; provide deletion. Treat imported media as untrusted, validate type/size/duration, handle decoding errors. Test airplane-mode operation after install. Never claim safety or injury prevention from a single camera clip.

## Agent working protocol
1. Inspect repository and existing instructions before edits; if empty, start M0. Do not overwrite unrelated user changes.
2. Work one milestone or small vertical slice at a time. Explain intended changes, assumptions and acceptance tests. Prefer simple implementation over speculative abstractions.
3. Verify APIs against installed versions and official docs; do not invent repositories, dependencies, citations or test results.
4. Run available tests and report exact commands/results. If Windows cannot run native tests, configure CI and mark device verification pending. Never mark tasks complete without evidence.
5. Maintain docs/STATUS.md with completed tasks, next tasks, blockers, decisions, test evidence and device verification. Maintain ADRs for architecture changes and update README on behavior changes.
6. Ask before spending money, publishing packages/releases, creating external repositories/forks, submitting PRs, changing licenses or transmitting private data. No secrets in source. Do not push or perform destructive operations without authorization.
7. Preserve offline-first design and stable provider boundary. Do not start later phases merely because earlier ones are blocked; identify independent tasks and obtain approval for scope changes.

## First Codex task
Inspect the working directory and identify whether a repository exists. Report available Node/npm/Git tools and any constraints. Begin M0 with a minimal workspace scaffold, README, docs/STATUS.md and a passing core unit test, if authorized to edit. Verify upstream facts separately; record unknowns instead of guessing. Then prepare M1 build spike. Do not implement the full app or fork openGym in the first session.

## Reference starting points
- https://github.com/DuarteSantos8/openGym (candidate upstream; verify)
- https://capacitorjs.com/docs/plugins/creating-plugins
- https://developer.apple.com/documentation/vision
- https://developer.apple.com/documentation/coreml
- https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker
- https://docs.github.com/en/actions
- https://sideloadly.io/

All timelines, package names, API sketches, exercise priorities and KPIs are proposed decisions. Current upstream details, licenses, costs, IPA compatibility and hardware performance must be verified. No implementation or device testing has been performed as part of this plan.

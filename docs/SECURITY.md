# SECURITY — cybersecurity preparation

> **Status: security prep (June 2026, idea phase).** A threat model + controls for the app, sized to a private friend-group app that *could* open to others. Companion to [`ARCHITECTURE.md`](ARCHITECTURE.md), [`STABILITY.md`](STABILITY.md), [`AI-READINESS.md`](AI-READINESS.md), [`CREATOR.md`](CREATOR.md). Decide now (cheap), build as we go.

## 0. We start with a strong posture

Choices already made are security wins, not afterthoughts: **Sign in with Apple** (no passwords to store/leak), **data-not-code** (no user code to execute — a security boundary *and* an App Store rule), **no ads / no third-party tracking SDKs** (no data brokers, smaller attack surface), **local-first + ~$0 self-run infra** (almost nothing we operate to be breached), and **validate-and-degrade** everywhere. The work below hardens the edges those choices leave.

## 1. Threat model (what we defend, against whom)

**Assets to protect:** the group's private content (chat, photos, events, location-ish "who's home"), member identities, group membership, and the integrity/availability of each group's space.

**Adversaries & top threats:**

| Adversary | Threat | Primary defense |
|---|---|---|
| Outsider | Reach a group's data they're not in | CKShare zone boundary (server-enforced) + RBAC |
| Malicious/compromised member (insider) | Read/alter beyond their role; grief the group | App-level RBAC checked in the data layer; audit log; undo |
| Malicious **imported bundle / kit** | Code exec, data theft, crash, resource bomb | Treat as untrusted data; validate-and-sandbox (§4) |
| Network attacker (MITM) | Intercept/alter traffic | TLS everywhere (ATS), optional cert pinning, E2EE payloads |
| Thief with the phone | Read local-first data at rest | iOS Data Protection (passcode), Keychain, E2EE |
| Account takeover | Hijack an Apple ID | Apple 2FA (required for iCloud); we never hold credentials |
| Abuser (public zone) | Harassment, illegal UGC | Two trust zones, report/block, moderation (§9) |
| Prompt-injection via content | Trick the AI into harmful actions | AI emits validated proposals only; human approves (§5) |
| Resource-exhaustion via rules/animations | DoS / battery drain / OOM | Bounded evaluation + per-frame budgets ([`STABILITY.md`](STABILITY.md)) |

**Out of scope (rely on Apple):** OS/kernel exploits, Apple ID infrastructure, CloudKit/iCloud server security, device-level malware. We assume a non-jailbroken device.

## 2. Identity & access

- **Auth:** Sign in with Apple only; no passwords, no password DB to breach. Apple ID 2FA is effectively mandatory (iCloud). Support **Hide My Email** (private relay) so members needn't share real emails.
- **Membership = CKShare participation.** The zone boundary is the hard wall: a non-participant simply can't reach a group's records, enforced **server-side by CloudKit**, independent of our UI.
- **Invites:** links/codes must be **single-use or expiring, revocable, and unguessable** (high-entropy tokens). Prefer **approve-to-join** over open links. Never put a long-lived secret in an invite.
- **RBAC in the data layer, not the UI.** Every privileged action (edit shared config, publish, moderate, remove member) is re-checked against the actor's role at write time — hiding a button is not a control. Keep roles **light inside a group** ([`VISION.md`](VISION.md) Inversion 4) but real.
- **Member removal caveat (local-first reality):** you can revoke future access (drop CKShare participation), but a removed member may keep **local copies** of past data — you can't un-ring that bell. If we add E2EE, **rotate the group key** on removal so they can't read *new* content (forward secrecy). Document this honestly.

## 3. Data protection

- **At rest (device):** mark sensitive local stores with **iOS Data Protection** (`NSFileProtectionComplete`/`CompleteUnlessOpen`), so data is encrypted under the passcode; keys/tokens in the **Keychain** (not UserDefaults/files). This is the main defense for a lost/stolen phone, alongside Find My remote wipe.
- **At rest (cloud):** CloudKit + R2 are encrypted at rest by the providers.
- **In transit:** TLS for everything (enforce **App Transport Security**, no arbitrary loads). Consider **certificate pinning** for our own R2 upload Worker / any relay.
- **End-to-end encryption (strong option, fits our ethos):** encrypt message/media payloads client-side so even CloudKit/R2/a relay can't read them. CloudKit offers **encrypted fields** (`encryptedValues`); for R2, **encrypt media on-device before upload** (Worker sees ciphertext). Hard part is **key management** — a per-group symmetric key shared only among CKShare participants (wrapped per member). Decide E2EE scope early (at least for media + message text); it turns "trust Apple/Cloudflare" into "trust only group members."
- **Data minimization:** collect the least possible; "who's home in Sandnes" is location-ish — keep it coarse (a day flag, not GPS) and group-scoped.

## 4. The customization-import attack surface (our biggest app-specific risk)

Imported **bundles** (themes, layouts, rules, assets) — especially from other groups or a third-party store — are **untrusted input**. Treat every byte as hostile:

- **Data, not code — enforced.** A bundle can carry no executable code; rules are the bounded DSL, "scripts" are declarative ([`RULES-SPEC.md`](RULES-SPEC.md)). This is the core defense and keeps us inside App Store 2.5.2.
- **Schema validation before anything renders/runs**, then **preview-as-diff + explicit accept** ([`AI-READINESS.md`](AI-READINESS.md) loop). Rules inside an imported bundle are shown for review **before they can run**.
- **Asset hardening** (each a known exploit class):
  - **Images:** cap dimensions/bytes, **downsample**, reject decompression bombs; decode off-main in a failure-isolated path.
  - **Fonts:** malicious font files have historically exploited text engines — validate format, cap size, rely on OS font APIs, sandbox parsing.
  - **Vectors/SVG:** if supported, **strip scripts/external refs** (SVG can embed JS/links) or convert to a safe internal vector; never render SVG in a webview.
  - **Motion (Lottie/Rive):** parse defensively, cap node/keyframe complexity and per-frame cost.
  - **Bundle archive:** cap **expanded size + file count** (zip-bomb defense), and **sanitize asset paths** (no `../` traversal, no absolute paths).
- **Sandbox of effect:** an imported view/rule can only touch **the current group's** data through the permission layer — never another group's zone, never the filesystem, never the network, never role escalation.
- **Resource bounds = DoS defense:** run caps, frame budgets, bounded caches ([`STABILITY.md`](STABILITY.md)) stop a malicious bundle from pegging CPU/battery or OOM-killing the app.

## 5. AI security

- **On-device first** (Apple Foundation Models): the safest path — **no data leaves the device**, so no third-party exposure.
- **Prompt injection** (malicious text in chat/imported content trying to steer the AI): contained by design — the AI **only proposes documents the engine re-validates**, and a **human approves a diff**; it can't silently act, exfiltrate, or reach another group's data (inputs are permission-filtered and minimized). Still: never auto-apply AI changes; show what changed in plain language.
- **External AI (opt-in only):** requires **Guideline 5.1.2(i)** in-app consent + disclosure of which third party and why ([`AI-READINESS.md`](AI-READINESS.md)); send the **minimum** data, never another group's data; surface a clear "external AI in use" indicator.

## 6. Backend & network

- **CloudKit:** never trust the client — permissions are server-enforced; design records so a tampered client can't read/write outside its zone/role.
- **R2 + upload Worker:** the bucket is **private** (no public listing); uploads use **short-lived, scoped presigned URLs** minted by the Worker per object; the Worker is authenticated and **rate-limited**. **No R2 credentials in the app.** Consider **App Attest / DeviceCheck** so only genuine, unmodified app instances can request uploads (anti-abuse).
- **Serverless cron (Workers/Val Town):** secrets in **env vars, never code**; least privilege; no PII in logs.
- **Optional real-time relay (typing/presence):** treat as **untrusted transport** — E2EE so it sees ciphertext, authenticated, rate-limited, stores nothing of record.

## 7. Secrets & developer supply chain

- **No secrets in the repo or app binary:** `*.p8`, `*.p12`, `*.mobileprovision`, API keys, `.env` stay out (already in `.gitignore`). Add **secret scanning** in CI; the repo is **public**, so assume anything committed is exposed forever.
- **Minimal, pinned dependencies** (also a security win — fewer to audit). Review the CRDT lib and any package; pin versions; enable **Dependabot/advisory scanning**; keep an eye on transitive deps. Third-party SDKs (if any) must ship Apple **privacy manifests + signatures** (§8).
- **Build integrity:** protect the signing key / App Store Connect with 2FA; limit who can publish.

## 8. Privacy & compliance

- **GDPR (we're in Norway/EU):** lawful basis + **data minimization**, **right to access/export** (our **bundle/data export** helps) and **right to deletion**, a clear **privacy policy**, and consent for any external processing (AI). Friends' data is personal data — treat it as such even for a hobby app.
- **Apple Privacy Manifest** (`PrivacyInfo.xcprivacy`) is **mandatory since May 2024**: declare data types collected and **required-reason APIs**; any third-party SDK must include its own manifest + signature. Plan to ship one.
- **App Store privacy nutrition labels** consistent with the manifest; with no ads/tracking, ours is minimal — a selling point.
- **Children:** the app targets adults; for any public release, **age-gating** (Guideline 1.2.1) and EU/Norway digital-consent-age handling.
- **Money later (Vipps):** keep the app **out of the money flow** (peer-to-peer only) to avoid PCI/financial-regulatory scope ([`MVP-and-Roadmap.md`](MVP-and-Roadmap.md) §6).

## 9. Abuse, moderation & the public/third-party zone

- **Two trust zones:** warm/light inside a group; **guarded at the edges**. The moment strangers or a shared/third-party store enter, apply UGC duties — **EULA, in-app report/block, content hold/removal, age-gating** ([`PIVOT.md`](PIVOT.md) §6).
- **Third-party store:** since the bundle format is open and a public store can be **third-party**, *we* avoid running/moderating one — but **imports are still validated/sandboxed** (§4) on our side regardless of source.
- **Per-group safety controls:** rule **kill-switch**, one-tap revert, and the audit log give admins fast response to a bad actor or bad automation.

## 10. Incident response & resilience

- **Safe mode / crash-loop recovery** ([`STABILITY.md`](STABILITY.md) §1.2) doubles as security resilience — a malicious bundle that crashes the app is quarantined automatically.
- **Availability / bus factor:** the CKShare zone has one owner — document ownership and keep **periodic encrypted backups/exports** so a lapsed account doesn't lose or expose the group.
- **Audit trail:** the rule-execution + change log (privacy-preserving, on-device/in-zone) supports "what happened and who did it."
- **Vulnerability disclosure:** add a root **`SECURITY.md` / GitHub Security Policy** with how to report privately, before any public release.

## 11. Prioritized checklist

**Decide/architect now (cheap, idea phase):**
- [ ] Commit to **validate-and-sandbox all imported bundles** as untrusted data (§4) — bake into the engine design.
- [ ] Decide **E2EE scope** (at least media + message text) and the **per-group key** model (§3).
- [ ] Design **invites** as expiring/revocable/unguessable, approve-to-join (§2).
- [ ] Put **RBAC checks in the data layer** from day one (§2).
- [ ] Plan the **`PrivacyInfo.xcprivacy`** manifest + a minimal privacy policy (§8).
- [ ] Keep secrets out of the **public repo**; enable secret + dependency scanning (§7).

**Before TestFlight / first real data:**
- [ ] Data Protection file classes + Keychain for anything sensitive (§3); enforce ATS (§6).
- [ ] R2 private bucket + presigned-URL Worker + rate limiting; consider App Attest (§6).
- [ ] Asset-hardening limits (image/font/SVG/motion/archive) implemented (§4).

**Before any public / multi-group release:**
- [ ] UGC report/block/moderation + age-gating + EULA (§9); root `SECURITY.md` disclosure policy (§10).
- [ ] External-AI consent flow (5.1.2(i)) if shipped (§5); GDPR export/delete flows (§8).
- [ ] A lightweight **threat-model review + dependency audit** as a release gate ([`EXECUTION-PLAN.md`](EXECUTION-PLAN.md)).

## 12. One-liner

**No passwords, no user code, no trackers, and (ideally) end-to-end-encrypted data the cloud can't read; every imported bundle is untrusted-and-sandboxed; RBAC and the zone boundary are server-enforced; and the AI can only propose changes a human approves.**

---

### Sources & related
- [Apple — Privacy manifest files](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files) · [Required-reason API (TN3183)](https://developer.apple.com/documentation/technotes/tn3183-adding-required-reason-api-entries-to-your-privacy-manifest) · [Privacy updates for App Store submissions](https://developer.apple.com/news/?id=3d8a9yyh)
- Internal: [`ARCHITECTURE.md`](ARCHITECTURE.md) · [`STABILITY.md`](STABILITY.md) · [`AI-READINESS.md`](AI-READINESS.md) · [`CREATOR.md`](CREATOR.md) · [`PIVOT.md`](PIVOT.md) (UGC duties)

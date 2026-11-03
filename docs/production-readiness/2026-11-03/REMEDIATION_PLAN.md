# SAAF Circuit Breaker — remediation plan

For findings in [REVIEW.md](REVIEW.md). All commands and changes are **future planned work**, not performed. Preserve the local-only boundary until approved; do not connect real refunds.

## Dependencies and phases

| Phase / task | Depends on | Entry → exit |
|---|---|---|
| A / T1 Baseline | Owner access | Pinned source and disposable environment → reproducible baseline recorded |
| B / T2 Report safety | T1 | Malicious report fixture → escaped output and independent negative control |
| C / T3 Access and data | T1, owner exposure decision | Data-flow inventory → approved boundary and denial tests |
| C / T4 Lock and provider | T1 | Concurrent scenario → bounded resource use with atomic state |
| D / T5 Integrity and automation | T1, T2 | Threat model and existing tests → meaningful gated verification |
| E / T6 Operations and UX | T3, T4, T5 | Approved deployment design → observed recovery and accessibility evidence |
| F / T7 Independent acceptance | T2–T6 | Candidate revision → independent go/no-go with evidence |

### T1 — Establish baseline and safe reproduction

**Links:** F3, F5; prerequisite for all changes. **Observed paths:** `package.json`, `saaf-circuit-breaker/pyproject.toml`, `src/db/index.ts`, `tests/`. **Steps:** (1) Check out the pinned SHA in isolation; record toolchain and disposable database/provider configuration. (2) Inspect test and migration scripts; provision synthetic fixtures and nonproduction credentials. (3) Run existing root `npm test`, `npm run typecheck`, `npm run lint`, `npm run build` and the nested configured pytest suite; record commands, outcomes and skips. **Deliverable:** reproducible baseline and failure ledger. **Verification:** another operator repeats it against the same SHA. **DoD:** [ ] No production data or credentials used. [ ] Both project results and skipped tiers recorded.

### T2 — Escape exported Python reports

**Links:** F2. **Prerequisite:** T1. **Observed paths:** `saaf-circuit-breaker/ledger/html_exporter.py`, `saaf-circuit-breaker/cli/main.py`, `saaf-circuit-breaker/tests/test_cli_and_guard_contract.py`. **Steps:** (1) Build valid and invalid imported workpapers with hostile markup in ID, audit standard and displayed event fields. (2) Escape every dynamic text insertion with a context-appropriate HTML encoder, including failure reports, without hiding the failed verification state. (3) Exercise `verify --generate-report` through the CLI and inspect the resulting DOM/text in an isolated browser; keep the nonzero exit for failed verification. **Deliverable:** safe exporter and regression fixtures. **Verification:** deliberately remove escaping and show the attack-oriented assertion fails; restore it and confirm literal text only. **DoD:** [ ] Both success and failure reports escape input. [ ] Exit status remains meaningful.

### T3 — Define access and data protection

**Links:** F1 and preventive privacy/secret requirements. **Prerequisites:** T1 and an owner decision on local-only versus shared access. **Observed paths:** `src/app/api/sentinel/intercept/route.ts`, `src/lib/live-sentinel.ts`, `src/lib/jev.ts`; other API routes require inspection. **Steps:** (1) Map all read/write endpoints, session ownership, CSRF exposure, network binding, sensitive fields and provider transfers; define retention and deletion. (2) For local-only use, document and verify actual loopback and trusted-host boundaries; for shared deployment design authenticated principals, per-session authorization, CSRF protection and least-privilege DB access across all relevant endpoints. (3) Under the shared design, add separate-identity cross-user read/write denial and unauthenticated tests; under the local-only design, test refusal of non-loopback exposure and forged browser-origin requests without inventing unused identities. In both designs, verify secret redaction and synthetic-canary absence in artifacts and logs. **Deliverable:** approved threat model, access implementation if authorized, and data-handling policy. **Verification:** shared-mode tests deny a different identity even with a known session ID; local-only tests demonstrate the approved network/browser boundary and rejection behavior. **DoD:** [ ] Exposure decision and applicable test branch recorded. [ ] Access and deletion behavior demonstrated in the chosen environment.

### T4 — Bound external calls without weakening atomicity

**Links:** F3. **Prerequisite:** T1. **Observed paths:** `src/lib/live-sentinel.ts`, `src/lib/jev.ts`, `tests/sentinel-api.test.ts`. **Steps:** (1) Measure pool occupancy, timeout and latency under same-session and different-session concurrency with a deliberately slow fake Jev. (2) Compare bounded admission with fetching outside the transaction plus version-checked commit/retry; choose a design that prevents stale approvals and keeps timeout or conflict fail-closed. (3) Test conflicts, interrupted commits, provider outage, malformed answers, cap violations and successful persisted rollback of prompt **and** memory. **Deliverable:** measured resource bounds and concurrency-safe design/change. **Verification:** inject delays and failures, inspect actual DB state after rollback, and demonstrate a deliberately disabled version check or lock makes the concurrency test fail. **DoD:** [ ] No stale decision commits. [ ] Resource limits and failure response are recorded.

### T5 — Integrity, dependency and CI gates

**Links:** F4, F5. **Prerequisites:** T1, T2; provenance features require owner approval. **Observed paths:** `tests/workpaper-integrity.test.ts`, `package.json`, `package-lock.json`, `saaf-circuit-breaker/pyproject.toml`; proposed `.github/workflows/` gates. **Steps:** (1) Specify whether consistency-only workpapers suffice; if authenticity is required, design trusted signing, key rotation and verification outside the document's own asserted identity. (2) In both designs, expand independent fixtures for field mutation, rehashed policy violations, malformed traces and both runtimes. Add untrusted-signer, key-rotation and signature-failure fixtures only if signing is selected. The consistency-only design must explicitly label provenance as unverified, not claim authenticity. (3) Audit locked Node and resolved Python dependencies; define reproducible Python dependency resolution and CI jobs for both suites, typecheck, lint and build. **Deliverable:** approved integrity contract, dependency report and application CI. **Verification:** introduce a controlled mutation or broken consistency verifier and require the relevant job to fail; when signing is selected additionally reject untrusted signers. Inspect actual CI results at the candidate SHA. **DoD:** [ ] Chosen consistency/provenance level and applicable tests recorded. [ ] No Pages-only result presented as application CI. [ ] No weakened tests or thresholds to obtain green.

### T6 — Recovery, release and accessible operation

**Links:** F1, F3, F4 and preventive operational/UX requirements. **Prerequisites:** T3–T5 and owner-approved environment. **Observed paths:** `README.md`, `scripts/migrate.mjs`, `src/app/`, `saaf-circuit-breaker/cli/main.py`; proposed operations runbook. **Steps:** (1) Specify DB roles, secret rotation, alerting, release, rollback, retention and migration. (2) In a disposable environment, drill backup, restore, migration and release rollback; verify state and workpaper readback. (3) Test keyboard, screen reader, contrast and HALT/ALLOW/review messaging; clarify no payment or notification occurred. **Deliverable:** runbook, drill evidence and UX fixes. **Verification:** independent operator records restore time and restored records; a user completes the blocked-action flow. **DoD:** [ ] Recovery observed. [ ] Accessibility and demo-boundary results recorded.

### T7 — Independent acceptance

**Links:** all findings. **Prerequisites:** T2–T6. **Observed paths:** changed paths and resulting CI/release evidence. **Steps:** (1) Review the candidate diff and map every finding to implementation, documented risk acceptance or explicit deferral. (2) Independently repeat the key negative tests, cross-user denial where applicable, persisted rollback, provider failure and restore drill; use a separate reviewer and synthetic data. (3) Check actual running revision and release artifacts, then record an owner-signed decision with residual risks. **Deliverable:** acceptance ledger tied to exact SHA. **Verification:** a broken control must make its corresponding independent test fail; test count alone is not evidence. **DoD:** [ ] Every finding has an evidenced status. [ ] No production claim without owner acceptance and observed deployment.

## Production acceptance, approval and rollback

- [ ] Owner approves users, exposure, payment boundary, provider/data agreements, retention and review workflow.
- [ ] Auth and authorization appropriate to exposure, safe reports, fail-closed transitions and reproducible CI are independently verified.
- [ ] Restore, release and rollback drills are observed; provenance requirements and governance claims are separately approved.
- [ ] Real provider behavior and domain safety are validated without presenting a mock as live evidence.

Owner approval is required for public exposure, real credentials or customer data, schema migration against persistent data, signing-key infrastructure, externally actuated actions and release. Ship immutable candidate revisions only after gates pass. On regression, stop traffic or retain loopback, roll back to the prior verified release and use a tested database recovery procedure if schema compatibility permits; never assume a code rollback reverses a migration. Keep evidence and report unresolved recovery constraints instead of silently discarding data.

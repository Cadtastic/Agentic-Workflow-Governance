# Agentic Workflow Governance (AWG)

*A framework for designing, gating, and pinning accountability in workflows where AI agents take action.*

**Version:** 0.2 — skeleton draft (gate primitives drilled; matrix updated)
**Layer:** 1 of 2 (org-agnostic framework; operating models live in Layer 2)
**Scope:** Agency level L1 and above (see §2)
**License:** TBD (Creative Commons recommended)

**Changes since v0.1:** §5 fully rewritten with a three-axis primitive taxonomy and ten atomic primitives; §8 matrix updated to specify primitive coordinates; §10 updated (circuit-breaker question resolved, calibration-scope boundary clarified); glossary extended.

---

## 0. Why this exists

Organizations are increasingly placing AI agents in roles that take action — not just produce text — inside core workflows. Empirical evidence (Wiles, Hsu, Bedard & Kropp, 2026, *Putting AI on the Org Chart*) shows that when AI is framed as an organizational actor with institutional credibility, two things happen simultaneously inside human reviewers:

1. The reviewer's perceived share of responsibility for errors drops (delegation shifts ownership).
2. Unlike with human delegation, no compensating moral-hazard suspicion arises (AI doesn't shirk, so no instinct to monitor it more carefully).

The combined effect is a measurable decline in oversight — in their RCT, ~16% lower review accuracy and a ~44% jump in escalation requests, despite identical underlying work. The mechanism is not anthropomorphism; it is a structural feature of how delegation interacts with the absence of moral hazard.

Existing AI governance frameworks (NIST AI RMF, ISO/IEC 42001, the EU AI Act compliance kits) address model risk, data handling, and high-level lifecycle management. None of them specifically address the *agentic governance gap* this paper identifies. AWG is designed to fill that gap.

---

## 1. Principles

The framework rests on four load-bearing principles. Every component below exists to operationalize one or more of them.

**P1 — Accountability is pinned to a human.**
The named Process Owner is accountable for outcomes regardless of how much the AI did or how many gates approved the action. Governance advises, audits, and certifies; governance does not absorb responsibility. Without this, AWG itself becomes a second layer of diffusion.

**P2 — Gates are structural substitutes for an absent reflex.**
With human delegation, monitoring is partly produced by suspicion of shirking. With agents, that reflex never fires. Procedural gates are not bureaucratic ceremony — they are the engineered replacement for an instinct that does not exist.

**P3 — Autonomy is earned per-tier, not granted globally.**
The default state for L1+ workflows is gated. Ungated autonomy is an explicit, classified, documented decision — never a quiet drift produced by team inertia.

**P4 — Cheap talk does not govern.**
Every rule must be enforceable structurally — through tooling, code paths, RACI assignments, or named ownership — rather than through individual discipline alone. A rule that depends on people remembering to follow it is not a control.

---

## 2. Scope: Agency Levels (L0–L3)

AWG classifies AI systems by their operational autonomy. The framework applies only to L1 and above.

| Level | Description | Example | AWG Applies? |
|------|------|------|------|
| **L0** | Suggest-only — AI proposes, human types/executes | A coding assistant that drafts code the engineer pastes in | No — use generic AI governance |
| **L1** | Approve-and-execute — AI proposes, human one-clicks to apply | An agent that drafts an email and a human clicks Send | Yes — light governance |
| **L2** | Bounded autonomy — AI executes within explicit guardrails, no per-action review | An agent that auto-files invoices ≤$X under a written policy | Yes — full governance |
| **L3** | Open autonomy — AI executes without per-action human review or fixed guardrails | A long-running agent that triages incidents, escalating only on uncertainty | Yes — maximum governance |

The agency level is one of the inputs to work classification (§3) but is also a standalone scoping signal for whether AWG applies at all.

---

## 3. Component 1 — Work Classification

Every workflow under AWG is classified along three axes plus the agency level. The combination yields a tier (GREEN / YELLOW / RED) that drives gate intensity.

**Axes:**

- **Stakes** — Low / Medium / High — financial, reputational, safety, regulatory, or relational exposure if the action is wrong.
- **Reversibility** — Reversible / Reversible-with-effort / Irreversible — how recoverable the action is and at what cost.
- **Blast Radius** — Single-user / Team / Org / External — how far a wrong action propagates before it can be contained.

Plus the **Agency Level** (L1 / L2 / L3) from §2.

**Tier output (skeleton — to be sharpened):**

- **GREEN** — Low stakes AND reversible AND ≤Team blast radius. Most L1 work lands here.
- **YELLOW** — Medium stakes OR reversible-with-effort OR Org blast radius. Typical L2 work.
- **RED** — High stakes OR irreversible OR External blast radius OR L3 agency. Always at least one of these triggers RED.

**Worked examples:**

- *Agent labels Slack channels for archival.* L2 / Low stakes / Reversible / Team → **GREEN**.
- *Agent makes refund decisions ≤$100.* L2 / Med stakes / Reversible-with-effort / External → **YELLOW**.
- *Agent merges PRs to main outside business hours.* L3 / High stakes / Reversible-with-effort / Team → **RED** (driven by L3).
- *Agent rotates production database credentials.* L2 / High stakes / Reversible-with-effort / Org → **RED** (driven by stakes).

**Open question:** A fourth axis for *data sensitivity* (PII, PHI, secrets, regulated) may belong here rather than being absorbed into Stakes. Flagged for §10.

---

## 4. Component 2 — Phase Model

A map of where gates *can* be placed across an agentic workflow. Not every workflow exercises all five phases — the model is a placement guide, not a checklist.

1. **Plan** — agent declares intent and proposes the action it will take.
2. **Act** — agent executes (in dry-run, sandbox, or production depending on tier).
3. **Verify** — agent or human confirms the action achieved the intended state and nothing else.
4. **Release** — change becomes visible to its blast-radius population.
5. **Monitor** — observe for downstream effects, drift, and reversion triggers.

The phase model exists so that gate primitives (§5) can be assigned to specific transition points rather than being applied as a generic "review step."

---

## 5. Component 3 — Gate Primitives

### 5.0 Design philosophy

AWG defines gates through a small library of **atomic primitives** rather than enumerating every named control adopters might recognize. Composite controls (circuit-breaker, dual-control, canary deploy) are explicitly *compositions* of these primitives, documented as patterns in §5.2 rather than living in the primitive library itself.

Every primitive is characterized by a coordinate in a three-axis space:

- **Lifecycle phase** — *design-time / pre-action / mid-action / post-action / continuous*
- **Authority required** — *automated / single-human / multi-human / out-of-band*
- **Function** — *constrain / verify / attest / observe / halt / reverse*

A primitive that does not fit cleanly in this space is either misnamed or actually composed of smaller primitives. The taxonomy is the framework's quality check on itself.

### 5.1 Primitive library

Ten atomic primitives. Each is described with a uniform schema: coordinates, description, structural requirements, composition partners, tier guidance, and anti-pattern.

---

#### `design-attestation`

| Lifecycle | Authority | Function |
|---|---|---|
| design-time | multi-human (RED) or single-human (lower tiers) | attest |

A formal sign-off that the workflow's *design* — including the chosen gate composition, scope bounds, Process Owner, rollback procedure, idempotency analysis, and TTL parameters — is approved. Verified once at workflow approval. Re-required on material change.

**Structural requirements.** A version-controlled design document; named approver(s) recorded with timestamps; signed off through an audit-traceable channel (PR approval, CAB record, signed change request); the workflow CANNOT be deployed without a current attestation referenced by ID.

**Composes with.** Every other primitive. Design-attestation is the umbrella sign-off that "this composition of primitives is the right one for this work."

**Tier guidance.** GREEN — single approver, lightweight template; YELLOW — single approver + governance sampling audit; RED — multi-approver (Process Owner + governance reviewer) + rehearsal artifact references.

**Anti-pattern.** A Slack message saying "looks good." No version-controlled artifact, no audit trail, no defined approver scope.

---

#### `human-attestation(n, sod)`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | single or multi-human (parameterized by `n`) | attest |

A named human (or `n` humans, with separation of duties when `sod=true`) reviews a specific run and approves before the agent's action commits. Bound to a run, not to the workflow design — that's `design-attestation`'s job.

**Structural requirements.** A handoff schema (§6) producing the artifact under review; approver identity captured per-run in the provenance log; the agent's execution path CANNOT proceed without recorded approval (enforced in code, not policy); when `sod=true`, the proposer and approver(s) must be distinct identities and the platform must reject self-approval.

**Composes with.** `dry-run` (the dry-run output is part of the artifact reviewed); `uncertainty-declaration` (high uncertainty can dynamically raise `n` or require `sod`); `provenance-log` (the approval becomes part of the record).

**Tier guidance.** GREEN — typically not required; YELLOW — `(n=1)`; RED — `(n=2, sod=true)` (this is "dual control").

**Anti-pattern.** An email asking for approval that the agent doesn't wait on, or a "default-approve after 5 minutes" timer. Optional approval is not a gate.

---

#### `dry-run`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | automated | verify |

The agent's proposed action executes against a non-production target (sandbox, staging snapshot, simulator) with logged effects before it executes against the real target. The dry-run output is presented as part of the handoff for review.

**Structural requirements.** A reproducible non-production environment whose semantics mirror production for the affected resources; dry-run effects logged in a form directly comparable to real-run effects; the dry-run is invoked automatically, not by request.

**Composes with.** `human-attestation` (the dry-run output is what the human reviews); `observable-period` (post-action state can be compared against dry-run predictions).

**Tier guidance.** GREEN — optional; YELLOW — required for production mutations; RED — required, and the dry-run output is a mandatory handoff field.

**Anti-pattern.** The agent producing a natural-language description of what it "would" do. That is not a dry-run — a real dry-run produces actual diffs, side-effect logs, and resource impacts in the same format as the production execution.

---

#### `uncertainty-declaration`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | automated | verify |

The agent must produce a calibrated uncertainty signal for each proposed action and include it in the handoff schema. Downstream gate intensity may be dynamically conditioned on the signal (e.g., uncertainty above threshold escalates to higher-tier review).

**Structural requirements.** The uncertainty value is a required field in the handoff schema; null or empty values are rejected; a threshold policy maps uncertainty values to downstream gate adjustments; the policy is version-controlled and reviewed at design-attestation.

**Composes with.** `human-attestation` (uncertainty above threshold raises `n` or triggers `sod`); `drift-monitor` (calibration drift over time triggers reclassification); `provenance-log` (declarations and outcomes are paired for calibration evaluation).

**Tier guidance.** GREEN — not required; YELLOW — declaration required, not necessarily conditioning gates; RED — declaration required AND conditioning gate intensity.

**Anti-pattern.** A static "high/medium/low" label the model outputs without calibration evidence; or a numeric confidence value that has never been validated against outcomes; or a threshold that's set once and never tuned.

**Scope boundary.** Calibration *evaluation* (i.e., does the agent's stated confidence track its actual accuracy?) is treated as out-of-AWG-scope. Adopters must satisfy it via NIST AI RMF or equivalent model-risk processes. AWG requires the *declaration*; the model-risk function ensures the *signal is meaningful*.

---

#### `scope-bound-execution`

| Lifecycle | Authority | Function |
|---|---|---|
| mid-action | automated | constrain |

The agent operates within explicit, machine-enforced bounds — rate limits, dollar limits, allowed-resource lists, geographic or identity scopes. Bounds are encoded in capability tokens, IAM policies, or middleware, not in the agent's prompt.

**Structural requirements.** Bounds are enforced at a layer the agent cannot override by reasoning; bounds are documented in the design doc and verified at design-attestation; bound violations produce immediate halt plus alert, never silent failure or quiet degradation.

**Composes with.** `policy-halt` (bound violation is a halt signal); `provenance-log` (bound checks and near-misses are recorded).

**Tier guidance.** GREEN — typically rate-limited only; YELLOW — domain-scoped (allowed resources, dollar caps); RED — full least-privilege identity + dollar caps + rate limits + geographic scope where applicable.

**Anti-pattern.** "The prompt instructs the agent not to do X." Prompt-level constraints are suggestions, not bounds. A real bound is enforced by infrastructure the agent does not control.

---

#### `provenance-log`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | observe |

An immutable, append-only record of every agent action: input, reasoning trace, output, side-effects, identity context, approvals, uncertainty declaration, and outcome. The log is the substrate for accountability (P1) and the signal source for drift-monitor and policy-halt.

**Structural requirements.** Append-only storage (write-once or cryptographic chain); per-action record keyed by a stable identifier; retention policy aligned with regulatory and accountability requirements; the agent cannot disable, rewrite, or selectively omit from its own log; access controls separate the *write* path (agent + tooling) from the *read* path (governance + audit).

**Composes with.** Every other primitive — provenance-log is the substrate they write to or read from.

**Tier guidance.** GREEN — basic action log with input/output/identity; YELLOW — full reasoning capture; RED — full capture + tamper-evidence + extended retention aligned with regulatory needs.

**Anti-pattern.** Application logs at INFO level that get rotated weekly. Not provenance — provenance is durable, queryable, and integrity-protected.

---

#### `time-to-live`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

The agent's authority to act expires automatically at a defined boundary — wall-clock time, action count, session length, or business event. Renewal requires re-attestation; silent continuation is structurally impossible.

**Structural requirements.** Capability tokens (or equivalent) with hard expiration enforced by the platform; a renewal flow that routes through the appropriate attestation primitive; expiry cannot be bypassed by the agent's reasoning.

**Composes with.** `human-attestation` (renewal requires a fresh approval); `design-attestation` (TTL parameters are set and approved at design time).

**Tier guidance.** GREEN — session-bound or per-task; YELLOW — short TTL measured in hours-to-days; RED — short TTL with mandatory re-attestation, ideally requiring a fresh `human-attestation` rather than automated renewal.

**Anti-pattern.** A never-expiring service account credential. The agent will outlive the assumptions of the workflow that originally authorized it.

---

#### `policy-halt`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

The agent is automatically halted when a named condition fires. Conditions are expressed as declarative policies (error rate exceeds threshold, anomaly score above bound, downstream alert active, manual kill switch triggered) and produce immediate suspension plus notification to the Process Owner.

**Structural requirements.** A declarative, version-controlled policy expression; halt mechanism enforced below the agent (not "the agent decides to stop"); notification routing to the named Process Owner; resume requires a fresh `human-attestation` to lift the halt.

**Composes with.** `provenance-log` (signal source for most halt conditions); `scope-bound-execution` (bound violation as a halt condition); `drift-monitor` (drift as a halt condition).

**Tier guidance.** GREEN — basic kill switch; YELLOW — error-rate and bound-violation halts; RED — comprehensive policy set including anomaly detection, downstream alert correlation, and manual kill.

**Anti-pattern.** A runbook step that says "operator should stop the agent if X." Not a halt — a halt is structurally automatic and routes to a named human, it does not depend on operator vigilance.

---

#### `drift-monitor`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

Compares the agent's behavior — input distributions, output patterns, uncertainty calibration, outcome distributions — to a documented baseline established at deployment. Significant drift halts the agent and forces reclassification (which may move the workflow to a higher tier).

**Structural requirements.** Baseline established at deployment with explicit signals named and recorded; statistical thresholds for drift (not eyeball detection); halt-and-reclassify trigger when drift exceeds threshold; reclassification routes back through `design-attestation` before resumption.

**Composes with.** `provenance-log` (signal source); `policy-halt` (halt mechanism); `uncertainty-declaration` (calibration drift is one tracked signal).

**Tier guidance.** GREEN — usually not required; YELLOW — basic input-distribution drift; RED — full multi-signal drift including uncertainty calibration, outcome distribution, and external-context drift.

**Anti-pattern.** "We'll check periodically." Periodic manual review is not drift detection. Drift detection is continuous, statistically grounded, and triggered automatically.

---

#### `observable-period`

| Lifecycle | Authority | Function |
|---|---|---|
| post-action | automated | reverse |

After the action commits, a defined window passes during which automated checks compare actual outcomes to predicted outcomes (typically from the dry-run). Deviation triggers automated rollback or halt-plus-alert.

**Structural requirements.** Predicted outcome captured pre-action; actual outcome observed within the window; explicit comparison logic that defines "deviation"; rollback procedure that is itself a tested, version-controlled artifact; the window must elapse before downstream dependencies treat the action as final.

**Composes with.** `dry-run` (predicted-outcome source); `policy-halt` (rollback as halt-plus-reverse); `provenance-log`.

**Tier guidance.** GREEN — typically not required; YELLOW — short window with simple invariant check; RED — extended window with full outcome comparison and a tested, rehearsed rollback path.

**Anti-pattern.** "We'll watch the dashboard for an hour." Watching is not observation — observation is structurally automated and the rollback is a tested procedure, not a manual decision.

---

### 5.2 Composition patterns

Named controls that adopters recognize are *compositions* of the primitives above. Documenting them as patterns (not primitives) keeps the library small while preserving recognizability.

**Circuit-breaker** = `provenance-log` (signal source) + `policy-halt` (automated halt action). Often deployed with `drift-monitor` and/or `scope-bound-execution` as additional halt sources.

**Dual control** = `human-attestation(n=2, sod=true)`. Optionally combined with `design-attestation` clauses requiring documented approver independence.

**Canary deploy** = `dry-run` (validate against staging) + `observable-period` (gradual rollout window with automated regression checks) + `policy-halt` (auto-halt on regression).

**Rollback-rehearsed** = `design-attestation` carrying a rehearsal-log artifact ID + (optionally) `observable-period` whose rollback path *is* the rehearsed procedure. The "rehearsed" property is an attribute of the design, not a runtime gate.

**Time-bounded contractor pattern** = `time-to-live` + `human-attestation` (renewal) + `scope-bound-execution`. Used when an agent is treated as a fixed-term contractor with renewable authority.

**Multi-agent supervisor pattern** = per-agent `provenance-log` + supervisor-level `policy-halt` + `human-attestation` for cross-agent decisions. Open design question (§10 #6) on accountability pinning in multi-agent settings.

### 5.3 Anti-patterns

Things that look like gates but do not satisfy P4 (cheap talk does not govern). Adopters reach for these instinctively; the framework's job is to name them so they get caught at design-attestation.

- **"Notification before acting."** Sending a Slack message before executing is not a gate — the agent proceeds without attestation. Use `human-attestation` with structurally enforced wait-for-approval.

- **"Approval requested in chat."** Chat messages aren't durably auditable, approver identity is socially verified rather than cryptographically established, and the agent typically proceeds regardless. Symptom: `human-attestation` lacking structural enforcement.

- **"The prompt says don't do X."** Prompt-level constraints are suggestions, not bounds. Use `scope-bound-execution` enforced at the capability layer.

- **"Confidence threshold without calibration."** Declaring uncertainty is performative unless calibration is independently validated. Either pair `uncertainty-declaration` with model-risk calibration, or do not condition gates on the signal.

- **"Log it for audit."** Application logs are not provenance. Provenance is durable, integrity-protected, and queryable as a primary record. Use `provenance-log` with proper substrate.

- **"We'll watch the dashboard."** Manual observation is not observation. Use `observable-period` with structural comparison and automated rollback.

- **"Stop button on the wiki."** A halt mechanism no one is responsible for monitoring is not a halt. Use `policy-halt` with named alert routing.

---

## 6. Component 4 — Handoff Schemas

The most error-prone gate is one that exists structurally but produces rubber-stamps in practice. Handoff schemas exist to make rubber-stamping structurally hard. They define the *required content* of the artifact passed at every AI→human or human→AI transition.

**Minimum handoff (required for `human-attestation` gates):**

- **Intent** — what the agent is about to do, in plain language, one paragraph.
- **Diff** — the concrete state change (file diff, DB delta, config change, API call payload). Not a description of the diff — the actual diff.
- **Justification** — why this is the right action, traced to specific inputs. Not "because the user asked."
- **Evidence** — what was checked: tests run, dry-run output, validations passed. Each evidence item is an artifact reference.
- **Uncertainty** — the value produced by `uncertainty-declaration`, plus the threshold policy outcome.
- **Reversibility** — explicit rollback procedure, or a flagged "irreversible after step X" marker.
- **Open Questions** — assumptions made and uncertainties unresolved. Empty list is suspicious and should be rejected by reviewers as a matter of policy.

**Higher-tier handoffs add:**

- **Alternatives Considered + Rejection Rationale** — for L3 / RED tier.
- **Impact Assessment** — for External blast radius (affected populations, expected magnitude).
- **Compliance Trace** — for regulated work (which control this satisfies, evidence reference).

**Comparative example:**

*Bad handoff (rubber-stamp bait):* "I'm about to run the cleanup script. Approve?"

*Good handoff:* "Intent: cancel orders flagged in ticket #4521. Diff: UPDATE orders SET status='cancelled' WHERE id IN (...) — 247 rows affected, IDs attached. Justification: each ID matches the ticket's customer-confirmed cancellation list. Evidence: dry-run on staging snapshot produced identical row count; spot-check of 5 random IDs confirmed customer email match. Uncertainty: 0.12 (below 0.25 threshold; no escalation triggered). Reversibility: rollback transaction in `rollback-4521.sql`, tested on staging. Open questions: should downstream notification webhooks fire? I recommend disabling them — please confirm."

The second handoff is reviewable. The first is not. The schema is what produces the difference.

---

## 7. Component 5 — Accountability Principle (the spine)

Restated as four normative requirements — these are not aspirational, they are the framework's structural backbone:

1. **Every L1+ workflow has exactly one named human Process Owner at any given time.** Vacancy is a governance violation.
2. **The Process Owner is accountable for outcomes** regardless of which gates fired, who reviewed, or how autonomous the agent was.
3. **The governance function** (whoever runs AWG in the org — see Layer 2) **advises, audits, and certifies — never replaces — the Process Owner's accountability.**
4. **Process Owner names appear on the workflow design document.** AI agent names do not appear on org charts.

The fourth point is not symbolic. It is the direct structural counter to the Wiles mechanism: if the AI is on the org chart, accountability diffuses; if the human responsible for the AI's work is on the workflow design doc, accountability re-pins.

---

## 8. The AWG Matrix (tier × phase × primitives)

Where the components compose. Each cell specifies the primitives (and parameters) required at that tier × phase intersection. Empty cells are permitted; rows are not — every L1+ tier has at least one gate somewhere.

| Tier | Plan | Act | Verify | Release | Monitor |
|------|------|------|------|------|------|
| **GREEN** | `design-attestation(n=1)` | `scope-bound-execution` (rate-limit) | spot-check via `provenance-log` | — | `provenance-log` |
| **YELLOW** | `design-attestation(n=1)` + governance sampling audit | `dry-run` + `scope-bound-execution` + `uncertainty-declaration` | `human-attestation(n=1)` triggered when uncertainty > threshold | `human-attestation(n=1)` | `provenance-log` + `policy-halt` + `time-to-live` (days) |
| **RED** | `design-attestation(n≥2, sod=true)` + governance review + `rollback-rehearsed` | `dry-run` + `scope-bound-execution` (least-privilege) + `uncertainty-declaration` + `human-attestation(n=2, sod=true)` | `human-attestation(n=2, sod=true)` | `observable-period` + `human-attestation(n=1)` | `provenance-log` + `policy-halt` + `drift-monitor` + `time-to-live` (hours) |

Read each cell as a *minimum* — workflows may add primitives above the tier baseline based on workflow-specific risks. Workflows cannot subtract below the baseline without a documented exception attested through `design-attestation` and routed to governance review.

---

## 9. What AWG Is Not

To keep scope honest:

- **Not a model risk framework.** Use NIST AI RMF for model evaluation, bias testing, and lifecycle.
- **Not a regulatory compliance framework.** Use ISO/IEC 42001 or jurisdiction-specific resources (EU AI Act, etc.) for that.
- **Not a security review process.** Threat modeling, AppSec, and supply-chain controls are separate concerns.
- **Not a calibration framework.** `uncertainty-declaration` requires *that* uncertainty is declared; it does not validate that the declared value is meaningful. Calibration evaluation lives in model risk.
- **Not a checklist.** It is a structural toolkit — workflows compose primitives based on classification, not by ticking boxes.
- **Not for L0 systems.** Suggest-only AI does not produce the responsibility-shift mechanism AWG addresses.
- **Not a substitute for engineering rigor.** Tests, code review, observability, incident response — those are still required and AWG composes with them, not over them.

---

## 10. Open Design Questions

Items flagged during skeleton drafting; intended to be resolved before v1.0.

1. **Data sensitivity as a fourth classification axis** — or absorbed into Stakes? Probably distinct, since regulated data triggers different controls than financially-high-stakes work.
2. **Interaction with existing change management** (ITIL CAB, SOX-style sign-offs) — does AWG replace, supplement, or pass through?
3. **Re-classification cadence** — workflows drift over time as agents are retrained or guardrails relax. What forces a re-classification beyond `drift-monitor` triggering?
4. **Tooling neutrality** — does the framework prescribe formats for handoff schemas (YAML schema? JSON?), or stay tool-agnostic and let adopters implement?
5. ~~**Circuit-breaker classification** — primitive or composed pattern?~~ *Resolved in v0.2: composed pattern (§5.2). Decomposes into `provenance-log` + `policy-halt`.*
6. **Multi-agent workflows** — how does accountability pin when multiple agents interact? Single Process Owner of the whole workflow, or per-agent? Likely single — but supervisor patterns need a worked example.
7. **The `q̂` problem** — partially addressed by `uncertainty-declaration` + `drift-monitor` in v0.2. Open sub-question: should AWG specify *which signals* should raise `q̂` (e.g., novel input distributions, recent incidents) as part of the dynamic gate-intensity policy, or leave the signal selection to adopters?
8. **Calibration boundary** — `uncertainty-declaration` requires the declaration but defers calibration to model risk. Is this delegation sound, or does AWG need to require evidence of calibration evaluation as part of `design-attestation`?

---

## 11. How to Apply (in 6 steps)

1. **Locate** — determine the agency level (§2). If L0, stop; AWG does not apply.
2. **Classify** — score the workflow on stakes / reversibility / blast radius (§3). Resolve the tier.
3. **Map** — identify which phases (§4) the workflow exercises.
4. **Compose** — assign gate primitives (§5) to phase × tier cells per the matrix (§8); customize where needed.
5. **Schema** — define the handoff content (§6) for each gate.
6. **Pin** — name the Process Owner (§7), record on the workflow design doc, register with the governance function.

The Layer 2 document (forthcoming) covers *who* in your organization performs each of these steps and how the work is reviewed and audited.

---

## Appendix A — Glossary

- **Agent** — an AI system that takes action, not merely produces text. (See L1+ in §2.)
- **Atomic primitive** — a gate building block characterized by a single coordinate in the three-axis space (lifecycle / authority / function). Cannot be meaningfully decomposed within AWG.
- **Composition pattern** — a named control composed of multiple primitives (e.g., circuit-breaker, dual-control, canary deploy). Documented for recognizability; not part of the primitive library.
- **Coordinate** — a primitive's position in the three-axis space.
- **Gate** — a structural checkpoint between phases that conditionally permits transition.
- **Governance Function** — the body that maintains AWG in an organization. Form varies by scale (Layer 2).
- **Handoff** — the artifact passed at a gate; its required content is defined by the handoff schema.
- **Process Owner** — the named human accountable for an AWG-governed workflow.
- **Provenance** — the durable, integrity-protected record of an agent's actions; the substrate for accountability.
- **Tier** — the GREEN / YELLOW / RED output of work classification; drives gate intensity.

---

## Appendix B — References

- Wiles, E., Hsu, M., Bedard, J., & Kropp, M. (2026). *Putting AI on the Org Chart: Evidence on Delegation and Oversight.* Boston University / BCG working paper.
- NIST AI Risk Management Framework (AI RMF 1.0).
- ISO/IEC 42001:2023 — Artificial Intelligence Management System.
- Aghion, P., & Tirole, J. (1997). *Formal and Real Authority in Organizations.* Journal of Political Economy.
- Holmström, B. (1979). *Moral Hazard and Observability.* Bell Journal of Economics.

---

*Skeleton ends here. v0.2 changes: §5 fully rewritten with three-axis primitive taxonomy; §5.2 composition patterns and §5.3 anti-patterns added; §8 matrix updated to specify primitive coordinates per tier × phase; §10 question 5 resolved, question 7 partially addressed, question 8 added on calibration scope. Next candidate drills: handoff schemas (§6), tier resolution logic (§3), or the `q̂` dynamic gate-intensity policy (§10 #7).*

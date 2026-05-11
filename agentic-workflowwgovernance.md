# Agentic Workflow Governance (AWG)

*A framework for designing, gating, and pinning accountability in workflows where AI agents take action.*

**Version:** 0.3 — skeleton draft (handoff schemas, tier resolution, phase model, composition patterns drilled; worked example added)
**Layer:** 1 of 2 (org-agnostic framework; operating models live in Layer 2)
**Scope:** Agency level L1 and above (see §2)
**License:** CC BY-SA 4.0

---

## 0. Why this exists

Organizations are increasingly placing AI agents in roles that take action — not just produce text — inside core workflows. Empirical evidence (Wiles, Hsu, Bedard & Kropp, 2026, *Putting AI on the Org Chart*) shows that when AI is framed as an organizational actor with institutional credibility, two things happen simultaneously inside human reviewers:

1. The reviewer's perceived share of responsibility for errors drops (delegation shifts ownership).
2. Unlike with human delegation, no compensating moral-hazard suspicion arises (AI doesn't shirk, so no instinct to monitor it more carefully).

The combined effect is a measurable decline in oversight — in their RCT, ~16% lower review accuracy and a ~44% jump in escalation requests, despite identical underlying work. The mechanism is not anthropomorphism; it is a structural feature of how delegation interacts with the absence of moral hazard.

Existing AI governance frameworks (NIST AI RMF, ISO/IEC 42001, the EU AI Act compliance kits) address model risk, data handling, and high-level lifecycle management. None specifically addresses the *agentic governance gap* this paper identifies. AWG is designed to fill that gap.

---

## 1. Principles

**P1 — Accountability is pinned to a human.** The named Process Owner is accountable for outcomes regardless of how much the AI did or how many gates approved the action. Governance advises, audits, certifies; governance does not absorb responsibility.

**P2 — Gates are structural substitutes for an absent reflex.** With humans, monitoring is partly produced by suspicion of shirking. With agents, that reflex never fires. Procedural gates are not ceremony — they are the engineered replacement for an instinct that does not exist.

**P3 — Autonomy is earned per-tier, not granted globally.** The default state for L1+ workflows is gated. Ungated autonomy is an explicit, classified, documented decision.

**P4 — Cheap talk does not govern.** Every rule must be enforceable structurally — through tooling, code paths, RACI, or named ownership — rather than through individual discipline alone.

---

## 2. Scope: Agency Levels (L0–L3)

| Level | Description | Example | AWG Applies? |
|------|------|------|------|
| **L0** | Suggest-only — AI proposes, human types/executes | A coding assistant that drafts code the engineer pastes in | No — use generic AI governance |
| **L1** | Approve-and-execute — AI proposes, human one-clicks to apply | An agent that drafts an email and a human clicks Send | Yes — light governance |
| **L2** | Bounded autonomy — AI executes within explicit guardrails, no per-action review | An agent that auto-files invoices ≤$X under a written policy | Yes — full governance |
| **L3** | Open autonomy — AI executes without per-action human review or fixed guardrails | A long-running agent that triages incidents, escalating only on uncertainty | Yes — maximum governance |

---

## 3. Component 1 — Work Classification

### 3.0 Axes

Every workflow under AWG is scored along four signals:

- **Agency level** — L1 / L2 / L3 (from §2)
- **Stakes** — Low / Medium / High — financial, reputational, safety, regulatory, or relational exposure if the action is wrong
- **Reversibility** — Reversible / Reversible-with-effort / Irreversible
- **Blast radius** — Single-user / Team / Org / External

### 3.1 Resolution algorithm

Each axis contributes a tier signal; the workflow tier is the maximum across signals. This is deterministic, not subjective.

```
agency_signal:
  L1                       → GREEN
  L2                       → YELLOW
  L3                       → RED

stakes_signal:
  Low                      → GREEN
  Medium                   → YELLOW
  High                     → RED

reversibility_signal:
  Reversible               → GREEN
  Reversible-with-effort   → YELLOW
  Irreversible             → RED

blast_radius_signal:
  Single-user              → GREEN
  Team                     → GREEN
  Org                      → YELLOW
  External                 → RED

tier = max(agency_signal, stakes_signal, reversibility_signal, blast_radius_signal)
```

Any single axis scoring RED produces a RED tier. RED triggers do not compound the tier (it is already maximal), but they DO compound *gate intensity* — see §3.2.

### 3.2 RED-trigger stacking

Multiple RED triggers compound the handoff and gate requirements:

- **External blast radius RED** → expanded `impact_assessment` required in handoff (§6); customer/regulatory notification path documented in design
- **Irreversibility RED** → expanded `reversibility` section; rehearsed rollback OR explicit `point_of_no_return` declaration required
- **High stakes RED** → expanded `compliance_trace` in handoff; governance review required at design-attestation
- **L3 agency RED** → `drift-monitor` required; `uncertainty-declaration` must condition downstream gates

Workflows triggering multiple RED conditions assemble the union of these requirements.

### 3.3 Borderline resolution

When an axis value is genuinely ambiguous (e.g., "is this Medium or High stakes?"), bias toward the higher tier. The default is over-governed, not under-governed. Explicit downgrade requires `design-attestation` review with documented rationale.

This is a P4 enforcement: ambiguity left to operator judgment is cheap talk. Forced upward-bias with a documented downgrade path is structural.

### 3.4 Worked examples

| Workflow | Agency | Stakes | Rev. | Blast | Signals | Tier |
|---|---|---|---|---|---|---|
| Agent labels Slack channels for archival | L2 | Low | Reversible | Team | Y, G, G, G | **YELLOW** |
| Agent makes refund decisions ≤$100 | L2 | Medium | RWE | External | Y, Y, Y, **R** | **RED** |
| Agent merges PRs to main outside business hours | L3 | High | RWE | Team | **R**, **R**, Y, G | **RED** |
| Agent rotates production DB credentials | L2 | High | RWE | Org | Y, **R**, Y, Y | **RED** |
| Agent drafts internal team standups | L2 | Low | Reversible | Team | Y, G, G, G | **YELLOW** |

Note: the Slack-archival example moved from GREEN (v0.2) to YELLOW under the v0.3 algorithm because L2 agency floors the signal at YELLOW. This is a feature, not a bug — bounded-autonomy AI doing anything in production should at minimum have provenance logging and a kill switch.

### 3.5 Data sensitivity — open

Whether data sensitivity (PII, PHI, secrets, regulated) belongs as a fifth axis or remains absorbed into Stakes is flagged for v1.0 resolution (see §10 #1). Proposed v1.0 direction: add as a fifth axis with regulated data → minimum YELLOW, special-category data → minimum RED.

---

## 4. Component 2 — Phase Model

### 4.0 Design philosophy

Phases are not just labels — they are transition boundaries where gate primitives have well-defined places to live. Each phase has explicit entry criteria, expected activity, exit criteria, and common gate placement. Workflow designers map their work to phases and place primitives at phase boundaries.

### 4.1 Phase definitions

**Plan**
- *Entry:* Agent receives task or triggering input; capability context (token, scopes, policy) is loaded.
- *Activity:* Agent decides what action(s) to propose; generates intent, diff, justification, evidence, and uncertainty declaration.
- *Exit:* A complete, schema-valid handoff (§6) is produced and recorded in `provenance-log`.
- *Common gates:* `design-attestation` (verified once per workflow, referenced here); `uncertainty-declaration` (signal generated and threshold policy applied).

**Act**
- *Entry:* Handoff schema is valid AND any pre-action `human-attestation` is recorded.
- *Activity:* Agent executes the proposed action against the target — typically dry-run first, then real, depending on tier.
- *Exit:* Action is committed OR halted; provenance record is complete.
- *Common gates:* `dry-run` (validates against staging before real); `scope-bound-execution` (active throughout); `policy-halt` (active throughout).

**Verify**
- *Entry:* Action committed; predicted-outcome and actual-outcome are both available.
- *Activity:* Compare predicted vs. actual; flag anomalies; record verification outcome in provenance.
- *Exit:* Verification result recorded (pass / fail / needs-review).
- *Common gates:* `human-attestation` triggered dynamically (e.g., by uncertainty signal); automated invariant checks.

**Release**
- *Entry:* Verified action; downstream-visibility criteria met.
- *Activity:* Change becomes visible to its blast-radius population (gradual rollout, full rollout, or contained).
- *Exit:* Change is "released" — downstream consumers may rely on it.
- *Common gates:* `human-attestation` for staged-rollout decisions; `observable-period` for canary windows.

**Monitor**
- *Entry:* Change is released; baseline signals are recording.
- *Activity:* Continuous observation for drift, error, rollback triggers, capability expiry.
- *Exit:* Monitoring window closes OR rollback fires OR escalation triggers OR `time-to-live` expires.
- *Common gates:* `provenance-log` (continuous); `policy-halt`, `drift-monitor`, `time-to-live`.

### 4.2 Phase optionality

Not every workflow exercises every phase. Common patterns:

- **Read-only inquiry workflows** — Plan only (no commit phase). Common for analyst-style agents.
- **Idempotent single-shot writes** — Plan + Act, no separate Verify (Act includes its own verification).
- **Bulk operations** — Plan + Act + Verify + Release + Monitor (the canonical 5-phase model).
- **Long-running agents** — Continuous Monitor with Plan/Act cycles nested inside.

The phase model is a placement guide. Workflows declare which phases they exercise in `design-attestation`.

### 4.3 Phase boundaries are gates

The framework's contention: *the moments between phases are where governance lives.* Within a phase, the agent operates with the authority granted by `design-attestation` and constrained by `scope-bound-execution`. At phase boundaries, transition gates fire, schemas are validated, and (where applicable) human attestation is required.

This is why §5's primitives are classified by lifecycle phase (design-time / pre-action / mid-action / post-action / continuous) — the taxonomy maps directly to the phase model.

---

## 5. Component 3 — Gate Primitives

### 5.0 Design philosophy

AWG defines gates through a small library of **atomic primitives** rather than enumerating every named control adopters might recognize. Composite controls (circuit-breaker, dual-control, canary deploy) are explicitly *compositions* of these primitives, documented as patterns in §5.2.

Every primitive is characterized by a coordinate in a three-axis space:

- **Lifecycle phase** — *design-time / pre-action / mid-action / post-action / continuous*
- **Authority required** — *automated / single-human / multi-human / out-of-band*
- **Function** — *constrain / verify / attest / observe / halt / reverse*

A primitive that does not fit cleanly in this space is either misnamed or actually composed of smaller primitives. The taxonomy is the framework's quality check on itself.

### 5.1 Primitive library

Ten atomic primitives. Uniform schema: coordinates, description, structural requirements, composition partners, tier guidance, anti-pattern.

---

#### `design-attestation`

| Lifecycle | Authority | Function |
|---|---|---|
| design-time | multi-human (RED) or single-human (lower) | attest |

A formal sign-off that the workflow's design — gate composition, scope bounds, Process Owner, rollback procedure, idempotency analysis, TTL parameters — is approved. Verified once at workflow approval. Re-required on material change.

**Structural requirements.** Version-controlled design document; named approver(s) with timestamps; audit-traceable channel (PR approval, CAB record, signed change request); the workflow CANNOT deploy without a current attestation referenced by ID.

**Composes with.** Every other primitive. Design-attestation is the umbrella "this composition is right for this work."

**Tier guidance.** GREEN — single approver, lightweight template; YELLOW — single approver + governance sampling audit; RED — multi-approver (Process Owner + governance reviewer) + rehearsal artifact references.

**Anti-pattern.** A Slack message saying "looks good." No version-controlled artifact, no audit trail, no defined approver scope.

---

#### `human-attestation(n, sod)`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | single or multi-human (parameterized by `n`) | attest |

A named human (or `n` humans, with separation of duties when `sod=true`) reviews a specific run and approves before the agent's action commits. Bound to a run, not to the workflow design — that's `design-attestation`'s job.

**Structural requirements.** A handoff schema (§6) producing the artifact under review; approver identity captured per-run in the provenance log; the agent's execution path CANNOT proceed without recorded approval (enforced in code, not policy); when `sod=true`, the proposer and approver(s) must be distinct identities and the platform must reject self-approval.

**Composes with.** `dry-run` (output is part of artifact reviewed); `uncertainty-declaration` (high uncertainty can dynamically raise `n` or require `sod`); `provenance-log` (approval becomes part of the record).

**Tier guidance.** GREEN — typically not required; YELLOW — `(n=1)`; RED — `(n=2, sod=true)`.

**Anti-pattern.** An email asking for approval that the agent doesn't wait on; or "default-approve after 5 minutes" timer. Optional approval is not a gate.

---

#### `dry-run`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | automated | verify |

The agent's proposed action executes against a non-production target with logged effects before real execution. Dry-run output is presented as part of the handoff.

**Structural requirements.** Reproducible non-production environment mirroring production semantics for affected resources; dry-run effects logged in a form directly comparable to real-run effects; dry-run is invoked automatically, not by request.

**Composes with.** `human-attestation` (output is what the human reviews); `observable-period` (post-action state can be compared against dry-run predictions).

**Tier guidance.** GREEN — optional; YELLOW — required for production mutations; RED — required, output is a mandatory handoff field.

**Anti-pattern.** Agent producing a natural-language description of what it "would" do. That is not a dry-run — a real dry-run produces actual diffs and side-effect logs in the same format as production execution.

---

#### `uncertainty-declaration`

| Lifecycle | Authority | Function |
|---|---|---|
| pre-action | automated | verify |

The agent must produce a calibrated uncertainty signal for each proposed action and include it in the handoff schema. Downstream gate intensity may be dynamically conditioned on the signal.

**Structural requirements.** Uncertainty value is a required schema field; null/empty rejected; a threshold policy maps values to downstream gate adjustments; policy is version-controlled and reviewed at design-attestation.

**Composes with.** `human-attestation` (high uncertainty raises `n` or triggers `sod`); `drift-monitor` (calibration drift triggers reclassification); `provenance-log` (declarations + outcomes paired for calibration evaluation).

**Tier guidance.** GREEN — not required; YELLOW — declaration required; RED — declaration required AND conditioning gate intensity.

**Anti-pattern.** Static "high/medium/low" label without calibration evidence; numeric confidence never validated against outcomes; threshold set once and never tuned.

**Scope boundary.** Calibration *evaluation* is out-of-AWG-scope — adopters satisfy via NIST AI RMF or equivalent model-risk processes. AWG requires the *declaration*; model risk ensures the *signal is meaningful*.

---

#### `scope-bound-execution`

| Lifecycle | Authority | Function |
|---|---|---|
| mid-action | automated | constrain |

The agent operates within explicit, machine-enforced bounds — rate limits, dollar limits, allowed-resource lists, geographic or identity scopes. Bounds are encoded in capability tokens, IAM policies, or middleware, not in the agent's prompt.

**Structural requirements.** Bounds enforced below the agent (not overridable by reasoning); bounds documented in design doc and verified at design-attestation; violations produce immediate halt + alert, never silent failure.

**Composes with.** `policy-halt` (bound violation is a halt signal); `provenance-log` (bound checks and near-misses recorded).

**Tier guidance.** GREEN — typically rate-limited only; YELLOW — domain-scoped (allowed resources, dollar caps); RED — full least-privilege identity + dollar caps + rate limits + geographic scope.

**Anti-pattern.** "The prompt instructs the agent not to do X." Prompt-level constraints are suggestions, not bounds. Real bounds are enforced by infrastructure the agent does not control.

---

#### `provenance-log`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | observe |

An immutable, append-only record of every agent action: input, reasoning trace, output, side-effects, identity context, approvals, uncertainty declaration, outcome. The log is the substrate for accountability (P1) and the signal source for `drift-monitor` and `policy-halt`.

**Structural requirements.** Append-only storage (write-once or cryptographic chain); per-action record keyed by stable identifier; retention policy aligned with regulatory/accountability needs; agent cannot disable, rewrite, or selectively omit; access controls separate write (agent + tooling) from read (governance + audit).

**Composes with.** Every other primitive — provenance-log is the substrate they write to or read from.

**Tier guidance.** GREEN — basic action log with input/output/identity; YELLOW — full reasoning capture; RED — full capture + tamper-evidence + extended retention.

**Anti-pattern.** Application logs at INFO that get rotated weekly. Not provenance — provenance is durable, queryable, and integrity-protected.

---

#### `time-to-live`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

The agent's authority to act expires automatically at a defined boundary — wall-clock time, action count, session length, or business event. Renewal requires re-attestation.

**Structural requirements.** Capability tokens with hard expiration enforced by platform; renewal flow routes through appropriate attestation primitive; expiry cannot be bypassed by agent reasoning.

**Composes with.** `human-attestation` (renewal requires fresh approval); `design-attestation` (TTL parameters set/approved at design time).

**Tier guidance.** GREEN — session-bound or per-task; YELLOW — TTL in hours-to-days; RED — short TTL with mandatory `human-attestation` for renewal rather than automated.

**Anti-pattern.** A never-expiring service account credential. The agent will outlive the assumptions of the workflow that authorized it.

---

#### `policy-halt`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

The agent is automatically halted when a named condition fires. Conditions are declarative policies (error rate threshold, anomaly bound, downstream alert active, manual kill switch). Produces immediate suspension plus Process Owner notification.

**Structural requirements.** Declarative, version-controlled policy expression; halt enforced below the agent; named notification routing to Process Owner; resume requires fresh `human-attestation`.

**Composes with.** `provenance-log` (signal source); `scope-bound-execution` (bound violation as halt condition); `drift-monitor` (drift as halt condition).

**Tier guidance.** GREEN — basic kill switch; YELLOW — error-rate and bound-violation halts; RED — comprehensive policy including anomaly detection, downstream alert correlation, manual kill.

**Anti-pattern.** A runbook step saying "operator should stop the agent if X." Not a halt — a halt is structurally automatic and routes to a named human; it does not depend on operator vigilance.

---

#### `drift-monitor`

| Lifecycle | Authority | Function |
|---|---|---|
| continuous | automated | halt |

Compares agent behavior — input distributions, output patterns, uncertainty calibration, outcome distributions — to a baseline established at deployment. Significant drift halts the agent and forces reclassification.

**Structural requirements.** Baseline established at deployment with explicit signals named and recorded; statistical thresholds for drift (not eyeball detection); halt-and-reclassify trigger on threshold breach; reclassification routes back through `design-attestation` before resumption.

**Composes with.** `provenance-log` (signal source); `policy-halt` (halt mechanism); `uncertainty-declaration` (calibration drift is one tracked signal).

**Tier guidance.** GREEN — usually not required; YELLOW — basic input-distribution drift; RED — full multi-signal drift including calibration, outcome distribution, external-context drift.

**Anti-pattern.** "We'll check periodically." Periodic manual review is not drift detection. Drift detection is continuous, statistically grounded, triggered automatically.

---

#### `observable-period`

| Lifecycle | Authority | Function |
|---|---|---|
| post-action | automated | reverse |

After action commits, a defined window passes during which automated checks compare actual to predicted outcomes. Deviation triggers automated rollback or halt-plus-alert.

**Structural requirements.** Predicted outcome captured pre-action; actual outcome observed within window; explicit comparison logic; rollback procedure as tested, version-controlled artifact; window must elapse before downstream dependencies treat action as final.

**Composes with.** `dry-run` (predicted-outcome source); `policy-halt` (rollback as halt-plus-reverse); `provenance-log`.

**Tier guidance.** GREEN — typically not required; YELLOW — short window with simple invariant check; RED — extended window with full outcome comparison and rehearsed rollback path.

**Anti-pattern.** "We'll watch the dashboard for an hour." Watching is not observation — observation is structurally automated, rollback is a tested procedure.

---

### 5.2 Composition patterns

Named controls adopters recognize are *compositions* of the primitives above. Each is specified with the same uniform schema as primitives.

---

#### Circuit-breaker

**Composes.** `provenance-log` (signal source) + `policy-halt` (halt action). Optionally + `drift-monitor` and/or `scope-bound-execution` as additional halt sources.

**When to use.** Any L2+ workflow with continuous operation. Required for RED tier monitoring phase.

**Structural requirements.** Named SLI definitions with thresholds; alert routing to named Process Owner; resume requires fresh `human-attestation`.

**Anti-pattern.** "If the dashboard turns red, page someone." That is an alert pattern, not a circuit-breaker. A circuit-breaker actually halts the agent's execution path automatically.

---

#### Dual control

**Composes.** `human-attestation(n=2, sod=true)`. Often + `design-attestation` clauses requiring documented approver independence.

**When to use.** RED tier; irreversible actions; sensitive data writes; regulated environments.

**Structural requirements.** Approver identities are platform-distinct; the proposer cannot be either approver; both approvals timestamped, immutable, and recorded in `provenance-log`.

**Anti-pattern.** Two people on a Zoom call clicking the same approve button. Not separation of duties — one approver in two seats.

---

#### Canary deploy

**Composes.** `dry-run` (staging validation) + `observable-period` (graduated rollout with automated regression checks) + `policy-halt` (auto-halt on regression).

**When to use.** Changes affecting External blast radius; high-traffic systems; deployments with uncertain regression risk.

**Structural requirements.** Percentage-based rollout mechanism; automated regression checks against pre-defined SLIs; rollback automation tied to halt trigger.

**Anti-pattern.** "Deploy and watch for an hour." Manual observation is not canary deployment.

---

#### Rollback-rehearsed

**Composes.** `design-attestation` carrying a rehearsal-log artifact ID + optionally `observable-period` whose rollback path *is* the rehearsed procedure.

**When to use.** RED tier; irreversible-with-effort actions; any production data mutation; any change to durable shared state.

**Structural requirements.** Rollback procedure executed end-to-end at least once before workflow is design-attested; rehearsal artifact preserved and referenced; re-rehearsal triggered on rollback procedure change.

**Anti-pattern.** "We have a rollback plan." A plan is not a rehearsal. Rehearsal is execution evidence with an artifact.

---

#### Time-bounded contractor pattern

**Composes.** `time-to-live` (auto-expiry) + `human-attestation` (renewal gate) + `scope-bound-execution` (least-privilege during the period).

**When to use.** Agent-as-contractor models; fixed-term automated processes; agents handling temporary projects with defined end conditions.

**Structural requirements.** TTL is platform-enforced not policy-enforced; renewal includes uncertainty review and outcome review since last renewal; on expiry without renewal, all capabilities revoke automatically.

**Anti-pattern.** A TTL the agent can extend itself. Self-renewal defeats the time bound.

---

#### Multi-agent supervisor

**Composes.** Per-agent `provenance-log` + supervisor-level `policy-halt` + supervisor-level `human-attestation` for cross-agent decisions.

**When to use.** Workflows where multiple agents act in concert; pipelines with intermediate agent handoffs.

**Structural requirements.** Single Process Owner for the entire workflow (P1 holds at the workflow level, not per-agent); each agent identity distinct in provenance logs; cross-agent handoffs use the same handoff schema (§6) as agent→human handoffs.

**Anti-pattern.** An "orchestrator agent" with implicit authority over other agents. The orchestrator is itself an agent and lives under the same primitives — it does not absorb accountability.

**Open.** Accountability split when agent A produces input that agent B acts on — see §10 #6.

---

### 5.3 Anti-patterns

Things that look like gates but fail P4 (cheap talk does not govern). Adopters reach for these instinctively; the framework's job is to catch them at `design-attestation`.

- **"Notification before acting."** Sending a Slack message before executing is not a gate — agent proceeds without attestation. Use `human-attestation` with structural wait-for-approval.
- **"Approval requested in chat."** Chat messages aren't durably auditable, approver identity is socially-verified, agent typically proceeds regardless. Symptom: `human-attestation` lacking structural enforcement.
- **"The prompt says don't do X."** Prompt-level constraints are suggestions. Use `scope-bound-execution` at the capability layer.
- **"Confidence threshold without calibration."** Performative unless calibration is independently validated. Pair `uncertainty-declaration` with model-risk calibration, or don't condition gates on the signal.
- **"Log it for audit."** Application logs are not provenance. Use `provenance-log` with proper substrate.
- **"We'll watch the dashboard."** Use `observable-period` with structural comparison + automated rollback.
- **"Stop button on the wiki."** A halt no one monitors is not a halt. Use `policy-halt` with named routing.

---

## 6. Component 4 — Handoff Schemas

### 6.0 Design philosophy

The handoff schema is the structural mechanism that defeats rubber-stamping. A `human-attestation` gate without a schema is a reviewer staring at whatever the agent decided to show them. A `human-attestation` gate with a validated schema is a reviewer staring at *specific required content*, and the gate machinery refuses to proceed if the content is missing or malformed.

Schemas operationalize three of the four principles:
- **P1** — Process Owner field and approval audit trail are schema-required
- **P2** — schema content is *what gets reviewed*, replacing the absent moral-hazard signal with required structured information
- **P4** — schema validation is machine-enforced at the gate, not "the reviewer should check"

### 6.1 Schema-as-contract

Every handoff schema is a contract between three parties:

- **The agent** — must produce content matching the schema or be structurally blocked from proceeding.
- **The reviewer** — has a uniform shape to react to; cannot dismiss the handoff as "not enough info" because the schema enumerates required content.
- **The provenance log** — receives the schema instance as a structured record, queryable post-hoc for audit.

Schema validation is the *first* gate, before human review. A malformed schema is rejected automatically and the agent's action is halted. Only well-formed schemas reach human attestation. This is the structural enforcement that defeats cheap-talk gating.

### 6.2 Base schema (required for any `human-attestation` gate)

| Field | Type | Required | Validation |
|---|---|---|---|
| `workflow_id` | string | yes | references a current `design-attestation` |
| `action_id` | string | yes | unique per run |
| `agent_identity` | object | yes | matches a registered agent |
| `process_owner` | identity | yes | matches the workflow's named owner |
| `intent` | object | yes | non-empty summary + description + affected_resources list |
| `proposed_action` | object | yes | typed operation + payload + declared side_effects |
| `diff` | object | yes | machine-readable state delta — not prose |
| `justification` | object | yes | primary_inputs + reasoning_trace + decision_basis |
| `evidence` | list | yes | ≥1 item with result=passed |
| `uncertainty` | object | yes | value ∈ [0,1] + signal_type + threshold_policy_outcome |
| `reversibility` | object | yes | is_reversible + (rollback_ref OR point_of_no_return) |
| `open_questions` | list | yes | empty list requires explicit "none" sentinel |
| `provenance_ref` | string | yes | references the provenance log entry |

Field semantics worth calling out:

- **`diff` must be machine-readable.** Prose descriptions of state changes are rejected at validation time. The diff is the action-equivalent payload, not an English summary.
- **`evidence` must include at least one passed item.** If no evidence passed, the agent should not be proposing the action.
- **`open_questions` empty list requires explicit sentinel.** Agents may attest "no open questions" but must do so explicitly. The field cannot be omitted.
- **`uncertainty.threshold_policy_outcome`** is the result of applying the `uncertainty-declaration` threshold policy: typically `below_threshold_no_escalation` / `above_threshold_escalate` / `above_hard_limit_halt`. This conditions downstream gate intensity.

### 6.3 Tier-graded extensions

**YELLOW adds (required):**

| Field | Type | Validation |
|---|---|---|
| `alternatives_considered` | list | present; may be empty with explicit "none considered" sentinel |
| `impact_assessment` | object | affected_populations + magnitude_estimate |

**RED adds (required, beyond YELLOW):**

| Field | Type | Validation |
|---|---|---|
| `alternatives_considered` | list | ≥1 item with explicit rejection rationale |
| `compliance_trace` | list | each: control_id + satisfied_by + evidence_ref |
| `rehearsal_ref` | string | references the design-attestation's rehearsal log |
| `dual_attestation` | object | both approver identities + timestamps + separation_of_duties_assertion |

### 6.4 Schema-to-primitive mapping

Which primitives produce, validate, or consume schema content:

| Field | Produced by | Validated by | Consumed by |
|---|---|---|---|
| `workflow_id` | agent | `design-attestation` registry | `human-attestation` |
| `agent_identity` | platform | identity provider | `human-attestation`, `provenance-log` |
| `process_owner` | `design-attestation` | identity provider | `human-attestation`, `policy-halt` routing |
| `diff` | agent + `dry-run` | schema validator + `scope-bound-execution` (verify bounds respected) | reviewer at `human-attestation` |
| `evidence` | `dry-run`, `provenance-log` queries | schema validator | reviewer |
| `uncertainty` | `uncertainty-declaration` | schema validator (range + presence) | gate-intensity policy |
| `reversibility` | agent + `design-attestation` (rollback procedure) | `design-attestation` (procedure exists and tested) | reviewer + `observable-period` |
| `provenance_ref` | `provenance-log` | provenance log | post-hoc audit |
| `rehearsal_ref` (RED) | `design-attestation` | rehearsal log registry | reviewer + auditor |
| `dual_attestation` (RED) | `human-attestation(n=2, sod=true)` | attestation platform | provenance log |

### 6.5 Anti-patterns

- **Free-form prose handoffs.** Not validatable, not queryable, vulnerable to selective emphasis.
- **Schemas validated by humans only.** Validation must be machine-enforced first. A malformed schema should never reach a human reviewer — the gate machinery rejects it before queuing for review.
- **Reviewer-judgment fields.** Vague fields like "is this safe?" with no objective criterion. Abdication, not gating. Subjective fields belong in commentary, not in required schema.
- **Partial schemas per tier.** Use a single schema with optional/required fields graduated by tier. Reduces drift, simplifies validators.
- **Agent-set approval status.** Approval status is set by the attestation system, never by the agent. An agent marking its own work approved is by definition ungoverned.
- **Mutable schemas.** Schema fields, once submitted, are immutable. Corrections require a new schema instance and typically a new attestation cycle.

### 6.6 Worked schema (YAML, YELLOW-tier example)

```yaml
workflow_id: WF-2026-04-orders-cancel
action_id: a-7c2f9d1e-2026-05-08T14:30:00Z
agent_identity:
  agent_name: OrderOps
  version: 3.2.1
  model: claude-sonnet-4-6
  capability_token_id: tok-OrderOps-prod-2026-05-08
process_owner: head-of-customer-ops@example.com
intent:
  summary: Cancel orders from ticket #4521 customer cancellation list
  full_description: |
    Customer service ticket #4521 contains a list of 247 customer-confirmed
    order cancellations. This action sets status='cancelled' on each.
  affected_resources:
    - orders table (rows in id_list)
proposed_action:
  operation_type: db_update
  payload:
    table: orders
    set: {status: cancelled, cancelled_at: NOW()}
    where_clause: id IN (<id_list>)
  side_effects:
    - downstream cancellation webhook (disabled in this run pending confirmation)
diff:
  state_before_ref: snapshot-pre-a7c2f9d1e
  state_after_ref: snapshot-post-a7c2f9d1e
  affected_row_count: 247
justification:
  primary_inputs:
    - ticket #4521 attachment v3
    - customer email confirmations 2026-05-01 to 2026-05-07
  reasoning_trace: |
    Each id in the ticket attachment was matched to a customer email
    confirmation within the past 7 days. No id mismatched.
  decision_basis: input_driven
evidence:
  - evidence_type: dry_run
    artifact_ref: dryrun-a7c2f9d1e-staging
    result: passed
  - evidence_type: spot_check
    artifact_ref: spotcheck-5-of-247
    result: passed
  - evidence_type: schema_validation
    artifact_ref: where-clause-lint
    result: passed
uncertainty:
  value: 0.12
  signal_type: predicted_outcome_match
  threshold_policy_outcome: below_threshold_no_escalation
reversibility:
  is_reversible: true
  rollback_procedure_ref: rollback-4521.sql
  point_of_no_return: null
open_questions:
  - question: Should downstream cancellation webhook fire?
    assumed_answer: No, disabled in this run
    blocking: true
alternatives_considered:
  - alternative: partial cancellation per order
    rejected_because: customer requested full cancellation on all 247
impact_assessment:
  affected_populations:
    - 247 customers in the cancellation list
  magnitude_estimate:
    financial: $0 (refund processed via separate workflow)
    reputational: low (this fulfills customer request)
provenance_ref: pl-a7c2f9d1e
```

This handoff is reviewable. A free-form "I'm about to run the cleanup script. Approve?" is not. The schema is what produces the difference.

---

## 7. Component 5 — Accountability Principle (the spine)

Restated as four normative requirements:

1. **Every L1+ workflow has exactly one named human Process Owner at any given time.** Vacancy is a governance violation.
2. **The Process Owner is accountable for outcomes** regardless of which gates fired, who reviewed, or how autonomous the agent was.
3. **The governance function advises, audits, and certifies — never replaces — the Process Owner's accountability.**
4. **Process Owner names appear on the workflow design document.** AI agent names do not appear on org charts.

Point 4 is not symbolic. It is the direct structural counter to the Wiles mechanism: if the AI is on the org chart, accountability diffuses; if the human responsible for the AI's work is on the workflow design doc, accountability re-pins.

---

## 8. The AWG Matrix (tier × phase × primitives)

Each cell specifies the primitives (and parameters) required at that tier × phase intersection. Empty cells are permitted; rows are not — every L1+ tier has at least one gate somewhere.

| Tier | Plan | Act | Verify | Release | Monitor |
|------|------|------|------|------|------|
| **GREEN** | `design-attestation(n=1)` | `scope-bound-execution` (rate-limit) | spot-check via `provenance-log` | — | `provenance-log` |
| **YELLOW** | `design-attestation(n=1)` + governance sampling audit | `dry-run` + `scope-bound-execution` + `uncertainty-declaration` | `human-attestation(n=1)` triggered when uncertainty > threshold | `human-attestation(n=1)` | `provenance-log` + `policy-halt` + `time-to-live` (days) |
| **RED** | `design-attestation(n=2, sod=true)` + governance review + rollback-rehearsed pattern | `dry-run` + `scope-bound-execution` (least-privilege) + `uncertainty-declaration` + `human-attestation(n=2, sod=true)` | `human-attestation(n=2, sod=true)` | canary-deploy pattern (`observable-period` + `human-attestation(n=1)`) | circuit-breaker pattern + `drift-monitor` + `time-to-live` (hours) |

Each cell is a *minimum* — workflows may add primitives above the tier baseline based on workflow-specific risks. Subtraction below the baseline requires documented exception attested through `design-attestation` and routed to governance review.

---

## 9. What AWG Is Not

- **Not a model risk framework.** Use NIST AI RMF for model evaluation, bias testing, lifecycle.
- **Not a regulatory compliance framework.** Use ISO/IEC 42001 or jurisdiction-specific resources.
- **Not a security review process.** Threat modeling, AppSec, supply-chain — separate concerns.
- **Not a calibration framework.** `uncertainty-declaration` requires that uncertainty is declared; it does not validate the declared value is meaningful. Calibration lives in model risk.
- **Not a checklist.** It is a structural toolkit — workflows compose primitives based on classification.
- **Not for L0 systems.** Suggest-only AI does not produce the responsibility-shift mechanism AWG addresses.
- **Not a substitute for engineering rigor.** Tests, code review, observability, incident response — still required; AWG composes with them, not over them.

---

## 10. Open Design Questions

Resolved or partially resolved in v0.3 marked accordingly.

1. **Data sensitivity as a fifth classification axis** — or absorbed into Stakes? Proposed v1.0 direction: distinct fifth axis (regulated → ≥YELLOW; special-category → ≥RED). Not yet ratified.
2. **Interaction with existing change management** (ITIL CAB, SOX) — replace, supplement, or pass through? Likely pass-through (AWG `design-attestation` and CAB are both required; they don't substitute for each other), but worked example needed.
3. **Re-classification cadence** — `drift-monitor` triggers reclassification on signal drift, but there's no time-based cadence for routine re-review. Proposal: annual re-attestation for all YELLOW+, semi-annual for RED.
4. **Tooling neutrality** — does the framework prescribe formats for handoff schemas (JSON Schema? Protobuf?), or stay tool-agnostic? Current bias: stay neutral; provide a reference YAML shape (see §6.6) without mandating a specific validator.
5. ~~**Circuit-breaker classification** — primitive or composed pattern?~~ *Resolved v0.2: composed pattern (§5.2).*
6. **Multi-agent workflows** — accountability when agent A produces input agent B acts on. Current v0.3 position: single Process Owner of the workflow eats the loss; supervisor pattern (§5.2) is the structural shape; per-agent attribution lives in `provenance-log` for post-hoc analysis. Open whether this is enough.
7. ~~**The `q̂` problem.**~~ *Partially resolved v0.2: `uncertainty-declaration` + `drift-monitor` together carry the signal. Remaining sub-question:* should AWG enumerate canonical signals that should raise `q̂` (novel input distributions, recent incidents, time since last calibration), or leave signal selection to adopters?
8. **Calibration boundary.** `uncertainty-declaration` requires the declaration but defers calibration to model risk. Sound delegation, or does `design-attestation` need to require calibration-evaluation evidence as a prerequisite?
9. **Schema format prescriptiveness.** §6.6 shows a YAML shape but does not mandate it. Should v1.0 publish an official AWG Schema (JSON Schema artifact) for interoperability across adopters? Tradeoff: prescription drives ecosystem; flexibility drives adoption breadth.

---

## 11. How to Apply (in 6 steps)

1. **Locate** — determine agency level (§2). If L0, stop; AWG does not apply.
2. **Classify** — score the workflow on stakes / reversibility / blast radius (§3); apply the resolution algorithm (§3.1); note any RED-trigger stacking (§3.2).
3. **Map** — identify which phases (§4) the workflow exercises.
4. **Compose** — assign gate primitives (§5) per the matrix (§8); reach for composition patterns (§5.2) where they apply; customize for workflow-specific risks.
5. **Schema** — define the handoff content (§6) for each gate; validate schemas are machine-enforceable before deployment.
6. **Pin** — name the Process Owner (§7); record on the workflow design doc; register with the governance function.

The Layer 2 document covers *who* in your organization performs each step and how the work is reviewed and audited.

---

## Appendix A — Glossary

- **Agent** — an AI system that takes action, not merely produces text. (See L1+ in §2.)
- **Atomic primitive** — a gate building block characterized by a single coordinate in the three-axis space. Cannot be meaningfully decomposed within AWG.
- **Composition pattern** — a named control composed of multiple primitives. Recognized in §5.2; not part of the primitive library.
- **Coordinate** — a primitive's position in the three-axis space (lifecycle / authority / function).
- **Gate** — a structural checkpoint between phases that conditionally permits transition.
- **Governance function** — the body that maintains AWG in an organization. Form varies by scale (Layer 2).
- **Handoff** — the artifact passed at a gate; required content defined by the handoff schema.
- **Process Owner** — the named human accountable for an AWG-governed workflow.
- **Provenance** — the durable, integrity-protected record of an agent's actions; substrate for accountability.
- **RED-trigger stacking** — additive requirements when multiple axes contribute a RED signal (§3.2).
- **Schema validator** — the machine-enforced gate that rejects malformed handoffs before they reach human review.
- **Tier** — the GREEN / YELLOW / RED output of work classification; drives gate intensity.

---

## Appendix B — References

- Wiles, E., Hsu, M., Bedard, J., & Kropp, M. (2026). *Putting AI on the Org Chart: Evidence on Delegation and Oversight.* Boston University / BCG working paper.
- NIST AI Risk Management Framework (AI RMF 1.0).
- ISO/IEC 42001:2023 — Artificial Intelligence Management System.
- Aghion, P., & Tirole, J. (1997). *Formal and Real Authority in Organizations.* Journal of Political Economy.
- Holmström, B. (1979). *Moral Hazard and Observability.* Bell Journal of Economics.

---

## Appendix C — Worked Example: Agentic Refund Processing

A complete walkthrough applying AWG to a real workflow: an AI agent ("RefundOps") that processes customer refund requests — reading tickets, validating eligibility against policy, and issuing refunds via the payment processor without per-action human review.

### Step 1: Locate (§2 scoping)

RefundOps initiates real money transfers within policy bounds without per-action human review. Agency level = **L2 (bounded autonomy)**.
→ AWG applies.

### Step 2: Classify (§3)

- **Stakes:** Medium (per-refund financial exposure capped by policy; reputation exposure exists)
- **Reversibility:** Reversible-with-effort (refunds can be reversed via a separate payment-processor flow over 3–5 business days)
- **Blast radius:** External (customers receive money)
- **Agency:** L2

Applying §3.1:

```
agency_signal       = YELLOW   (L2)
stakes_signal       = YELLOW   (Medium)
reversibility_signal = YELLOW  (RWE)
blast_radius_signal = RED      (External)

tier = max(YELLOW, YELLOW, YELLOW, RED) = RED
```

RED triggered by External blast radius → expanded `impact_assessment` required (§3.2).

### Step 3: Map (§4)

Workflow exercises all five phases:
- **Plan** — read ticket, decide eligibility, draft refund proposal
- **Act** — call payment processor (dry-run on sandbox, then production)
- **Verify** — confirm refund posted; reconcile with processor response
- **Release** — notify customer
- **Monitor** — watch for chargebacks, disputes, and processor errors

### Step 4: Compose (§5, §8)

Per the RED row of the matrix:

| Phase | Primitives / Patterns |
|---|---|
| Plan | `design-attestation(n=2, sod=true)` + governance review + rollback-rehearsed pattern |
| Act | `dry-run` (sandbox processor) + `scope-bound-execution` (refund ≤ policy limit; ≤N refunds/hour; only tickets in approved queue) + `uncertainty-declaration` + `human-attestation(n=2, sod=true)` triggered for refunds > $500 |
| Verify | `human-attestation(n=2, sod=true)` for refunds > $500; automated invariant check for refunds ≤ $500 (refund_id present in processor response) |
| Release | canary pattern: `observable-period` (24-hour watch for processor errors) + `human-attestation(n=1)` for customer-notification batch |
| Monitor | circuit-breaker pattern (`provenance-log` + `policy-halt` on >2% error rate in 1 hour) + `drift-monitor` (refund-eligibility decision drift vs historical; halt + reclassify on >3σ) + `time-to-live` (capability token expires every 24 hours; renewal requires fresh `human-attestation`) |

The dynamic gate intensity — refunds above $500 escalate to dual control — is parameterized in `design-attestation`, not in the agent's prompt. The threshold itself is a schema-validated field.

### Step 5: Schema (§6)

A RED-tier RefundOps handoff includes the base schema (§6.2), YELLOW extensions (§6.3 — `alternatives_considered`, `impact_assessment`), and RED extensions (`compliance_trace`, `rehearsal_ref`, `dual_attestation` when above threshold). Sketched:

```yaml
workflow_id: WF-refund-ops-2026
action_id: a-refund-2026-05-08-00723
agent_identity: {agent_name: RefundOps, version: 2.4, ...}
process_owner: head-of-customer-ops@example.com
intent:
  summary: Issue $87.50 refund for ticket #4521
  affected_resources: [customer-account-id-1234567, payment-processor-tx]
proposed_action:
  operation_type: payment_processor_refund
  payload: {tx_id: ..., amount_cents: 8750, currency: USD}
  side_effects: [customer_email_notification (deferred to Release phase)]
diff:
  state_before_ref: ...
  state_after_ref: ...
justification:
  primary_inputs: [ticket #4521, policy doc REF-2026-Q2 §3.2]
  decision_basis: policy_eligibility_match
evidence:
  - {evidence_type: policy_eligibility_check, result: passed}
  - {evidence_type: fraud_check, result: passed}
  - {evidence_type: dry_run_against_sandbox, result: passed}
uncertainty:
  value: 0.08
  signal_type: eligibility_model_confidence
  threshold_policy_outcome: below_threshold_no_escalation
reversibility:
  is_reversible: true
  rollback_procedure_ref: reverse-refund.sql + processor-reversal-flow
open_questions: []  # explicit empty
alternatives_considered:
  - {alternative: deny, rejected_because: policy criteria fully met}
  - {alternative: partial, rejected_because: customer requested full}
impact_assessment:
  affected_populations: [1 customer]
  magnitude_estimate: {financial: $87.50, reputational: low}
compliance_trace:
  - {control_id: PCI-DSS-3.2.1, satisfied_by: dry-run-sandbox-id, evidence_ref: ...}
rehearsal_ref: rollback-rehearsal-2026-Q1
provenance_ref: pl-refund-00723
# dual_attestation omitted — below $500 threshold
```

### Step 6: Pin (§7)

**Process Owner:** Head of Customer Operations (named individual on the workflow design doc).
**RefundOps does NOT appear on the org chart.**

If a refund goes wrong — wrong customer, wrong amount, regulatory exposure — the Process Owner is accountable. The fact that the agent decided, the dry-run passed, and two reviewers approved does not redirect accountability. It documents how the failure occurred.

### What this gets us vs. the pre-AWG version

A typical pre-AWG version of this workflow:

> "RefundOps processes refunds. Engineers monitor errors. Humans approve refunds over $500 via Slack."

Failures of that version against P1–P4:
- **No Process Owner** — accountability diffused across "the team."
- **No design-attestation** — no audit trail of who approved the gate composition.
- **No scope-bound enforcement** — "policy limit" lives in prompt, not in capability token.
- **No schema** — handoff is whatever the agent says in Slack.
- **Slack approval** — not auditable, identity socially-verified, agent can proceed regardless.
- **"Engineers monitor errors"** — no `policy-halt`, no named alert routing, no `drift-monitor`.

AWG transforms each of these into a structural control rather than a procedural expectation. That transformation is the framework's payoff.

---

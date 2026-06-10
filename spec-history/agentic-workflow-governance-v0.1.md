# Agentic Workflow Governance (AWG)

*A framework for designing, gating, and pinning accountability in workflows where AI agents take action.*

**Version:** 0.1 — skeleton draft (structural review, not field-ready)
**Layer:** 1 of 2 (org-agnostic framework; operating models live in Layer 2)
**Scope:** Agency level L1 and above (see §2)
**License:** TBD (Creative Commons recommended)

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

A small, reusable library of gate types. Workflows compose them rather than inventing bespoke gates.

| Primitive | Description | Typical Use | Structural Requirement |
|------|------|------|------|
| `human-review-required` | A named human reviews and approves before transition | YELLOW tier transitions, L1 production action | Handoff schema (§6) satisfied |
| `dual-control` | Two independent humans approve; the proposer cannot be the approver | RED tier irreversible actions, sensitive data writes | Separation of duties enforced in tooling |
| `dry-run-required` | Action runs against non-production with logged effects before real execution | Any L2/L3 production-mutation | Reproducible dry-run environment |
| `observable-period-required` | Action takes effect; a defined window of automated checks can roll back | Bulk operations, gradual rollouts | Rollback automation + monitoring SLIs |
| `rollback-rehearsed` | Rollback procedure has been executed end-to-end at least once before this action goes live | RED tier, irreversible-with-effort | A rehearsal log entry referenced by ID |
| `circuit-breaker` | Automated condition (rate, error, signal) halts the agent and pages a human | L3 open autonomy | SLI definition + alert routing |

**Composition principle:** higher-tier work assembles multiple primitives. A RED-tier deployment might require `dual-control` + `dry-run-required` + `rollback-rehearsed` + `circuit-breaker`. The framework does not enumerate every valid composition — it provides the primitives and the matrix (§8) and lets adopters compose.

**Open question:** whether `circuit-breaker` is a primitive or a design pattern composed of monitoring + automated halt. Flagged for §10.

---

## 6. Component 4 — Handoff Schemas

The most error-prone gate is one that exists structurally but produces rubber-stamps in practice. Handoff schemas exist to make rubber-stamping structurally hard. They define the *required content* of the artifact passed at every AI→human or human→AI transition.

**Minimum handoff (required for `human-review-required` gates):**

- **Intent** — what the agent is about to do, in plain language, one paragraph.
- **Diff** — the concrete state change (file diff, DB delta, config change, API call payload, etc.). Not a description of the diff — the actual diff.
- **Justification** — why this is the right action, traced to specific inputs. Not "because the user asked."
- **Evidence** — what was checked: tests run, dry-run output, validations passed. Each evidence item is an artifact reference.
- **Reversibility** — explicit rollback procedure, or a flagged "irreversible after step X" marker.
- **Open Questions** — assumptions made and uncertainties unresolved. Empty list is suspicious.

**Higher-tier handoffs add:**

- **Alternatives Considered + Rejection Rationale** — for L3 / RED tier.
- **Impact Assessment** — for External blast radius (affected populations, expected magnitude).
- **Compliance Trace** — for regulated work (which control this satisfies, evidence reference).

**Comparative example:**

*Bad handoff (rubber-stamp bait):* "I'm about to run the cleanup script. Approve?"

*Good handoff:* "Intent: cancel orders flagged in ticket #4521. Diff: UPDATE orders SET status='cancelled' WHERE id IN (...) — 247 rows affected, IDs attached. Justification: each ID matches the ticket's customer-confirmed cancellation list. Evidence: dry-run on staging snapshot produced identical row count; spot-check of 5 random IDs confirmed customer email match. Reversibility: rollback transaction in `rollback-4521.sql`, tested on staging. Open questions: should downstream notification webhooks fire? I recommend disabling them — please confirm."

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

Where the components compose. A skeleton matrix — adopters extend / customize for their domain.

| Tier | Plan | Act | Verify | Release | Monitor |
|------|------|------|------|------|------|
| **GREEN** | none | none | spot-check audit | none | standard alerts |
| **YELLOW** | `human-review-required` | `dry-run-required` | `human-review-required` | `human-review-required` | `circuit-breaker` |
| **RED** | `dual-control` | `dry-run-required` + `rollback-rehearsed` | `dual-control` | `observable-period-required` | `circuit-breaker` + named-owner pager |

Read as: "for a YELLOW-tier workflow, the Plan phase requires a human-review-required gate." Cells can be empty (no gate at this phase for this tier) but rows cannot — every L1+ tier has at least one gate somewhere.

---

## 9. What AWG Is Not

To keep scope honest:

- **Not a model risk framework.** Use NIST AI RMF for model evaluation, bias testing, and lifecycle.
- **Not a regulatory compliance framework.** Use ISO/IEC 42001 or jurisdiction-specific resources (EU AI Act, etc.) for that.
- **Not a security review process.** Threat modeling, AppSec, and supply-chain controls are separate concerns.
- **Not a checklist.** It is a structural toolkit — workflows compose primitives based on classification, not by ticking boxes.
- **Not for L0 systems.** Suggest-only AI does not produce the responsibility-shift mechanism AWG addresses.
- **Not a substitute for engineering rigor.** Tests, code review, observability, incident response — those are still required and AWG composes with them, not over them.

---

## 10. Open Design Questions

Items flagged during skeleton drafting; intended to be resolved before v1.0.

1. **Data sensitivity as a fourth classification axis** — or absorbed into Stakes? Probably distinct, since regulated data triggers different controls than financially-high-stakes work.
2. **Interaction with existing change management** (ITIL CAB, SOX-style sign-offs) — does AWG replace, supplement, or pass through?
3. **Re-classification cadence** — workflows drift over time as agents are retrained or guardrails relax. What forces a re-classification?
4. **Tooling neutrality** — does the framework prescribe formats for handoff schemas (YAML schema? JSON?), or stay tool-agnostic and let adopters implement?
5. **Circuit-breaker classification** — primitive or composed pattern?
6. **Handling of multi-agent workflows** — how does accountability pin when multiple agents interact? Single Process Owner of the whole workflow, or per-agent?
7. **The `q̂` problem** — if perceived risk is what raises monitoring effort, can the framework specify *signals* that should raise it (drift in agent confidence, recent incidents, novel input distributions) and tie them to gate intensity dynamically?

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

## Appendix A — Glossary (skeleton)

- **Agent** — an AI system that takes action, not merely produces text. (See L1+ in §2.)
- **Process Owner** — the named human accountable for an AWG-governed workflow.
- **Governance Function** — the body that maintains AWG in an organization. Form varies by scale (Layer 2).
- **Gate** — a structural checkpoint between phases that conditionally permits transition.
- **Handoff** — the artifact passed at a gate; its required content is defined by the handoff schema.
- **Tier** — the GREEN / YELLOW / RED output of work classification; drives gate intensity.

---

## Appendix B — References

- Wiles, E., Hsu, M., Bedard, J., & Kropp, M. (2026). *Putting AI on the Org Chart: Evidence on Delegation and Oversight.* Boston University / BCG working paper.
- NIST AI Risk Management Framework (AI RMF 1.0).
- ISO/IEC 42001:2023 — Artificial Intelligence Management System.
- Aghion, P., & Tirole, J. (1997). *Formal and Real Authority in Organizations.* Journal of Political Economy.
- Holmström, B. (1979). *Moral Hazard and Observability.* Bell Journal of Economics.

---

*Skeleton ends here. Next iteration: drill the component most likely to be wrong on first pass — candidates flagged in §10.*

# Changelog

All notable changes to Agentic Workflow Governance (AWG) are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The `Resolved (open design questions)` section is a project-specific extension that tracks closure of items previously flagged in §10 of the AWG specification.

---

## [0.2.0] — 2026-05-11

A structural rework of the gate primitive library. The taxonomy now sorts primitives along a three-axis space (lifecycle / authority / function) and distinguishes atomic primitives from composition patterns. The handoff schema and AWG matrix are updated to reflect the new primitive library.

### Added

- **Composition patterns section (§5.2).** Six named patterns — circuit-breaker, dual-control, canary-deploy, rollback-rehearsed, time-bounded-contractor, multi-agent-supervisor — declared as compositions of atomic primitives rather than primitives in their own right. Each follows the same uniform schema as primitives (composes / when-to-use / structural requirements / anti-pattern).
- **Anti-patterns section (§5.3).** Catalog of gate-shaped constructs that fail Principle 4 ("cheap talk does not govern"), with the structurally enforced primitive replacement called out for each.
- **`uncertainty-declaration` primitive.** New atomic primitive requiring agents to surface a calibrated uncertainty signal as a required handoff field; downstream gate intensity may be dynamically conditioned on the value.
- **`scope-bound-execution` primitive.** Machine-enforced bounds (rate limits, dollar limits, allowed-resource lists, identity scopes) encoded in capability tokens or middleware rather than in the agent's prompt.
- **`provenance-log` primitive.** Immutable append-only record of every agent action, promoted to first-class status to serve as the substrate for accountability (P1) and the signal source for `drift-monitor` and `policy-halt`.
- **`time-to-live` primitive.** Platform-enforced expiration of agent authority; renewal requires re-attestation rather than silent continuation.
- **`drift-monitor` primitive.** Continuous comparison of agent behavior to a deployment-time baseline; significant drift halts the agent and forces reclassification through `design-attestation`.
- **`policy-halt` primitive.** Declarative, version-controlled halt-on-condition primitive enforced below the agent; resume requires fresh `human-attestation`.
- **`design-attestation` primitive.** Design-time multi-human (RED) or single-human (lower tiers) attestation that the workflow design — including gate composition, scope bounds, Process Owner, rollback procedure, idempotency analysis, TTL parameters — is approved.
- **Calibration scope boundary (§9).** Explicit clarification that AWG requires the *declaration* of uncertainty but defers calibration *evaluation* to model-risk frameworks (e.g., NIST AI RMF). Closes a structural ambiguity in v0.1.
- **Glossary entries (Appendix A).** *Atomic primitive*, *composition pattern*, *coordinate*, *provenance*.

### Changed

- **§5 (Gate Primitives) fully rewritten.** Replaced v0.1's six-item primitive list — which mixed gate types, gate parameters, pre-conditions, post-conditions, design-time prerequisites, and composed controls as if they were peers — with a three-axis primitive taxonomy:
  - **Lifecycle phase** — design-time / pre-action / mid-action / post-action / continuous
  - **Authority required** — automated / single-human / multi-human / out-of-band
  - **Function** — constrain / verify / attest / observe / halt / reverse

  Every primitive is now characterized by a coordinate in this space. The taxonomy is the framework's self-check: a "primitive" that doesn't fit cleanly is either misnamed or actually composed of smaller primitives.
- **Primitive library size: 6 → 10.** The new set is more atomic and closes gaps from v0.1 — there is now a primitive for bounded scope, for provenance, for time-bounded authority, for drift detection, and for uncertainty declaration, none of which existed in v0.1.
- **`dual-control` collapsed into `human-attestation(n, sod)`.** Dual control is now expressed as a parameter on the human-attestation primitive (`n=2, sod=true`) rather than as a primitive of its own. Reduces taxonomy noise and clarifies that "single approver" and "dual control" are points on a single parameterized primitive.
- **§6 handoff schema.** Added `uncertainty` as a required schema field to carry the `uncertainty-declaration` output into the gate review artifact. Reviewers can now condition decisions on the agent's stated confidence.
- **§8 matrix.** Cells now specify primitive coordinates (with parameters) per tier × phase rather than referencing v0.1's higher-level gate names. The matrix is no longer self-contained; reading it requires the §5 primitive library and §5.2 patterns as inputs.

### Removed

- **`dual-control` as a standalone primitive.** Replaced by `human-attestation(n=2, sod=true)`. See Changed.
- **`circuit-breaker` from the primitive library.** Moved to §5.2 composition patterns; expressed as `provenance-log` (signal source) + `policy-halt` (halt action), optionally with `drift-monitor` and/or `scope-bound-execution` as additional halt sources.
- **`rollback-rehearsed` from the primitive library.** Moved to §5.2 composition patterns; expressed as `design-attestation` carrying a rehearsal-log artifact reference, optionally combined with `observable-period`.
- **`human-review-required` and `observable-period-required` as standalone primitive names.** Renamed to `human-attestation` (parameterized) and `observable-period` respectively, to align with the new naming convention (primitive names describe the *operation*, not the *imperative*).

### Resolved (open design questions)

- **§10 Q5** — *Is circuit-breaker a primitive or a composed pattern?* **Resolved as a composed pattern.** Decomposes into `provenance-log` (signal source) + `policy-halt` (halt action).

### Partially addressed

- **§10 Q7** — *The `q̂` problem (perceived-risk dynamics).* Partially addressed in v0.2 by the combination of `uncertainty-declaration` (raises `q̂` from the agent side) and `drift-monitor` (raises `q̂` from the environment side). Remaining sub-question: should AWG enumerate canonical signals that should raise `q̂`, or leave signal selection to adopters? Carried forward to v0.3+.

### Added to the open-questions list

- **§10 Q8** — *Calibration boundary.* `uncertainty-declaration` requires the declaration but defers calibration to model risk. Is this delegation sound, or does AWG need to require evidence of calibration evaluation as part of `design-attestation`?

---

## [0.1.0] — 2026-05-09

Initial skeleton draft. Establishes the framework's structure, principles, scope, and component skeleton. Multiple sections marked explicitly as v0.1-quality and pending structural review.

### Added

- **Origin section (§0).** Mechanism and motivation grounded in Wiles, Hsu, Bedard & Kropp (2026), *Putting AI on the Org Chart: Evidence on Delegation and Oversight* (Boston University / BCG working paper). AWG positioned as filling the agentic governance gap between model-risk frameworks (NIST AI RMF), management-system standards (ISO/IEC 42001), and regulatory compliance kits (EU AI Act).
- **Four load-bearing principles (§1).**
  - **P1** — Accountability is pinned to a human.
  - **P2** — Gates are structural substitutes for an absent moral-hazard reflex.
  - **P3** — Autonomy is earned per-tier, not granted globally.
  - **P4** — Cheap talk does not govern.
- **Agency-level scoping (§2).** Four-level taxonomy (L0–L3) distinguishing suggest-only systems (L0, out of AWG scope), approve-and-execute (L1), bounded autonomy (L2), and open autonomy (L3). Explicit statement that AWG applies only to L1+.
- **Work classification (§3).** Three-axis scoring (stakes / reversibility / blast radius) plus agency level, producing a GREEN / YELLOW / RED tier that drives gate intensity downstream.
- **Phase model (§4).** Five-phase model — Plan, Act, Verify, Release, Monitor — as the placement guide for gate primitives. Phase optionality is allowed; not every workflow exercises every phase.
- **Initial gate primitive library (§5).** Six primitives: `human-review-required`, `dual-control`, `dry-run-required`, `observable-period-required`, `rollback-rehearsed`, `circuit-breaker`. *(Restructured in v0.2; see the v0.2 entry above.)*
- **Handoff schema concept (§6).** Defined required content for `human-review-required` artifacts: Intent, Diff, Justification, Evidence, Reversibility, Open Questions. *(Extended in v0.2 with `Uncertainty` as a required field.)*
- **Accountability principle (§7).** Four normative requirements pinning workflow accountability to a named human Process Owner. Codifies that AI agent names do not appear on org charts; Process Owner names appear on workflow design documents.
- **AWG matrix (§8).** Tier × phase composition matrix specifying gate requirements per intersection.
- **Scope-boundary section (§9, "What AWG Is Not").** Explicit non-goals: not a model risk framework, not a regulatory compliance framework, not a security review process, not a checklist, not for L0 systems, not a substitute for engineering rigor.
- **Open design questions (§10).** Initial set of seven open questions deferred to subsequent versions: (1) data sensitivity as a fourth classification axis, (2) interaction with existing change management, (3) re-classification cadence, (4) tooling neutrality, (5) circuit-breaker classification, (6) multi-agent accountability, (7) the `q̂` problem.
- **Application guide (§11).** Six-step adopter walkthrough — locate, classify, map, compose, schema, pin.
- **Glossary (Appendix A).** Initial entries — *agent*, *Process Owner*, *governance function*, *gate*, *handoff*, *tier*.
- **References (Appendix B).** Wiles et al. (2026), NIST AI RMF 1.0, ISO/IEC 42001:2023, Aghion & Tirole (1997), Holmström (1979).

### Notes

- v0.1 self-identified the primitive list in §5 as the most likely structurally wrong component on first pass. The v0.2 rework confirmed and addressed this.
- v0.1's §8 matrix referenced primitive *names* rather than coordinates with parameters. Sharpened in v0.2.

[0.2.0]: https://github.com/Cadtastic/Agentic-Workflow-Governance/releases/tag/v0.2.0
[0.1.0]: https://github.com/Cadtastic/Agentic-Workflow-Governance/releases/tag/v0.1.0

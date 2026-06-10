# AWG v0.3 — Strategic Review (June 2026)

*A deep review of the Agentic Workflow Governance framework against the post-Fable-5 landscape: does it still make sense, what should change, and what it takes to become an industry standard.*

**Reviewed:** AWG v0.3 skeleton
**Date:** 2026-06-09
**Method:** Five-angle deep research (competing frameworks, human-oversight empirics, capability frontier, standardization dynamics, runtime substrate), followed by adversarial verification of 18 load-bearing claims against primary sources (17 confirmed, 1 minor correction). Sources in Appendix R.

---

## 1. Verdict: Is AWG still beneficial?

**Yes — and more than it was at v0.1. The release of Claude Fable 5 strengthens the case rather than weakening it.** Four findings support this:

**1.1 The gap AWG targets is still empty.** As of June 2026, every adjacent framework was checked at the primary source. The pattern is consistent: everyone says "human approval for consequential actions"; *nobody* specifies where in a workflow gates go, how to derive placement from risk, how to pin accountability to a named human, or what the governance payload of a handoff must contain. Specifically:

| Layer | Occupant | What it covers | What it leaves to AWG |
|---|---|---|---|
| Threat awareness | OWASP Agentic Security Initiative (Top 10 for Agentic Applications, Dec 2025, ASI01–ASI10; Threats & Mitigations v1.0, Feb 2025) | What can go wrong | No gate-placement methodology, no accountability model |
| Audit controls | CSA AI Controls Matrix v1.0 (Jul 2025; 18 domains, 243 control objectives) | What auditors check | Organization-level controls, not workflow-level design |
| Federal baselines | NIST COSAiS (single-agent and multi-agent SP 800-53 overlays **announced but unpublished**); CAISI AI Agent Standards Initiative (Feb 2026) | Coming, not landed | NIST's own Jan 2026 RFI *asks* how approval gates for consequential actions should work — open question, not answer |
| Management systems | ISO/IEC 42001:2023 (+42005:2025 impact assessment) | Org-level AIMS; Annex A.9 says "human oversight" exists | No workflow granularity; no published ISO agent deliverable exists |
| Regulation | EU AI Act (Art. 14 human oversight; Annex III high-risk now deferred to Dec 2027 by the May 2026 Digital Omnibus) | Outcome requirements ("effective oversight") | Not workflow design; widely criticized as unoperationalizable |
| Operational guidance | Five Eyes "Careful Adoption of Agentic AI Services" (May 1, 2026) | Names **accountability opacity** as a top-5 agentic risk | Advisory only — no controls, schemas, or placement rules |
| Protocol/runtime | MCP (AAIF/Linux Foundation), A2A v1.0, Entra Agent ID, AWS AgentCore Policy, Google Agent Registry, Microsoft Agent Governance Toolkit (Apr 2026) | Transport, identity, per-action policy enforcement | Enforcement machinery with no doctrine: *where humans approve* is left entirely to the implementer |

The white space AWG occupies — a practitioner-level, workflow-scoped governance methodology sitting between OWASP's threat list and CSA's 243 audit controls — is real and verified. Searching "agentic workflow governance" still returns vendor marketing, not a canonical open framework.

**1.2 The capability frontier raises, not lowers, the value of workflow governance.** METR's Time Horizon 1.1 (Jan 2026): the task duration agents complete at 50% reliability doubles every ~7 months long-run (~89–131 days in the post-2023/24 sub-trend); Claude Opus 4.5 measured at ~320 minutes. Fable 5's launch claims include a codebase migration compressed from two months to a day and >1 week of largely autonomous research work. Longer autonomous chains mean compounding error: a 90%-per-step agent over 20 independent steps lands near 12% end-to-end — and errors in practice correlate and amplify ("spiral of hallucination," Salesforce 2026). Per-step competence is rising faster than end-to-end trajectory reliability. That is precisely the regime where *phase-boundary* governance (AWG's core contention, §4.3) beats per-action review.

**1.3 Fable 5 itself validates the AWG pattern — one layer down.** Anthropic ships Fable 5 with classifier-gated capability tiers: flagged requests fall back to Opus 4.8 in <5% of sessions, and Mythos 5 (same model, fewer safeguards) is restricted to approved organizations. That is tier-resolution + policy-halt + scope-bound-execution implemented *inside the model layer*. It does nothing for the workflow layer: which business actions the agent may commit, who approves them, and who is accountable remain the deployer's problem. The model layer getting governance makes the ungoverned workflow layer the conspicuous residual risk.

**1.4 Independent convergence on AWG's core ideas.** Three things AWG asserted in v0.1–v0.3 have since appeared independently: Gartner (May 26, 2026) — "applying uniform governance across AI agents will lead to enterprise AI agent failure"; governance must be proportional to autonomy level (≈ AWG's tier model), and 40% of enterprises will demote or decommission agents by 2027 due to governance gaps. Microsoft Entra Agent ID (GA Apr 2026) ships **owner/sponsor** fields on every agent identity (≈ P1 Process Owner — the only shipped accountability-pinning mechanism found anywhere). Five Eyes names accountability opacity a top-5 risk (≈ P1's motivation). Convergence by parties who have never read AWG is the strongest available evidence that the framework is pointed at the right problem.

**Risk to the thesis:** the window is 12–24 months. NIST's agent overlays and the CAISI standards initiative will eventually occupy adjacent territory, and OWASP's ASI suite is expanding quarterly. AWG wins by being earlier, more concrete, and machine-readable — or not at all.

---

## 2. The evidence demands one major correction: gates are not free

The single most important finding of this review cuts *against* v0.3's instincts. The framework's §3.3 declares "the default is over-governed, not under-governed." The empirical literature says over-governance is not a safe harbor — it is a distinct failure mode:

- **Human review degrades outcomes more often than intuition suggests.** Vaccaro, Almaatouq & Malone (Nature Human Behaviour, 2024; 106 studies, 370 effect sizes): on average, human+AI combinations performed *worse* than the best of human or AI alone, with losses concentrated in decision tasks. Synergy occurs only when the human outperforms the AI on the gated judgment. Green & Chen (2019): human reviewers of algorithmic risk predictions underperformed the algorithm alone *and* introduced racial disparities.
- **Approval gates rubber-stamp at scale.** Anthropic's production data (Mar 2026): Claude Code users approve **93% of permission prompts** — stated plainly as approval fatigue making manual prompts weak protection. Clinical decision support shows 49–96% alert override rates sustained across two decades of study. A gate everyone approves is P4 cheap talk wearing a gate costume.
- **Escalation can be blame-laundering, not quality control.** The Wiles et al. RCT that motivates AWG §0 found the AI-employee framing *increased* escalation requests (+44%) while *decreasing* review quality (−16%) — more process, less protection — and explicitly models escalation as creating "a record that the manager did not sign off alone."
- **Mandated oversight without designed efficacy is theater.** Green (2022) surveyed 41 government policies requiring human oversight of algorithms: people empirically could not perform the prescribed oversight, and the requirement then *legitimized* flawed systems. The same critique is now standard against EU AI Act Art. 14.
- **And yet gates work when designed well.** Atlassian's HULA agent: human-approved plans yielded 87% code-generation success; selective-review frameworks (learning-to-defer; KnowNo's conformal prediction, CoRL 2023 Best Student Paper) achieve near-full-review quality at a fraction of review cost by gating *selectively* on calibrated uncertainty.

The reconciliation across this literature is clean and AWG should own it: **a human gate adds value if and only if the human can actually catch what the machine misses, has the information and authority to act, and reviews at a volume that preserves attention.** Otherwise the correct control is automated (scope bounds, invariant checks, sampling audits, halts) — not a human pretending to review.

v0.3 already has the right anti-pattern instincts (§5.3, §6.0 "defeats rubber-stamping"). What it lacks is the affirmative decision rule. Section 3 below supplies it. This also means §3.3's bias-upward rule needs a counterweight: biasing *tier* upward is fine; biasing toward *more human gates* specifically is not, because dead gates are negative-value (they consume attention budget, manufacture false assurance, and produce the Wiles blame-record dynamic).

---

## 3. The Stop-Gate Decision Model (proposed new spec section)

This is the framework's answer to "when and when not to use human stop gates" — the piece every adjacent framework punts on. Proposed as a new §5.4 (or §6.7) in v0.4.

### 3.1 Placement test

A `human-attestation` gate is justified at a given point **only if all five conditions hold** (derived from Sterz et al. FAccT 2024's effectiveness conditions + Vaccaro et al.'s synergy condition + the attention-budget literature):

| # | Condition | Test | Failure means |
|---|---|---|---|
| G1 | **Competence advantage** | The named reviewer class plausibly catches material errors that the automated checks (dry-run, invariants, scope bounds) do not, for *this* judgment type | Use automated primitives instead |
| G2 | **Epistemic access** | The handoff schema (§6) gives the reviewer the specific content needed to make that catch — not a wall of context | Fix the schema or remove the gate |
| G3 | **Causal power** | Execution structurally waits; the reviewer can block, modify, or escalate, and self-approval is rejected | Not a gate — remove or re-implement |
| G4 | **Attention budget** | Expected gate volume × honest review time fits the named reviewers' capacity with slack. A gate firing 50×/day per reviewer is pre-failed | Raise the gate threshold, gate selectively, or automate |
| G5 | **Leverage point** | The gate sits at a phase boundary involving irreversibility, blast-radius expansion, or plan commitment — not uniformly per-action | Move the gate to the boundary; govern the steps with `scope-bound-execution` |

When any condition fails, the spec should *name the substitute*: `scope-bound-execution` + `policy-halt` for high-volume/low-stakes actions; `dry-run` + automated invariant checks where machines outperform reviewers; post-hoc sampling audits via `provenance-log` where per-action review fails G4; `observable-period` with rehearsed rollback where reversibility, not prevention, is the cheaper control.

### 3.2 Selective gating beats uniform gating

The strongest result in the deferral literature (learning-to-defer; KnowNo): gate on *calibrated uncertainty*, not uniformly. AWG already has the machinery — `uncertainty-declaration` conditioning gate intensity (§5.1, RED tier guidance). v0.4 should promote this from a RED-tier feature to the *default recommended shape* of `human-attestation` at YELLOW+: the gate fires when the uncertainty signal exceeds threshold, when the action is irreversible, or when scope bounds are approached — and otherwise does not. Reference implementation guidance: conformal/set-based methods (ask when the prediction set is non-singleton) over raw confidence scalars, because RLHF-tuned confidence calibration is unreliable (flagged as a hypothesis with growing support, not settled fact).

A second granularity lever, validated in production by Anthropic's plan-mode pattern: **plan-level attestation** — approve the agent's plan once at the Plan/Act boundary, let execution proceed ungated within the approved plan's scope, and re-gate on plan deviation. This should become an explicit parameterization: `human-attestation(granularity: plan | action | batch)`.

### 3.3 Gate-health monitoring (new requirement)

A placement test at design time is insufficient — gates die in production. v0.4 should require, for every deployed `human-attestation` gate, that `drift-monitor` track gate-health signals:

- **Approval rate.** Sustained ≥ ~95–98% approval = presumptively dead gate (Claude Code's 93% is the empirical anchor). Triggers redesign review through `design-attestation`, with three sanctioned outcomes: raise the threshold so the gate fires less but means more; convert to selective gating; or convert to automated control + sampling audit.
- **Review latency distribution.** Median time-to-approve below a floor calibrated to the schema's content (e.g., approving a 247-row destructive diff in 4 seconds) = stamping, not reviewing.
- **Seeded-error catch rate (optional, RED).** Periodically inject known-bad handoffs (flagged in `provenance-log`, never executable) and measure reviewer catch rate — the Wiles RCT methodology operationalized as a control, analogous to phishing simulation. This makes oversight *measurable*, which no adjacent framework currently offers.

This closes AWG's own P4 loop: a gate whose effectiveness is never measured is itself cheap talk.

### 3.4 What this gives the framework

A defensible, citable answer to the question regulators, NIST's RFI, and every enterprise architect are currently asking — with the supporting evidence base (Vaccaro, Green, Sterz, KnowNo, production approval-rate data) built into the spec's references rather than asserted by fiat.

---

## 4. Keep / Change / Add / Drop

### 4.1 KEEP (validated, now with stronger evidence)

- **P1 Process Owner.** Independently convergent with Entra Agent ID sponsors, Five Eyes "accountability opacity," and the Wiles policy conclusion ("an explicitly accountable human owner"). This is AWG's most defensible principle. Keep §7 point 4 ("AI agent names do not appear on org charts") — it is now directly supported by the Wiles subgroup finding (below).
- **P4 and the anti-pattern catalogs (§5.3, §6.5).** The 93% approval-rate datum is the empirical vindication of "optional approval is not a gate." Cite it.
- **Tier model + deterministic resolution (§3).** Convergent with Gartner's proportional-governance position and Claude Code's tiered autonomy design. The `max()` rule and RED-stacking survive review.
- **Primitive taxonomy (§5).** All ten primitives map cleanly onto the 2026 runtime substrate (see 4.3 mapping) — none is obsolete, none missing from the atomic set except identity (treated below as a cross-cutting requirement, not an eleventh primitive).
- **Schema-as-contract (§6).** The ecosystem converged on JSON Schema 2020-12 (MCP's 2026-07-28 spec, A2A Agent Cards, structured outputs). AWG's bet on machine-validatable handoffs was correct.
- **Phase model (§4) and phase-boundaries-are-gates.** The compounding-error math is the quantitative argument for this design: governing boundaries scales with chain length; governing actions does not.

### 4.2 CHANGE

1. **§0 — correct the Wiles characterization (credibility-critical).** v0.3 presents the 16%/44% numbers as the study's headline effects. Verified against the primary PDF: *full-sample average effects are small*; the headline effects are pre-registered heterogeneous effects concentrated in the ~23% of managers whose organizations already list AI agents on org charts. It is also a framing experiment (identical documents), one-shot, HR/finance reviewers, working paper, not yet peer-reviewed. An industry-standard-aspiring spec must state this precisely — the subgroup finding is actually *more* useful to AWG (it shows the damage follows the org-chart framing, which is exactly what P1/§7.4 prohibits), and precision here is what separates a standard from an opinion piece. Simultaneously, **broaden the evidence base** beyond one working paper: Vaccaro et al. 2024, Green & Chen 2019, Green 2022, the automation-bias literature, and the production approval-rate data all point at the same mechanism and are independently citable.
2. **§3.3 — re-scope the bias-upward rule.** Bias *tier* upward under ambiguity: keep. Add the counterweight: tier upgrades must route through the §3-new stop-gate placement test before adding human gates. Over-gating is a named failure mode (cite Gartner's 40% decommission prediction and the dead-gate evidence), not a safe default.
3. **§3.5 — ratify the fifth axis.** Add data sensitivity as a classification axis (regulated → ≥YELLOW; special-category → ≥RED) as proposed. The landscape ratifies it: every audit framework (AICM, 42001, AI Act) keys on data category, and absorbing it into Stakes loses the mapping.
4. **§5.1 `uncertainty-declaration` — harden the calibration caveat.** Evidence is mounting that preference-tuning degrades confidence calibration, and overconfidence concentrates where it is most dangerous. Keep the declaration requirement; change the recommended form from scalar confidence to set-based/conformal signals where available; make the §5.1 scope-boundary language ("AWG requires the declaration; model risk ensures the signal is meaningful") more prominent, and require at `design-attestation` (RED) that *some* calibration evidence exists before gates are conditioned on the signal — resolving open question §10 #8 in the affirmative.
5. **§5.1 evidence fields — close the adversarial hole.** v0.3's schema treats agent-produced content as honest-but-fallible. OWASP ASI01 (goal hijacking) is the #1 agentic risk, and 2026 brought working attacks where the approval prompt itself lies (symlink-disguised actions across five major coding agents; tool-description injection; the Supabase/Cursor ticket-injection incident). Required change: **`evidence` and `diff` fields must be generated by infrastructure the agent does not control** (the dry-run harness writes its own artifact; the diff comes from the execution environment's snapshot, not the agent's narration). One sentence in §6.2 field semantics and one new anti-pattern ("agent-narrated evidence") close this.
6. **§5.2 multi-agent supervisor — strengthen with the failure taxonomy.** MAST (arXiv:2503.13657) gives 14 failure modes in 3 categories (specification ~42%, inter-agent misalignment ~37%, task verification ~21%) — map the supervisor pattern's structural requirements onto them. Add: cross-agent provenance must be *correlated* (W3C Trace Context propagation, now formalized in MCP) so the workflow-level `provenance-log` reconstructs causality across agents. Add a sober note on limits: text-output monitoring does not reliably detect steganographic inter-agent collusion (2026 interpretability results); AWG's mitigation is scope minimization and supervisor-level halts, not content monitoring. Keep single Process Owner (§10 #6: resolve as-is — the workflow owner eats the loss; per-agent attribution is forensic, not accountability-bearing).
7. **§2 L0 boundary — keep, with one caveat sentence.** L0 (suggest-only) stays out of scope, but §2 should note that automation bias operates at L0 too (humans over-adopt suggestions; Green & Chen's reviewers *were* the executors) — adopters wanting L0 coverage should look to model-risk and decision-support literature. This preempts the most obvious scope criticism without expanding scope.
8. **§6 — version the schema and assign stable IDs.** Adopt JSON Schema 2020-12 as the normative schema language (resolving §10 #9: yes, publish an official artifact; flexibility-vs-prescription is settled by the ecosystem having already converged). Assign stable, citable identifiers to every primitive, pattern, and anti-pattern (`AWG-P01`…`AWG-P10`, `AWG-C01`…, `AWG-A01`…) — the ATT&CK/ASI lesson: stable IDs are what let other people's documents, tools, and audits *reference* you, and referencing is adoption.

### 4.3 ADD

1. **The Stop-Gate Decision Model** (§3 of this review) — the centerpiece addition.
2. **Substrate mapping appendix (normative "map, don't reinvent" layer).** AWG primitives should declare their reference enforcement substrates, which in 2026 exist for every primitive:

| AWG construct | Maps to (2026 substrate) |
|---|---|
| `human-attestation` | MCP elicitation (protocol-native HITL); approval middleware (HumanLayer, gotoHuman, Permit.io); Microsoft AGT quorum approvals |
| `scope-bound-execution` | Cedar/OPA policy enforcement points at the tool-call layer (AWS AgentCore Policy pattern: default-deny, evaluated per action, decided *below* the LLM); capability tokens; sandbox + network-layer egress policy |
| `provenance-log` | OTel GenAI semantic conventions + W3C Trace Context for correlation; OCSF v1.8 for security events; W3C PROV as the data model; in-toto-style attestation envelopes for tamper-evidence |
| `time-to-live` | SPIFFE/SPIRE short-lived SVIDs; Entra Agent ID lifecycle policies; OAuth token expiry |
| Agent identity (cross-cutting) | Entra Agent ID (org), SPIFFE (runtime), A2A signed Agent Cards (cross-org), IETF OAuth on-behalf-of drafts (delegation — track, don't hard-depend) |
| Handoff schemas (§6) | JSON Schema 2020-12; A2A task state machine for agent-to-agent handoffs |
| `dry-run` / `observable-period` | Sandbox infrastructure (microVM-class isolation); canary/progressive-delivery tooling |

   This converts AWG from prose into something a platform team can implement against, and it positions AWG as the *doctrine layer* over the enforcement stack the vendors built — complementary to all of them, competitive with none. (That complementarity is itself an adoption strategy: vendors can cite AWG to explain *how* to use their enforcement products.)
3. **Crosswalks.** Ship mappings from AWG primitives/requirements to NIST AI RMF functions, ISO/IEC 42001 Annex A.9, CSA AICM control IDs, and OWASP ASI01–ASI10 mitigations. Crosswalks are how a new framework enters audit scope (the AICM playbook), and NIST's OLIR program accepts third-party crosswalk submissions into an official NIST catalog — a concrete, underused legitimacy channel available to a single-author framework.
4. **Agent identity as an explicit structural requirement.** v0.3 assumes "registered agent" and `agent_identity` fields without defining what makes an identity governance-grade. Add to §5/§6 structural requirements: agent identity must be platform-issued, distinct per agent, lifecycle-managed (created/owned/expired), and verifiable at the gate — then point at the substrate table rather than specifying a format. Do **not** add an eleventh primitive; identity is a precondition of every primitive, like the provenance substrate.
5. **A threat-model section (one page).** AWG governs honest-but-fallible agents *and* must survive adversarial ones (goal hijacking is OWASP's #1). State the trust boundaries explicitly: what AWG assumes the platform enforces (identity, log integrity, gate wait), which schema fields are agent-asserted vs. infrastructure-asserted, and a pointer to OWASP ASI / MAESTRO for full threat enumeration. This both closes a real hole and pre-empts the security community's first review comment.
6. **Conformance checklist (v1.0 horizon).** A short, testable "AWG-conformant workflow" checklist (every L1+ workflow has a named owner; every human gate passes the placement test; gate-health monitored; schemas validate; provenance correlated). Conformance — not prose — is what badges, attestations, and procurement language get built on (Certified-Kubernetes lesson).

### 4.4 DROP / DEPRIORITIZE

1. **Drop the implicit "more gates = safer" posture** wherever it appears (§3.3's framing, the YELLOW Release row's unconditional `human-attestation(n=1)`). Every human gate in the matrix should carry a placement-test justification or become selective/automated. The matrix rows survive; their unconditional human entries become conditional.
2. **Drop "skeleton" framing.** After this revision the document is a v0.4 draft framework, not a skeleton. (The closing paragraph's self-description should go in the same pass.)
3. **Deprioritize Layer 2.** The org-operating-model layer is the right long-term scope cut, but building it now competes with the machine-readable artifacts for effort, and the artifacts are what the adoption window rewards. Keep the Layer 1/2 split declared; publish Layer 2 after v1.0.
4. **Nothing in the primitive library is dropped.** All ten survive contact with the 2026 landscape — the review's most reassuring finding. The taxonomy's self-check (§5.0) held: everything new the ecosystem shipped (quorum approvals, Cedar policies, kill switches, canary tooling) decomposes into existing primitives.

---

## 5. Roadmap to industry standard

The standardization research yields a clear pattern: frameworks win through neutral homes, machine-readable canonical artifacts, stable identifiers, crosswalks into incumbent audit regimes, and tooling embedment — and single-author frameworks that skip governance either stagnate (semver), get forked (Keep a Changelog), or must retrofit open governance later (Twelve-Factor, opened by Heroku in 2024 after 13 frozen years).

**v0.4 (next, weeks):** Incorporate §§2–4 of this review — stop-gate decision model, evidence-base correction and broadening, fifth axis, adversarial-evidence fix, stable IDs, identity requirement, threat-model page. Pure spec work, no infrastructure.

**v0.5 (artifacts, 1–2 months):** Publish the JSON Schema 2020-12 artifact for §6 (base + tier extensions); substrate mapping appendix; NIST AI RMF / ISO 42001 / AICM / OWASP ASI crosswalks; a minimal reference validator (a few hundred lines — validates handoff instances, checks tier-required fields). The validator matters disproportionately: tooling embedment is the single strongest adoption force in every case study.

**v1.0 (standard posture, 3–6 months):** Conformance checklist; OLIR crosswalk submission to NIST; worked example expanded to a multi-agent case. **License decision:** keep CC BY-SA for the prose if desired (OWASP Top 10 itself is CC BY-SA 4.0 — proven viable for a referenced framework), but schemas, validator, and any code must be Apache-2.0 (patent grant; embeddable in commercial tools without copyleft contamination); consider CC BY for prose to ease incorporation into other bodies' documents, and note that CC licenses do nothing for patent claims (the Community Specification License exists for exactly this if AWG becomes a multi-party spec). The OpenStreetMap migration (CC BY-SA → ODbL, 2012, ~1% data loss) is the cautionary tale for choosing virality early.

**Governance home (decision by v1.0):** The dominant winning pattern is early donation to a neutral home. Realistic candidates, in order of fit: **OWASP** (formal project-donation process, Incubator→Flagship ladder, the agentic-security community already lives there, and AWG fills the governance-methodology gap their threat-centric suite leaves); **Agentic AI Foundation** (Linux Foundation; where MCP, AGENTS.md, and goose live — right ecosystem, heavier lift for a methodology vs. a protocol); CSA (controls-adjacent). Single-author stewardship through v1.0 is fine — speed is currently an advantage — but the spec should *declare* the intent to move to community governance, because adopters discount frameworks with bus-factor 1.

**What makes this winnable:** every successful precedent (ATT&CK, OWASP Top 10, AICM) entered an empty layer early with citable structure. The workflow-governance layer is empty, the demand signals are loud (937 comments on NIST's agent RFI; Gartner's failure predictions; Five Eyes naming the exact problem), and AWG's primitives decompose everything the vendors have shipped. The framework's job now is to convert from a well-argued document into a referenceable, implementable, measurable standard before the institutional players fill the gap with something heavier and worse.

---

## Appendix R — Key sources (verified June 9, 2026)

**Oversight empirics:** Wiles, Hsu, Bedard & Kropp (2026), *Putting AI on the Org Chart* — https://www.emmawiles.com/storage/ai_employee.pdf (primary PDF verified: full-sample effects small; 16%/44% are subgroup effects among orgs listing AI on org charts) · Vaccaro, Almaatouq & Malone (2024), *Nature Human Behaviour* — https://www.nature.com/articles/s41562-024-02024-1 · Green & Chen (2019) — https://www.benzevgreen.com/wp-content/uploads/2019/02/19-fat.pdf · Green (2022), *CLSR* — https://arxiv.org/abs/2109.05067 · Sterz et al. (FAccT 2024) — https://dl.acm.org/doi/10.1145/3630106.3659051 · Anthropic, *How we built Claude Code auto mode* (Mar 25, 2026; 93% approval rate) — https://www.anthropic.com/engineering/claude-code-auto-mode · KnowNo (Ren et al., CoRL 2023 Best Student Paper) — https://arxiv.org/abs/2307.01928 · Elish (2019), *Moral Crumple Zones* · Crootof, Kaminski & Price (2023), *Humans in the Loop*, 76 Vand. L. Rev. 429.

**Landscape:** OWASP Top 10 for Agentic Applications (Dec 9, 2025) — https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/ · CSA AI Controls Matrix v1.0 (Jul 2025) — https://cloudsecurityalliance.org/blog/2025/07/10/introducing-the-csa-ai-controls-matrix-a-comprehensive-framework-for-trustworthy-ai · NIST COSAiS — https://csrc.nist.gov/projects/cosais · CAISI AI Agent Standards Initiative (Feb 17, 2026) — https://www.nist.gov/news-events/news/2026/02/announcing-ai-agent-standards-initiative-interoperable-and-secure · Five Eyes, *Careful Adoption of Agentic AI Services* (May 1, 2026) — https://www.cisa.gov/resources-tools/resources/careful-adoption-agentic-ai-services · Gartner (May 26, 2026) — https://www.gartner.com/en/newsroom/press-releases/2026-05-26-gartner-says-applying-uniform-governance-across-ai-agents-will-lead-to-enterprise-ai-agent-failure · EU Digital Omnibus provisional agreement (May 7, 2026) — https://www.consilium.europa.eu/en/press/press-releases/2026/05/07/artificial-intelligence-council-and-parliament-agree-to-simplify-and-streamline-rules/ · Anthropic, *Trustworthy agents in practice* (Apr 9, 2026) — https://www.anthropic.com/research/trustworthy-agents · OpenAI, *Practices for Governing Agentic AI Systems* (Dec 2023, unrevised).

**Capability frontier:** Anthropic, Claude Fable 5 / Mythos 5 (Jun 9, 2026) — https://www.anthropic.com/news/claude-fable-5-mythos-5 · METR, Time Horizon 1.1 (Jan 29, 2026) — https://metr.org/blog/2026-1-29-time-horizon-1-1/ · MAST, *Why Do Multi-Agent LLM Systems Fail?* — https://arxiv.org/abs/2503.13657 · Multi-agent collusion interpretability (2026) — https://arxiv.org/abs/2604.01151.

**Substrate:** MCP 2026-07-28 RC — https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/ · AAIF formation (Dec 9, 2025) — https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation · A2A v1.0 — https://a2a-protocol.org/latest/ · Entra Agent ID owners/sponsors — https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers · Microsoft Agent Governance Toolkit (Apr 2, 2026) — https://github.com/microsoft/agent-governance-toolkit · AWS AgentCore Policy — https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/policy.html.

**Standardization dynamics:** MITRE ATT&CK data/licensing — https://attack.mitre.org/resources/ · OWASP Top 10 license (CC BY-SA 4.0) — https://github.com/OWASP/Top10/blob/master/LICENSE · Twelve-Factor open-sourcing (Nov 2024) — https://www.heroku.com/blog/heroku-open-sources-twelve-factor-app-definition/ · OpenTelemetry CNCF graduation (May 2026) — https://www.cncf.io/announcements/2026/05/21/cloud-native-computing-foundation-announces-opentelemetrys-graduation-solidifying-status-as-the-de-facto-observability-standard/ · NIST OLIR — https://csrc.nist.gov/projects/olir · OSM license migration — https://osmfoundation.org/wiki/Licence/About_The_Licence_Change · OWASP project donation — https://owasp.org/www-committee-project/.

**Known evidence caveats carried honestly:** Wiles et al. is a working paper (not peer-reviewed) and its headline effects are subgroup effects; the RLHF-degrades-calibration claim is well-sourced but not settled; METR's faster sub-trend (~89 days) sits inside the long-run ~7-month trend's uncertainty; vendor benchmark numbers (Fable 5, GPT-5.5, Gemini 3) are self-reported on non-identical harnesses.

---

*Review prepared for Addam Boord, June 9, 2026. Method: five parallel research agents (frameworks, oversight empirics, capability frontier, standardization dynamics, runtime substrate) + adversarial verification of 18 load-bearing claims (17 confirmed against primary sources; 1 corrected: KnowNo won CoRL 2023 Best Student Paper, not Best Paper).*

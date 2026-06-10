# Agentic Workflow Governance (AWG)

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

*A framework for designing, gating, and pinning accountability in workflows where AI agents take action.*

## What AWG is

Agentic Workflow Governance (AWG) is an organization-agnostic framework for designing the
stop-gates, handoffs, and accountability structures in business workflows that involve AI
agents. It targets the *agentic governance gap* — the layer every adjacent framework leaves
open: not *whether* humans should oversee consequential agent actions (everyone says yes),
but **where in a workflow gates belong, when a human gate is justified and when it is
counterproductive, how to keep gates alive once deployed, and how accountability pins to a
named human**.

AWG supplies the structure: per-tier work classification, a five-phase model, a library of
ten atomic gate primitives, a stop-gate decision model, schema-as-contract handoffs, and a
single named human Process Owner per workflow.

AWG is **Layer 1** — the generic framework. Organization-specific operating models (who
performs each role, how work is audited) live in a separate Layer 2 document that adopters
write for themselves.

## Repository layout

| Path | What it is |
|---|---|
| [`SPEC.md`](SPEC.md) | **The canonical specification.** Always reflects HEAD; start here. |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history ([Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format, [SemVer](https://semver.org/spec/v2.0.0.html)). Each entry summarizes what changed and which open design questions moved. |
| [`spec-history/`](spec-history/) | Frozen snapshots of early drafts (v0.1, v0.2), kept for convenient side-by-side reading. Full history is also in git: `git log --follow SPEC.md`. |
| [`reviews/`](reviews/) | Strategic review records — the rationale and evidence behind each version's changes. [`REVIEW-v0.3.md`](reviews/REVIEW-v0.3.md) reviews the framework against the June 2026 landscape and sets the roadmap to v1.0. |
| [`CITATION.cff`](CITATION.cff) | Citation metadata (drives GitHub's "Cite this repository" button and DOI archiving). |
| [`LICENSE`](LICENSE) | CC BY-SA 4.0, full legal code. |

## Where to start

Pointers only — the content lives in the spec:

- **Why this exists** → [`SPEC.md`](SPEC.md) §0 (the two empirically documented failure modes: oversight erosion and oversight theater)
- **Does AWG apply to my system?** → §2 (agency levels L0–L3; AWG applies at L1+)
- **How to apply it, end to end** → §11 (six steps), then Appendix C (complete worked example)
- **When to use a human stop-gate — and when not to** → §5.4 (placement test, selective gating, gate-health monitoring)
- **What a gate handoff must contain** → §6 (machine-validatable schemas)
- **What AWG deliberately does not cover** → §9 (model risk, regulatory compliance, security review, calibration — and where to go for each)

## Status & roadmap

Current state: **v0.4.0** — complete framework with the Stop-Gate Decision Model; not yet 1.0.

Direction (detailed in [`reviews/REVIEW-v0.3.md`](reviews/REVIEW-v0.3.md)): v0.5 adds machine-readable
artifacts — a JSON Schema (2020-12) for the handoff schemas, stable control IDs, crosswalks
to NIST AI RMF / ISO/IEC 42001 / CSA AICM / OWASP ASI, and a reference validator. v1.0 adds
a conformance checklist and settles the long-term governance home. Open design questions are
tracked in [`SPEC.md`](SPEC.md) §10 and closed through `CHANGELOG.md` entries.

## Motivation

AWG began as the structural response to the oversight decline documented in:

> Wiles, E., Hsu, M., Bedard, J., & Kropp, M. (2026). [*Putting AI on the Org Chart:
> Evidence on Delegation and Oversight.*](https://www.emmawiles.com/storage/ai_employee.pdf)
> Boston University / BCG working paper.

Since v0.4 the evidence base is broader — including the meta-analytic finding that
human+AI review often underperforms the best of either alone, and production data on
approval-prompt fatigue. The framework therefore treats *adding* a human gate as a decision
requiring justification, not a default act of diligence. See [`SPEC.md`](SPEC.md) §0 and
Appendix B for the full evidence base.

## How to cite

GitHub's **"Cite this repository"** button (top right of the repo page) generates APA/BibTeX
from [`CITATION.cff`](CITATION.cff). Or directly:

> Boord, A. (2026). *Agentic Workflow Governance (AWG)* (Version 0.4.0)
> [Framework specification]. https://github.com/Cadtastic/Agentic-Workflow-Governance

```bibtex
@misc{boord2026awg,
  author       = {Boord, Addam},
  title        = {Agentic Workflow Governance ({AWG})},
  year         = {2026},
  version      = {0.4.0},
  howpublished = {\url{https://github.com/Cadtastic/Agentic-Workflow-Governance}},
  note         = {Licensed under CC BY-SA 4.0}
}
```

## Contributing

AWG is built to be adopted and adapted. Issues and pull requests that sharpen the primitives,
tighten the schemas, or add worked examples are welcome. Substantive changes are tracked in
[`CHANGELOG.md`](CHANGELOG.md) and against the open design questions in [`SPEC.md`](SPEC.md) §10.

Because AWG is licensed under **CC BY-SA 4.0**, derivative frameworks must stay under the same
license — the commons stays open.

## License

Licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).
Full text in [`LICENSE`](LICENSE).

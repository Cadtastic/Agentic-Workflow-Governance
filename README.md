# Agentic Workflow Governance (AWG)

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

*A framework for designing, gating, and pinning accountability in workflows where AI agents take action.*

## What AWG is

Agentic Workflow Governance (AWG) is an organization-agnostic framework for designing the
stop-gates, handoffs, and accountability structures in business workflows that involve AI
agents. It targets the *agentic governance gap*: when an AI agent is framed as an
organizational actor, a human reviewer's perceived share of responsibility drops — and,
unlike with human delegation, no compensating moral-hazard suspicion arises to restore
oversight. AWG supplies the structural countermeasures: per-tier work classification, a
five-phase model, a library of ten atomic gate primitives, schema-as-contract handoffs, and
a single named human Process Owner per workflow.

AWG is **Layer 1** — the generic framework. Organization-specific operating models (who
performs each role, how work is audited) live in a separate Layer 2 document that adopters
write for themselves.

## What AWG is not

- Not a model-risk framework — use NIST AI RMF for evaluation, bias testing, lifecycle.
- Not a regulatory compliance framework — use ISO/IEC 42001 or jurisdiction-specific resources.
- Not a security review process, not a calibration framework, not a checklist.
- Not for L0 (suggest-only) systems — AWG applies at agency level **L1 and above**.

See [`SPEC.md`](SPEC.md) §9 for the full scope boundary.

## The specification

[`SPEC.md`](SPEC.md) is the canonical specification and always reflects the current state of
the framework (HEAD). Tagged releases are recorded in [`CHANGELOG.md`](CHANGELOG.md) and as
git tags; frozen historical drafts are preserved under [`spec-history/`](spec-history/).

## Motivation

AWG is the structural response to the oversight decline documented in:

> Wiles, E., Hsu, M., Bedard, J., & Kropp, M. (2026). [*Putting AI on the Org Chart:
> Evidence on Delegation and Oversight.*](https://www.emmawiles.com/storage/ai_employee.pdf)
> Boston University / BCG working paper.

## How to cite

**Plain text:**

> Boord, A. (2026). *Agentic Workflow Governance (AWG)* (Version 0.3.0)
> [Framework specification]. https://github.com/Cadtastic/Agentic-Workflow-Governance

**BibTeX:**

```bibtex
@misc{boord2026awg,
  author       = {Boord, Addam},
  title        = {Agentic Workflow Governance ({AWG})},
  year         = {2026},
  version      = {0.3.0},
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

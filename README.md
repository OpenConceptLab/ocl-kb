# OCL Knowledge Base

Consolidated institutional knowledge for the Open Concept Lab platform. This repository captures 7+ years of product development context — what was built, what's pending, key decisions, and open design questions — extracted from the analysis of 600+ issues in [OpenConceptLab/ocl_issues](https://github.com/OpenConceptLab/ocl_issues).

Most thematic knowledge bases were produced in March 2026 using the [Thematic Bucket Analysis Playbook](methodology/thematic-bucket-analysis-playbook.md) as part of the OCL issues cleanup initiative. The TBv3 specification library is an active, evolving requirements document.

## Contents

### Active Specification

| Directory | Description |
|-----------|-------------|
| [tbv3/](tbv3/) | **TermBrowser v3 Requirements Library** — 40-file hierarchical specification covering context, data objects, capabilities, workflows, UI surfaces, and architectural decisions. This is the active spec for TBv3 development. Includes a `bucket-analysis/` subdirectory with the original 262-issue analysis that seeded the library. |

### Thematic Knowledge Bases

Each KB was produced by analyzing a thematic bucket of issues. They capture executive summaries, completed work, open items, contributor maps, and timelines.

| Directory | Issues | Description |
|-----------|--------|-------------|
| [api2/](api2/) | 314 | **oclapi2 Backend** — Django REST API: bugs, import/export, search, collections, cascade, FHIR integration. 6 development eras from pre-launch through v3 hardening. |
| [fhir/](fhir/) | 59 | **FHIR Service** — HL7 FHIR-compliant interface layer: CodeSystem, ValueSet, ConceptMap operations, canonical URLs, validation. |
| [design/](design/) | 46 | **Design System & UX** — UI/UX specifications and design system work. 61% overlaps with TBv3. |
| [epic-meta/](epic-meta/) | 64 | **Epic/Meta Issues** — Umbrella tracking issues. Many are "zombie" epics from a 2021 organizational sprint. |
| [orphaned/](orphaned/) | 113 | **Orphaned (Unlabeled) Issues** — Issues with no labels or milestones. 82% are recent (2025-2026) active work that was never categorized. |
| [v3-unreviewed/](v3-unreviewed/) | 70 | **V3 Unreviewed** — V3-labeled issues not on the TBv3 project board: design specs, foundation epics, late-filed features. |
| [roadmap-2026/](roadmap-2026/) | 31 | **2026 Roadmap Priorities** — High-priority issues selected for the 2026 roadmap, organized by 11 themes. |

### Cross-Cutting Reports

| Directory | Description |
|-----------|-------------|
| [cross-cutting/](cross-cutting/) | Synthesis and evidence reports that span all buckets: historical backlog synthesis (510 issues), code evidence scanning, Phase 1 review outcomes. |

### Reference

| Directory | Description |
|-----------|-------------|
| [methodology/](methodology/) | The Thematic Bucket Analysis Playbook — the reusable 7-step process that produced these KBs. |
| [_process-artifacts/](_process-artifacts/) | Classification CSVs, project hierarchies, and raw archives from the analysis process. Kept for audit trail. |

## How to Use This KB

- **Working on TBv3 features?** Start with `tbv3/` — load `00_context/glossary.md` + relevant objects + capabilities + surfaces.
- **Understanding a platform area?** Pick the thematic KB (api2, fhir, design, etc.) and start with the executive summary section.
- **Looking for cross-bucket patterns?** Read `cross-cutting/historical-backlog-synthesis.md` for the 510-issue thematic synthesis.
- **Auditing a specific issue disposition?** Check `_process-artifacts/` for the classification CSVs.
- **Running a new bucket analysis?** Follow the playbook in `methodology/`.

## Provenance

This KB consolidates material from two sources:
- **ocl-workspace** — The working repo for the OCL issues cleanup initiative (March 2026)
- **ocl-kb (direct)** — The TBv3 spec library and roadmap-2026 bucket, developed directly in this repo

## The OCL Digital Public Good

This repository is not part of the [OCL Digital Public Good](https://github.com/OpenConceptLab/.github/blob/main/DIGITAL-PUBLIC-GOOD.md). That page lists the repositories that are.

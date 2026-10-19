# ServiceNow CAD Mastery Guide — Modules 14–25

A hands-on study and build guide for ServiceNow application development, written around scoped applications, integration, reporting, debugging, scale testing, the development lifecycle, and the Certified Application Developer exam domains.

## Scope

This repository currently contains **Modules 14–25 only**.

- **Modules 1–13 are not included.** They were never generated in this session and do not exist in the source material.
- **Modules 14–19 are marked `MISSING FROM HISTORY`.** Only summary fragments were recoverable; their full bodies, proof tables, code blocks, and design notes are not present.
- **Modules 20–25 are complete** in this repository. Module 20 (integration), 21 (reports and dashboards), 22 (debugging and performance), 23 (scale test), 24 (dev lifecycle), and 25 (CAD mapping and checklist) are assembled from the generated text.

Do not infer Modules 1–19 from this repository.

## CAD blueprint domains and weights

1. Designing and Creating an Application — 15%
2. Application User Interface — 20%
3. Security and Restricting Access — 20%
4. Application Automation — 20%
5. Working with External Data — 10%
6. Managing Applications — 15%

## Module 25 mapping summary

Module 25 maps every sub-topic (1A–6C) to module numbers and marks each as `Mapped`, `Partial`, or `Unmapped`.

- **Strongest coverage:** security and access control, external data via REST and import sets, and application lifecycle.
- **Fully unmapped:** mobile application versions, mobile client-side scripts, Document Feeds, and SOAP integration depth.
- **Partial:** app-fit assessment, module creation depth, automatic ACL generation, application properties, scheduled jobs, outbound email, classic homepages, and the Team Development product.
- The checklist contains 57 skills mapped to modules, with unmapped items marked explicitly.

## Audit summary

The Phase 4 audit in [`audit/phase4_audit_modules_14_to_25.md`](audit/phase4_audit_modules_14_to_25.md) is scoped to Modules 20–25.

- Source-bearing quotes checked: 11
- Matched after repair: 11
- Mismatch rate: 0.0%
- Code blocks: 29 of 29 marked and unindented
- Named-page claims without an inline URL: 84

The scoped quote and code-block pass marks are met. The guide is **not fully sourced**: 84 claims cite a named page without a URL, because the proof tables for Modules 20–22 were lost from history. Modules 1–13 and the full bodies of 14–19 were not audited because they do not exist in this session.

**Keep this repository private.** The audit is partial and the guide contains unreviewed, unexecuted lab code. Do not make it public unless the missing modules are supplied and the audit is rerun against a complete file.

## Folder structure

```text
servicenow-cad-mastery-guide/
├── README.md
├── NOTES.md
├── guide/
│   └── modules_14_to_25.md
└── audit/
    └── phase4_audit_modules_14_to_25.md
```

## How to use the guide

1. Read the scope note and `NOTES.md` first.
2. Work the modules on a sub-production PDI, never production.
3. Treat every code block marked `Written by me, not fetched; test it.` as unverified until you run it.
4. Use the “How to know it works” sections as the acceptance test for each lab.
5. Read Module 25 as a coverage map, not as an exam dump. The unmapped topics still need separate study.

## Source limits

- No PDI was available during authoring. No seed, index, report, clone, SDK, or integration command in this guide was executed.
- Performance numbers are not reported as measured results; Module 23 provides a blank measurement sheet and says so.
- Community thread titles and URLs were fetched. Dates were not consistently available and are recorded as `not extracted` in the audit.
- The `sdk-examples` folder list is quoted from the repository README, not from a live directory listing.

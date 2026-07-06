---
name: paper-reader
description: "PDF文献结构化精读与信息抽取技能。当用户提供PDF论文文件并需要精读拆解、提取核心信息、生成结构化阅读报告时触发。触发词包括但不限于：精读论文、读论文、拆解文献、论文解析、帮我读这篇论文、提炼论文核心、结构化阅读、批量读论文、论文精读报告。"
version: 2.0.0
author: 卓增明, refactored into static/dynamic layers
---

# 精读拆解 · 结构化信息抽取 — Router

This skill is split into two layers:

- A **static layer** under `static/` that holds versioned, reusable content fragments (core principles, the reading workflow, the output contract, and per-source-format extraction guidance).
- A **dynamic layer** (this file plus `manifest.yaml`) that detects the request's source format and loads only the fragments needed for the current job.

Do not try to apply the reading logic from memory or from this router. Always load fragments from disk as described below.

## Routing protocol

Follow these five steps every time the skill is invoked.

### 1. Load the manifest and the core layer

Read [manifest.yaml](manifest.yaml). It declares the `source_format` axis, the allowed values, and the file paths each value maps to.

Also read every file listed under `always_load`. These hold the core principles, the reading workflow, and the output contract that apply to every reading job.

### 2. Detect the source format

Decide the `source_format` value using the manifest's `detect:` hint and the user's input:

- `pdf-text` — selectable-text PDF. Default.
- `scanned-pdf` — image-only or OCR-required PDF.
- `doi-url` — a bare DOI, arXiv ID, or publisher URL that must be resolved first.
- `pasted-text` — pasted abstract, prose, or notes with no retrievable original file.

State the detected value in one short line to the user before processing, so they can correct you cheaply. A source may map to more than one value (for example a DOI that resolves to a PDF); load the resolution fragment first, then the fragment for the resolved artifact.

### 3. Load the matching fragment(s)

Read the file mapped for the detected `source_format`. Do **not** read every fragment in `static/`. Load only what step 2 selected.

### 4. Build the reader using the loaded material

Apply the loaded fragments in this priority order:

1. Core principles (`core/principles.md`) — 8-dimension extraction, 300-word cap, user-research anchoring.
2. Source-format fragment — how to extract text, figures, and tables for this input.
3. Reading workflow (`core/workflow.md`) — the four-step extraction-to-report process.
4. Output contract (`core/output-contract.md`) — required files, HTML template, and the pre-response verification checklist.

Load `references/` fragments on demand as needed per the manifest's `references.on_demand` table.

If the source is a DOI/URL that cannot be resolved, or the PDF is unreadable, still create a draft report and label missing content clearly. Do not switch to a generic summary.

### 5. Reach for references only when needed

The files under `references/` are deep references, not defaults. Open them on demand per the `references.on_demand` table in the manifest:

- Source block ID assignment and lightweight tracing → `references/source-tracing.md`.
- Extracting key tables/figures from the paper → `references/figure-extraction.md`.
- Judging fit level and mapping to user hypotheses → `references/fit-assessment.md`.
- Selecting quotable sentences and citation format → `references/quotable-sentences.md`.

## Why this split

- The static layer is versioned and reviewable. Adding a new source format is one new fragment plus one manifest line.
- The dynamic layer keeps each invocation cheap: only the fragment relevant to this input enters context.
- The router itself is short on purpose. Update fragments, not this file, when adding scope.

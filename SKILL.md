---
name: codex-word-skill
description: Read a Word document's content or make precise edits to its content and formatting (.docx). Use when the user wants to inspect, summarize, or extract text from a Word file, or when they ask to modify wording, restructure sections, adjust styles, tweak tables, fix spacing, or otherwise refine a document's content and visual layout. Do not use for creating spreadsheets, slides, or PDFs (use the spreadsheets, presentations, or pdf skills instead), and do not use for hand-drawn or illustrative artifacts that belong in a graphics tool.
---

# Codex Word Skill

A single entrypoint for two Word capabilities this environment already has: **reading** content and **precisely editing** both content and formatting.

## Capabilities

- **Read** a `.docx`: extract and return structured text (summaries, answers, targeted lookups).
- **Edit precisely**: change wording, restructure sections, adjust styles/headings, fix tables, spacing, headers, footers, and other layout details — then verify the result visually.

## Tool contract

This skill delegates to two existing skills. Do not duplicate their machinery; route to them.

- `.docx` content extraction and all document authoring → **`documents`** skill (the source of truth for `render_docx.py`, the `scripts/` helpers, and the render-verify loop).
- Choosing the right reader and discovering workspace runtimes → **`read-documents`** skill (routes `.docx` to the `documents` skill, and points at the `pdf` / `spreadsheets` / `presentations` skills for other formats).

When either skill's read path needs the underlying runtime, resolve it through `load_workspace_dependencies` and use the returned Node/Python paths — never system `node`/`python` or global packages.

## Golden path

1. **Read first.** For any non-trivial edit, extract the current content via the `documents`/`read-documents` read path so you edit with full context, not assumptions.
2. **Author the edit.** Use `python-docx` for paragraphs, runs, styles, tables, and headers/footers. For tracked changes, comments, hyperlinks, or fields, use the `documents` skill's OOXML patches.
3. **Render and verify.** Run `render_docx.py` to produce `page-<N>.png`, inspect every page at 100% zoom, and fix anything off. Repeat until flawless. Word's own view is not ground truth — the rendered PNGs are.
4. **Deliver the `.docx` only**, unless the user explicitly asks for intermediates.

## Precision editing

When the task is specifically about formatting accuracy, read [references/format-precision.md](references/format-precision.md) before authoring. It captures the non-obvious checks (cell padding, border consistency, run-vs-paragraph spacing, tracked-changes visibility) that separate a visually correct document from a hand-tweaked one.

## Final response

- Create/edit: cite the final `.docx` exactly once with a plain output citation and summarize representative changes.
- Q&A / read-only: cite the needed page(s) once each; do not re-export.
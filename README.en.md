# Codex Word Skill

A Codex skill that unifies two capabilities into a single entrypoint: **reading** Word documents and **precisely editing** their content and formatting.

## Capabilities

- **Read** a `.docx`: extract and return structured text (summaries, answers, targeted lookups).
- **Edit precisely**: change wording, restructure sections, adjust styles/headings, fix tables, spacing, headers, footers, and other layout details, then verify the result visually.
- **Built-in scripts and examples**: ships generic, self-contained Python tooling for render verification, auditing, and precise table editing.

## Structure

```text
codex-word-skill/
|-- SKILL.md                       Entry point: scope, tool contract, golden path
|-- agents/
|   `-- openai.yaml               UI metadata
|-- scripts/
|   |-- render_docx.py            Generic DOCX renderer (LibreOffice headless, page-<N>.png)
|   |-- word_audit.py             Generic document audit (heading hierarchy, numbering, direct overrides)
|   `-- table_geometry.py        Generic table geometry (tblW/tblGrid/tcW exact widths)
|-- examples/
|   |-- read_content.py           Dump paragraphs, tables, headers/footers, section geometry
|   |-- edit_content.py           Precise find-and-replace across runs and table cells
|   |-- edit_tables.py            Set content-derived column weights, cell margins, borders
|   `-- tracked_changes.py        Add a tracked replacement via raw OOXML patch
`-- references/
    |-- content-precision.md      Content-accuracy checklist (wording, terms, numbers, cross-refs)
    `-- format-precision.md       Formatting-accuracy checklist (spacing, borders, alignment, tables)
```

## How It Works

This skill is an orchestrator. It does not duplicate existing machinery; it routes to two skills already available in the environment:

- **Reading Word content**: routed through the `read-documents` skill to the `documents` skill.
- **Precise content and formatting edits**: handled by the `documents` skill, using `python-docx` for routine paragraphs/styles/tables and OOXML patches for tracked changes, comments, hyperlinks, fields, and other advanced constructs.

## Usage

Describe your Word task in natural language inside Codex, for example:

- "Read this docx and summarize it"
- "Make the chapter 3 headings bold"
- "Fix the column widths and borders in this table"

The skill activates automatically and follows the golden path: read for context, author the edit, render to PNG and inspect every page, then iterate until the document is flawless. Only the final `.docx` is delivered.

Scripts and examples can be run directly in a Python environment (use `load_workspace_dependencies` to resolve the runtime):

```bash
# Render verification
python scripts/render_docx.py input.docx --output_dir out/

# Pre-edit audit
python scripts/word_audit.py input.docx

# Precise table editing
python examples/edit_tables.py input.docx output.docx
```

## Scope

- Intended for `.docx` content reading and precise editing.
- Not for creating spreadsheets, slides, or PDFs (use the `spreadsheets`, `presentations`, or `pdf` skills instead).
- Not for hand-drawn or illustrative artifacts.

## Runtime

Resolves Node/Python runtimes through `load_workspace_dependencies`. No system-global dependencies required.

## License

MIT

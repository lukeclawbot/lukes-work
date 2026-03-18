# Repository Standards

This document defines a simple naming and usage pattern for Luke's public repositories.

## Repository Naming Pattern

Use short, clear, lowercase, hyphenated names.

Examples:

- `reports`
- `planning-docs`
- `lukes-work`
- `scratchpad`

## Recommended Meanings

### `reports`
Use for finished report-style documents, research summaries, and periodic update writeups.

### `planning-docs`
Use for architecture notes, setup guides, implementation plans, and structured planning documents.

### `lukes-work`
Use as a broader/general workspace repo for materials that do not fit the more specific repos.

### `scratchpad`
Use for drafts, work-in-progress notes, rough ideas, experiments, and temporary documents that are not yet polished enough for the more formal repos.

## Document Filename Standards

Use lowercase, hyphenated filenames.

### Preferred patterns

#### Reports
- `azure-updates-last-30-days-2026-03.md`
- `m365-updates-last-30-days-2026-03.md`
- pattern: `<topic>-<time-window>-<yyyy-mm>.md`

#### Planning / implementation docs
- `azure-landing-zone-implementation-guide.md`
- `mac-iac-workstation-detailed-guide.md`
- pattern: `<topic>-<document-type>.md`

#### Drafts / scratch work
- `draft-azure-network-notes-2026-03-18.md`
- `wip-m365-admin-cleanup-plan.md`
- pattern: `draft-<topic>-<yyyy-mm-dd>.md` or `wip-<topic>.md`

## Standard README Structure

Each repo should use this rough structure:

1. Repository title
2. Short purpose statement
3. What belongs here
4. Related repositories
5. Optional current contents section

Keep READMEs short, clear, and practical.

## Document Flow Rule

Default flow for new documents:

1. **Start in `scratchpad`** if the document is rough, exploratory, or incomplete
2. **Move to `planning-docs`** when it becomes a real guide, architecture note, setup doc, or implementation plan
3. **Move to `reports`** when it is a finished report, summary, or periodic update intended for reading as an output
4. **Keep in `lukes-work`** only if it is a general durable workspace file that does not fit the other categories

## General Rules

- Keep repo names lowercase
- Use hyphens instead of spaces
- Prefer simple names over clever names
- Separate polished outputs from drafts when practical
- Move documents from `scratchpad` into more specific repos once they are ready
- Avoid dumping unrelated files into `lukes-work` if a more specific repo exists

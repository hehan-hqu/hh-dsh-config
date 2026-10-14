# dsh-better-edit guidance

Each agent preset has its own guidance directory here: `<preset>/<section>.md`.
On first boot the plugin seeds the shipped presets (`standard`, `code`,
`minimal`, `cordis`) with the compiled defaults; existing files are never
overwritten, so your edits survive.

Each file is one tool section:

- `read.md` -> `tool:read`
- `edit.md` -> `tool:edit`
- `undo_last_edit.md` -> `tool:undo_last_edit`

## Customize

Edit the `<section>.md` file for the preset and section you want to change —
the file body IS the text the model reads for that section. Files are read once
per agent at session-start, so edits apply to new sessions.

An optional YAML front-matter fence places the section at a custom position in
the assembled system prompt:

    ---
    order: 150
    ---

    <section text>

A file without front-matter keeps the default order. A malformed fence (a
missing closing `---`, a non-integer `order`, an unknown key) makes the whole
file plain prose.

## Fallback

A preset with no directory here, or a missing section file, falls back to the
compiled defaults in the plugin bundle. To customize a preset that has no
seeded directory, copy a seeded one to its name.

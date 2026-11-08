# Prompts

One-off prompts worth keeping, to paste into a Claude Code session. Each file holds one prompt, verbatim, in a code
block, with a line on when to use it.

These are not skills. The plugin does not load this folder, so nothing here becomes a command, and the prompts are not
held to the sourcing rule in `CLAUDE.md`. A prompt that you run often enough to want as a command moves to
`skills/<name>/SKILL.md` and meets that rule first.

| Prompt | Use when |
|--------|----------|
| [`implicit-state-machines.md`](implicit-state-machines.md) | An app tracks status with loose flags and you want it made explicit. |

To add one: create `prompts/<kebab-case-name>.md` in the same shape and add a row to this table.

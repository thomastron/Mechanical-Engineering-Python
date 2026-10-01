# CLAUDE.md

@AGENTS.md

`AGENTS.md` is the single source of truth for this repository's conventions. It is imported
above so Claude Code and other agents follow the same rules. Put shared guidance there, not here.
Only Claude-specific notes belong in this file.

## Claude-specific notes

- **Read on demand, not up front.** The checklist is ~2,300 lines (~140 KB). Find the node you
  need with `grep -n '^## ' MECHANICAL-DESIGN-CHECKLIST__20260816-0939.md` and read that range.
  The traveller (~600 lines) can usually be read whole.
- **Notebooks:** use `NotebookEdit` for cell edits. After editing, re-execute with the
  `nbconvert` command in AGENTS.md so committed outputs match the code.
- **Reporting results:** give the governing number, its unit, the check it was compared against,
  and the ledger rows it depends on. If a §9.6 audit row fails, lead with that — do not bury it
  under a margin-of-safety figure.

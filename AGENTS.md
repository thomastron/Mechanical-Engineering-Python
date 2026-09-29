# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex, Cursor, Copilot, etc.) working in this
repository. Human-facing overview lives in [`README.md`](README.md).

## What this repository is

A proof-of-concept **mechanical engineering analysis workflow**, not a software package. There is
no library, no build, and no test suite. The deliverables are engineering analyses whose
assumptions are written down and checked against the answer.

The intended use: load these files as context, point an agent at a textbook or design problem, and
have it produce a defensible analysis — framed with the checklist, computed in a notebook, recorded
in a traveller.

| File | Role | Edit policy |
|---|---|---|
| `MECHANICAL-DESIGN-CHECKLIST__20260816-0939.md` | Reference decision tree (§0–§10 + Appendices A–E). The method. | **Reference — do not edit** unless explicitly asked. |
| `MECHANICAL-DESIGN-TRAVELLER__20260816-0931.md` | Blank per-part design record template. | **Template — never fill in place.** Copy it. |
| `Cantilever_Beam_Analysis_Triangular_Load.ipynb` | Worked example: 9 m steel cantilever, triangular load, PlaneSections vs closed form. | Example — pattern to imitate. |
| `README.md` | Human overview plus the "Python for the Design Desk" package map and gotchas. | Edit when adding files or packages. |
| `assets/` | Images referenced by notebooks. `assets/temp` is a placeholder. | — |

The `__YYYYMMDD-HHMM` suffix on the two Markdown documents **is their revision identifier**. The
traveller hard-links to the checklist by filename and heading anchor (`...CHECKLIST__20260816-0939.md#24-triage-gate-...`).
Renaming the checklist or rewording a numbered heading breaks every `guide §N.N` link in the
traveller. If a revision is requested, create a new timestamped file and update the traveller's
links in the same change.

## Workflow for solving a problem

Follow the README's four stages. The non-negotiable parts:

1. **Frame before computing.** Walk the checklist from §1. Respect node types: **FORK** = pick
   one, **ALL** = every item applies, **GATE** = screen and route, **EXIT** = leave the document.
   Treating an ALL node as a FORK is how load cases get dropped.
2. **Run the triage gate (§2.4).** If any of the nine specialized regimes applies (fatigue,
   fracture, creep, contact/wear, joints, impact, corrosion/SCC, plate/shell buckling, vibration)
   or the material is anisotropic/nonlinear, **route it out and say so** — record it in the §10.3
   route-out register. Do not quietly apply static linear-elastic methods outside their scope.
3. **Keep an Assumption Ledger** (checklist Appendix A). Every fork adds a row with its
   justifying number (L/h, σ vs S_y, δ vs t/2, frequency ratio, …).
4. **Compute in a notebook** (see conventions below).
5. **Audit the ledger against the computed answer (§9.6).** In particular:
   - Computed σ above S_y from a linear-elastic model means the result is **invalid**, not
     "fails with FoS < 1". Report that the model is violated and return to §3.2.
   - Computed δ above ~½ thickness, or rotations above ~5°, likewise invalidates small-deflection
     results.
6. **Record in a traveller copy** named `TRAVELLER_<part>__<YYYYMMDD-HHMM>.md`. Leave nothing
   blank: `—` means "deliberately not applicable"; an empty box is ambiguous.

Do not state a margin of safety, pass/fail verdict, or sign-off without the independent check
(§9.1) and ledger audit (§9.6) having been done. Never fill in the Sign-off block's human names,
dates, or approvals — leave those to the engineer.

## Notebook conventions

Imitate `Cantilever_Beam_Analysis_Triangular_Load.ipynb`:

- **State the problem, units, and sign convention in the first markdown cell.** Default to SI
  (N, m, N·m, Pa). Comment every input with its unit: `L = 9.0  # span [m]`.
- **Every numerical result gets an independent check** — closed form, a second package, or hand
  calc — compared with `math.isclose(..., rel_tol=...)` and printed as OK/FAIL. A notebook
  without a check is not finished.
- End with a **summary table** of the governing quantities.
- Document package API traps inline where they bite (the example's "API notes" section is the
  model).
- Commit notebooks **with outputs** so the results are readable on GitHub without running them.

### Library traps already found (see README "Gotchas" for the full list)

- **planesections 1.4.2** (solver backend is now PyNiteFEA): use `ps.PyNiteAnalyzer2D`, not
  the removed `ps.OpenSeesAnalyzer2D`. `ps.SectionBasic2D` is broken — use `ps.SectionRectangle`.
  `getSFD()`/`getBMD()` return each node twice; take `[1::2]`. `ps.getVertDisp()` returns
  `(disp, x)` — displacement first.
- **anaStruct** forces are exact with internal hinges; **hinged deflections are not**.
- **handcalcs**: no underscores or `^` in comments inside `%%render` cells (KaTeX in VS Code
  fails); define short pint units (`ksi`) or it prints canonical names; keep `%%render` cells to
  plain assignments.
- Use `scipy.linalg.eigh`, never `eig`, on stress tensors and K/M pairs.

## Environment and commands

No dependency file is committed. Minimal environment for the example notebook (verified on
Python 3.11; the notebook's own metadata says 3.13):

```bash
python -m venv .venv && source .venv/bin/activate
pip install "planesections==1.4.2" matplotlib nbconvert ipykernel
```

Other packages from the README map (`anastruct`, `sympy`, `scipy`, `pint`, `handcalcs`,
`uncertainties`, `sectionproperties`, `PyNiteFEA`, …) are installed only when a notebook needs them.

Execute a notebook headless — this is the closest thing to a test run:

```bash
MPLBACKEND=Agg jupyter nbconvert --to notebook --execute <notebook>.ipynb --inplace
```

It must finish without errors and every closed-form check must print `OK`
(the example ends with `all checks passed`). Do not commit `.venv/` or `.ipynb_checkpoints/`.

## Known inconsistencies (do not propagate)

- README §2 refers to `Beam_bending_shear_displacement_PlaneSections.ipynb`, and the example
  notebook refers to a separate "anaStruct notebook". Neither is in the repo. The only notebook is
  `Cantilever_Beam_Analysis_Triangular_Load.ipynb`, and it uses PlaneSections only.
- README "Under-used" examples cite `Shigley chapter 2 and 3.ipynb`, which is also not in the repo.

When referencing files, check that they exist.

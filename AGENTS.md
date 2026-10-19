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
| `Design_Desk_Smoke_Test.ipynb` | Environment smoke test: every pinned analysis package against a known answer, installed versions against the pins. Raises if any check fails. | Re-run after any change to `requirements.txt`; commit with outputs. |
| `README.md` | Human overview plus the "Python for the Design Desk" package map and gotchas. | Edit when adding files or packages. |
| `assets/` | Images referenced by notebooks and the README. `assets/temp` is a placeholder. | — |
| `LICENSE` | MIT, for the repository's own documents and code. Third-party images keep their own licenses (see the README's License section). | Don't change without being asked. |

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
- **CALFEM** (`calfem-python`, imported as `calfem`) numbers DOFs from 1 in `edof` and in the `bc`
  list for `solveq`. A 0-based list runs without error and fixes the wrong DOFs, so check the
  reactions against statics. Its `eigen` calls `eig`; use `eigh` instead (next item).
- Use `scipy.linalg.eigh`, never `eig`, on stress tensors and K/M pairs. When porting MATLAB,
  `eig(K)` becomes `eigh`: MATLAB and Octave switch to the symmetric solver themselves, SciPy doesn't.
- **scikit-fem** (imported as `skfem`): `lame_parameters(E, nu)` gives the 3D λ, μ, which makes a
  2D model plane strain (9% stiff for a thin plate); use `plane_stress(E, nu)` for plane stress.
  `plane_stress` returns λ, μ from 12.0.2 but an equivalent (E, ν) up to 12.0.1, so check the pin
  before copying older examples. Straight-edged elements on a curved boundary cap convergence at
  O(h²) whatever the element order; run the mesh study on the governing stress.
- **PathSim** defaults to a fixed-step 2nd-order solver (`SSPRK22`, dt = 0.01 s). Pass an adaptive
  solver (e.g. `Solver=RKDP54`) with `tolerance_lte_rel`/`tolerance_lte_abs`.
  `PID(f_max=...)` is in rad/s, not Hz. Its FMU blocks moved to `pathsim_fmi`.
- **OpenModelica** co-simulation FMUs integrate with explicit Euler, one step per communication
  interval, unless exported with `--fmiFlags=s:cvode`. Prefer model exchange (the importing tool
  integrates), and pass `relative_tolerance` to FMPy's `simulate_fmu`.
- **SimuPy** is not in `requirements.txt` (its last release fails on NumPy 2). If you meet it, its
  default `nsteps=500` silently truncates the result at a tight `rtol`; check `res.t[-1]`.
- **python-control** `step_info` uses its own coarse time grid and under-reports overshoot; pass a
  fine `T`. `forced_response` needs evenly spaced times.

## Environment and commands

`requirements.txt` pins every Python package in the README map, tested together on Python 3.11
(the notebook's own metadata says 3.13):

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

On Linux, `openseespy` installs but fails to import without the system BLAS/LAPACK libraries
(`libblas.so.3: cannot open shared object file`). Fix with `apt install libblas3 liblapack3`.
Likewise `gmsh`, which EasyFEA and CALFEM use for meshing, needs `apt install libglu1-mesa libxft2`
(`libGLU.so.1` / `libXft.so.2: cannot open shared object file`).
GXBeam is Julia, not Python, and is not in the file. Neither is FEniCSx, which has no pip
wheels (conda-forge `fenics-dolfinx` or the `dolfinx/dolfinx` Docker image).
`oct2py` installs from the file but needs GNU Octave itself (`apt install octave`). OpenModelica is
a standalone program and is not in the file either; `OMPython` is, and needs OpenModelica's `omc`
(locally, or in the `openmodelica/openmodelica` Docker image run with `--user`, since `omc` refuses
to serve ZeroMQ as root). SimuPy is compared in the README but deliberately not pinned.

Every change to `requirements.txt`, a new pin or a version bump, ends the same way: a fresh venv
from the file, `pip install -r requirements.txt`, `pip check`, and every notebook re-executed
headless, `Design_Desk_Smoke_Test.ipynb` included.

Execute a notebook headless — this is the closest thing to a test run:

```bash
MPLBACKEND=Agg jupyter nbconvert --to notebook --execute <notebook>.ipynb --inplace
```

It must finish without errors and every closed-form check must print `OK`
(the example ends with `all checks passed`). Do not commit `.venv/` or `.ipynb_checkpoints/`.

`Design_Desk_Smoke_Test.ipynb` is the environment-wide version: every pinned analysis package
against a known answer, plus installed versions against the pins. Its last cell raises if any
check fails, so `nbconvert` exits non-zero. Checks that need GNU Octave or OpenModelica report
`SKIP`, which is not a pass: say so when you report a run. Its OpenModelica checks use a local
`omc`, or one serving ZeroMQ at `OMC_PORT`, with `OM_WORKDIR` shared at the same path when `omc`
runs in Docker.

## Adding a package to the map

A package earns a row in the README map and a pin in `requirements.txt` only when all of these
hold. Record the evidence in its README section's **Checked:** line.

1. **It does a job no current row does.** Name the row it adds, or the row it splits, and say
   what it does that the existing entry can't. Popularity, polish and a good gallery are not
   reasons.
2. **It installs alongside every existing pin.** Add an exact `==` pin, build a fresh venv, and
   run `pip check`. Then import it and run it: `pip check` reads only declared metadata, so a
   package that declares `numpy>=1.11` but breaks on NumPy 2 passes it (SimuPy did). A package
   whose own dependencies pin below ours (`numpy<2`, as pathsim-vehicle does) can't join.
3. **It reproduces a known answer, in a run you did.** Compare against a closed form, a series
   solution or an established package with `math.isclose`, at a tolerance the method should meet,
   not one loosened until it passes. Write the value, the reference and the error in the README.
   Use a shared benchmark when one fits (the 2 m cantilever, the m-c-k system, the Lamé cylinder)
   so the result also cross-checks the other tools, and add the same check to
   `Design_Desk_Smoke_Test.ipynb`, with the package's import name in its `IMPORTS` table.
4. **It works on the stack we pin.** Its current release must support the pinned NumPy, SciPy and
   SymPy. A fix that exists only on an unreleased branch doesn't count.
5. **Its license is recorded,** and every image taken from its documentation is credited under
   the image with that license.
6. **The traps found on the way are written down:** in the README Gotchas, and in short form
   under "Library traps already found" above.

**How a package section reads.** The heading is the package name alone, with its links on the
next line, so the map's anchors stay short (`#scikit-fem`, not `#scikit-fem-docs-gallery-repo`).
Then **Use for:** in a sentence or two, **Where it fits:** against its neighbours in the map, and
**Checked:** with the numbers. Link the package's docs rather than restating its feature list.
Put a snippet longer than about 15 lines, or a table only some readers need, in a `<details>`
block. Images must show what the package computes; no logos or banners. Every new package also
gets a linked row in the map table, in its group.

Two exceptions:

- **Not pip-installable** (standalone programs, Julia packages, conda-only builds): it may have a
  row, but say so in the `requirements.txt` header and the README Setup section, and mark its
  section **Not checked here** until someone runs it. Its Python bridge (oct2py, OMPython) is
  pinned like any other package, and its smoke-test check skips when the program is missing.
- **Compared, not pinned:** a package that fails rule 2 or 4 may still be described when the
  comparison helps a reader choose (SimuPy against PathSim). Say plainly why it isn't pinned,
  with the evidence, and keep it out of the map table.

## Known inconsistencies (do not propagate)

- The example notebook mentions an anaStruct model of the same beam (marked "not in this
  repo"). It is not in the repo, and the notebook uses PlaneSections only. Its checks are
  against the closed form, not another package. Don't cite the anaStruct comparison as checked.

When referencing files, check that they exist.

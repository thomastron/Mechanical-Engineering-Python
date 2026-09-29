# Welcome to the Mechanical Engineering Seed Repository

This repository serves as a proof-of-concept for a rigorous, modern mechanical engineering workflow. It bridges the gap between structured decision-making, documented assumptions, and Python-based computational analysis. Python packages are introduced in this visual guide.

> [!Note:]
> This repo contains a single example notebook. Load these files in as AI context and point it at a textbook problem!

The materials here are designed to prevent the classic engineering trap of silently inheriting flawed assumptions, providing a clear path from problem framing to final sign-off.

## 1. The Design & Analysis Framework
The repository contains a two-part documentation system designed to separate reference material from live project work.
* **The Reference Guide (`MECHANICAL-DESIGN-CHECKLIST`):** A dense, navigable decision tree covering the 80% common case of mechanical design: static loading, linear-elastic response, and standard strength/deflection limits. Rather than attempting to cover everything poorly, it deliberately detects and routes out specialized regimes like fatigue, fracture, and creep for separate analysis. Its core engine is the Assumption Ledger, which forces analysts to explicitly record the idealizations (e.g., Euler-Bernoulli vs. Timoshenko, linear-elasticity) that govern their mathematical models.
* **The Project Record (`MECHANICAL-DESIGN-TRAVELLER`):** A blank, copyable template that serves as the official design record for a specific part. It strips out the explanatory text of the main checklist, leaving only the operational checkboxes, the decision trail, the Assumption Ledger audit, and the final sign-off blocks. It is designed to be highly reviewable, requiring explicit documentation of the governing failure mode and margin of safety.

## 2. The Computational Sandbox
* **The Python FEA Template (`Beam_bending_shear_displacement_PlaneSections.ipynb`):** A practical demonstration of using open-source Python packages (`anaStruct`, `PlaneSections`, and `PyNiteFEA`) to solve structural problems.
* **The Test Case:** The notebook models a 9-meter steel cantilever beam (300mm x 200mm section) subjected to a linearly varying load. It computes continuous shear, moment, and deflection diagrams, extracting exact nodal results. This serves as seed material for engineers looking to automate routine structural equivalents without losing visibility into the underlying physics. It also offers comparisons of the results between different packages. 

## 3. Standard Workflow
To test this proof-of-concept in a live scenario:
1. **Frame & Model:** Use the Reference Guide to define the problem criticality, load paths, and appropriate idealizations.
2. **Compute:** Utilize the concepts in a Jupyter Notebook to build a tractable mathematical model and extract internal resultants.
3. **Document:** Record the decisions, bounds, and limits in a fresh copy of the Traveller.
4. **Audit:** Run the final numbers back through the Assumption Ledger in the Traveller to ensure the computed stresses and deflections do not violate the initial linear-elastic or small-deflection assumptions.


# Python for the Design Desk - Visual Map

Organized by what you are trying to do, not by package taxonomy.

| The job                                           | Reach for                                |
| ------------------------------------------------- | ---------------------------------------- |
| Document a calculation so a reviewer can audit it | `handcalcs`                              |
| Keep units straight across kpsi and SI            | `pint` (`forallpeople` if SI-only)       |
| Carry tolerance and scatter through a formula     | `uncertainties`, `scipy.stats`           |
| Symbolic derivation, algebra, calculus            | `sympy`                                  |
| Beam by singularity functions, symbolically       | `sympy.physics.continuum_mechanics.beam` |
| Beam or frame with an **internal hinge**          | `sympy…beam.apply_rotation_hinge`        |
| Multibody dynamics — Kane's method, Lagrange      | `sympy.physics.mechanics` (PyDy's core)  |
| Quick beam diagrams, numeric                      | `symbeam`, `planesections`               |
| Torsion in a shaft (diagram, internal T)          | `Pynite` (`'MX'` loads, `torque_array`)  |
| 2D frame and truss analysis                       | `anastruct`, `PyNite`                    |
| Cross-section properties, arbitrary shapes        | `sectionproperties`                      |
| Root-finding, curve fits, ODEs, eigenvalues       | `scipy`                                  |
| Fatigue cycle counting from a load history        | `wetb.fatigue_tools`                     |
| Large-deflection / geometrically exact beams      | GXBeam.jl                                |

## GXbeam [docs](https://flow.byu.edu/GXBeam.jl/stable/)
**Use for:** Simulating large-deflection, highly flexible structures (like wind turbine blades or high-aspect-ratio wings) using geometrically exact beam theory.

<table>
  <tr>
    <td width="50%"><img src="https://flow.byu.edu/GXBeam.jl/stable/assets/dynamic-joined-wing-simulation.gif" alt="Dynamic joined wing simulation"></td>
    <td width="50%"><img src="https://flow.byu.edu/GXBeam.jl/stable/assets/wind-turbine-blade-simulation.gif" alt="Wind turbine blade simulation"></td>
  </tr>
</table>

## Section Properties [docs](https://sectionproperties.readthedocs.io/)
**Use for:** Calculating complex cross-sectional properties (area, moments of inertia, torsion constant, shear center) for arbitrary custom extrusions or built-up shapes.

<table>
  <tr>
    <td colspan="2" align="center"><img src="https://sectionproperties.readthedocs.io/en/stable/_images/logo-dark-mode.png" alt="Section Properties Logo" width="300"></td>
  </tr>
  <tr>
    <td width="50%"><img src="https://sectionproperties.readthedocs.io/en/stable/_images/examples_geometry_geometry_coordinates_14_0.svg" alt="Geometry coordinates"></td>
    <td width="50%"><img src="https://sectionproperties.readthedocs.io/en/stable/_images/examples_advanced_advanced_plot_10_0.svg" alt="Advanced plot"></td>
  </tr>
</table>

## Pynite 3D truss FEA [docs](https://github.com/jwock82/pynite)
**Use for:** Lightweight 3D finite element analysis to rapidly evaluate internal shear, bending moments, and deflections in frames, trusses, and structural beam assemblies.

<table>
  <tr>
    <td colspan="2" align="center"><img src="https://pynite.readthedocs.io/en/latest/_images/TransparentLogo.png" alt="Pynite Logo" width="200"></td>
  </tr>
  <tr>
    <td width="50%"><img src="https://www.engineeringskills.com/_next/image?url=%2Fimages%2Fposts%2Fa-pynite-crash-course-open-source-finite-element-modelling-for-structural-engineers%2Fimg1.jpg&w=640&q=75" alt="Pynite Crash Course"></td>
    <td width="50%"><a href="https://www.linkedin.com/posts/connorferster_opensource-python-structuralengineering-activity-7038535825235607552-_3t_"><img src="https://media.licdn.com/dms/image/v2/C5622AQHbyXUDAE5_PQ/feedshare-shrink_800/feedshare-shrink_800/0/1678117583415?e=2147483647&v=beta&t=nd5-E6mBDJgmUwEk2VeR07TSsQflVsMY8aNzfprwFXs" alt="Pynite 3D visualization (animated) — Connor Ferster on LinkedIn"></a></td>
  </tr>
</table>

## Anastruct
**Use for:** Quick 2D structural analysis of complex frames and trusses to map bending moments, shear forces, and axial loads.

<table>
  <tr>
    <td width="50%"><img src="https://anastruct.readthedocs.io/en/latest/_images/tower_bridge_struct.png" alt="Tower bridge structure"></td>
    <td width="50%"><img src="https://anastruct.readthedocs.io/en/latest/_images/tower_bridge_displa.png" alt="Tower bridge displacement"></td>
  </tr>
  <tr>
    <td colspan="2" align="center"><img src="https://anastruct.readthedocs.io/en/latest/_images/heart.png" alt="Heart truss" width="50%"></td>
  </tr>
</table>

¹ `anastruct` handles internal hinges natively and its **forces** are exact, but its **hinged deflections are not** — see the gotchas below.

## PlaneSections [docs](https://github.com/cslotboom/planesections)
**Use for:** Fast 1D beam analysis specifically tailored for plotting standard shear force and bending moment diagrams (SFD/BMD) for straightforward load cases.

<table>
  <tr>
    <td colspan="2" align="center"><img src="https://github.com/cslotboom/planesections/raw/main/doc/img/Beam-Image-2.png" alt="Beam Image"></td>
  </tr>
  <tr>
    <td width="50%"><img src="https://github.com/cslotboom/planesections/raw/main/doc/img/Beam-Image-2-SFD.png" alt="SFD"></td>
    <td width="50%"><img src="https://github.com/cslotboom/planesections/raw/main/doc/img/Beam-Image-2-BMD.png" alt="BMD"></td>
  </tr>
</table>

## SymBeam [docs](https://github.com/amcc1996/symbeam)
**Use for:** Extracting exact symbolic, analytical solutions (algebraic equations) for beam bending, shear, and internal moments. A pedagogical package for beam bending.

<div align="center">
  <img src="https://gist.githubusercontent.com/amcc1996/ca430a30adc69b1c1616cd949196466a/raw/00c07608d21e2f692e91032e82094f2f0d7d8931/example_readme.svg" alt="SymBeam Example">
</div>

## handcalcs [documentation](https://github.com/connorferster/handcalcs)
**Use for:** Automatically converting Python variables and math into rendered LaTeX step-by-step equations, producing readable, auditable "hand calculations" for design reports.

<div align="center">
  <img src="https://github.com/connorferster/handcalcs/raw/main/docs/images/basic_demo1.gif" alt="Handcalcs basic demo">
</div>

## uncertainties [docsA](https://github.com/connorferster/handcalcs) [docsB](https://github.com/lmfit/uncertainties)
**Use for:** Propagating dimensional tolerances, material property scatter, and measurement errors through complex engineering formulas automatically.

The following pics are actually from `uncertainty-toolbox`. The following plots are a few of the [visualizations](https://github.com/uncertainty-toolbox/uncertainty-toolbox/blob/main/uncertainty_toolbox/viz.py) provided by Uncertainty Toolbox. See [this example](https://github.com/uncertainty-toolbox/uncertainty-toolbox/blob/main/examples/viz_readme_figures.py) for code to reproduce these plots.

<table>
  <tr>
    <td width="33%"><strong>Overconfident</strong><br><em>too little uncertainty</em><br><img src="https://raw.githubusercontent.com/uncertainty-toolbox/uncertainty-toolbox/master/docs/images/row_1.svg" alt="Overconfident"></td>
    <td width="33%"><strong>Underconfident</strong><br><em>too much uncertainty</em><br><img src="https://raw.githubusercontent.com/uncertainty-toolbox/uncertainty-toolbox/master/docs/images/row_2.svg" alt="Underconfident"></td>
    <td width="33%"><strong>Well calibrated</strong><br><br><img src="https://raw.githubusercontent.com/uncertainty-toolbox/uncertainty-toolbox/master/docs/images/row_3.svg" alt="Well calibrated"></td>
  </tr>
</table>

## OpenSeesPy [documentation](https://openseespydoc.readthedocs.io/en/stable/index.html)
**Use for:** Advanced nonlinear structural and geotechnical finite element simulations, highly applicable to seismic response, plastic hinge formation, and complex material models.

<table>
  <tr>
    <td width="50%"><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_model.png" alt="3D Model"></td>
    <td width="50%"><img src="https://openseespydoc.readthedocs.io/en/stable/_images/Model_Plot3D.png" alt="Model Plot 3D"></td>
  </tr>
  <tr>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/ModeShape_5_Plot3D.png" alt="Mode Shape 5"></td>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_extruded_members.png" alt="Extruded Members"></td>
  </tr>
  <tr>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_defo.png" alt="Deformation"></td>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_Mz.png" alt="Mz Moment"></td>
  </tr>
  <tr>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_My.png" alt="My Moment"></td>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_Vz.png" alt="Vz Shear"></td>
  </tr>
  <tr>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_Vy.png" alt="Vy Shear"></td>
    <td><img src="https://openseespydoc.readthedocs.io/en/stable/_images/demo_cantilever_3el_3d_T.png" alt="Torsion"></td>
  </tr>
</table>

## SymPi (SymPy)
**Use for:** Symbolic algebraic derivations, calculus for analytical beam theory, and constructing equations of motion via Kane's or Lagrange's methods for rigid body multibody dynamics.

<div align="center">
  <img src="https://github.com/sympy/sympy/raw/master/banner.svg" alt="SymPy Banner" width="400">
</div>

[`sympy.physics.mechanics`](https://docs.sympy.org/latest/modules/physics/mechanics/api/index.html#module-sympy.physics.mechanics) provides a joints framework. This system consists of two parts. The first are the [`joints`](https://docs.sympy.org/latest/modules/physics/mechanics/api/joint.html#module-sympy.physics.mechanics.joint) themselves, which are used to create connections between [`bodies`](https://docs.sympy.org/latest/modules/physics/mechanics/api/part_bod.html#sympy.physics.mechanics.rigidbody.RigidBody). The second part is the [`System`](https://docs.sympy.org/latest/modules/physics/mechanics/api/system.html#sympy.physics.mechanics.system.System), which is used to form the equations of motion. Both of these parts are doing what we can call “book-keeping”: keeping track of the relationships between [`bodies`](https://docs.sympy.org/latest/modules/physics/mechanics/api/part_bod.html#sympy.physics.mechanics.rigidbody.RigidBody). 
- [Vector & ReferenceFrame](https://docs.sympy.org/latest/explanation/modules/physics/vector/vectors/vectors.html)     
- [Vector](https://docs.sympy.org/latest/explanation/modules/physics/vector/vectors/vectors.html#vector)     
- [Vector Algebra](https://docs.sympy.org/latest/explanation/modules/physics/vector/vectors/vectors.html#vector-algebra)     
- [Vector Calculus](https://docs.sympy.org/latest/explanation/modules/physics/vector/vectors/vectors.html#vector-calculus)     
- [Using Vectors and Reference Frames](https://docs.sympy.org/latest/explanation/modules/physics/vector/vectors/vectors.html#using-vectors-and-reference-frames) 
- [Vector: Kinematics](https://docs.sympy.org/latest/explanation/modules/physics/vector/kinematics/kinematics.html)     
- [Introduction to Kinematics](https://docs.sympy.org/latest/explanation/modules/physics/vector/kinematics/kinematics.html#introduction-to-kinematics)     
- [Kinematics in physics.vector](https://docs.sympy.org/latest/explanation/modules/physics/vector/kinematics/kinematics.html#kinematics-in-physics-vector)

<table>
  <tr>
    <td width="50%"><img src="https://static.lwn.net/images/2020/sympy-3Dgraph.png" alt="Sympy 3D Graph"></td>
    <td width="50%"><img src="https://docs.sympy.org/latest/_images/ipythonnotebook.png" alt="IPython Notebook"></td>
  </tr>
  <tr>
    <td><img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*KDXg_d_umU3cTqZQenE-1w.gif" alt="Mechanics GIF"></td>
    <td><img src="https://i.ytimg.com/vi/1SQ4-k6wr3Y/hqdefault.jpg" alt="YouTube Default"></td>
  </tr>
  <tr>
    <td><img src="https://docs.sympy.org/latest/_images/biomechanics-steerer.svg" alt="Biomechanics Steerer"></td>
    <td><img src="https://docs.sympy.org/latest/_images/biomechanical-model-example-35.png" alt="Biomechanical Model Example"></td>
  </tr>
</table>

## Numpy
MATLAB alternative core. NumPy is the fundamental package for scientific computing in Python. It is a Python library that provides a multidimensional array object, various derived objects (such as masked arrays and matrices), and an assortment of routines for fast operations on arrays, including mathematical, logical, shape manipulation, sorting, selecting, I/O, discrete Fourier transforms, basic linear algebra, basic statistical operations, random simulation and much more.

## SciPy
SciPy is a collection of mathematical algorithms and convenience functions built on [NumPy](https://numpy.org/). It adds significant power to Python by providing the user with high-level commands and classes for manipulating and visualizing data. See note at bottom about under-use.

## Pandas
Import and write data from/to Excel, csv, json. Structure, access and manipulate data efficiently

## Matplotlib
create plots and graphs

---

# Gotchas

- `handcalcs` renders pint's **canonical** unit name and ignores the registry formatter, so
  `12 * u.kpsi` prints as `kilopound_force_per_square_inch`. Define short units instead:
  `u.define("ksi = 1000 * pound_force / inch ** 2")` → renders as `ksi`.
- Keep `%%render` cells to plain assignments. `.to()` and `print()` inside a rendered cell will not
  render — do those in a normal cell.
- **No underscores or `^` in comments.** handcalcs wraps every comment in `\textrm{...}`, and an
  underscore inside LaTeX text mode is illegal. MathJax (classic Jupyter, Colab) tolerates it;
  **KaTeX, which VS Code uses, throws** `ParseError: Expected 'EOF', got '_'` and the cell fails to
  render entirely. Write `# bore radius`, not `# r_i is the bore`. Subscripts belong in the variable
  names, where handcalcs turns them into real math — never in the prose beside them.
- `forallpeople` 3.0.0 is installed but ships **no unit environments** in this build, and is
  SI-centric. Use `pint` for imperial work.

---

# For Future Development and Learning
## Under-used (under-understood)

### `sympy.physics.mechanics`

Kane's method, Lagrange's method, `dynamicsymbols`, reference-frame algebra for multibody systems.
This **is** the core of PyDy — PyDy is largely this module plus numerical codegen and visualisation,
so there is nothing to install to get started.

---

### SciPy already does several interesting things

#### `optimize.brentq` — solve an implicit design equation

Size a member for a target factor of safety instead of guess-and-check. Bracketed, so it cannot
wander off — the right default for monotonic design relations. Same family: `root` for systems,
`fsolve` for legacy code.

```
shaft for n = 2.0 under M = 12 000 lbf·in, Sy = 50 kpsi
→ d = 1.6973 in   (check: n = 2.000)
```

#### `optimize.curve_fit` / `least_squares` — fit a material model to test data

Basquin S–N, Ramberg–Osgood, Paris law, creep correlations. `curve_fit` also returns the covariance
matrix, which feeds straight into `uncertainties` for confidence bands on the fitted curve.

```
4 points of S–N data → S = 2.262e5 · N^(−0.1274)
S at 5×10⁵ cycles = 42 485 psi
```

#### `integrate.quad` — Castigliano without closed-form integration

When ∂U/∂P is ugly — tapered sections, curved beams, combined loading — integrate numerically. Also
the honest route to section properties of shapes with no formula.

```
cantilever δ = ∫ M(∂M/∂P)/EI dx
→ 3.072000 in    exact PL³/3EI = 3.072000 in
```

#### `integrate.solve_ivp` — transient and vibration response

SDOF and MDOF time histories, shock response, rotor run-up. `dense_output=True` gives a continuous
interpolant you can sample anywhere afterwards. Feed the result to `wetb` for rainflow counting and
you have a fatigue chain end to end.

```
m=1, c=2, k=100, x₀=0.01
→ ωn = 10.000 rad/s, ζ = 0.1000, peak |x| = 0.01000 m
```

#### `linalg.eigh` / `eigvalsh` — principal stresses in 3D

Symmetric-matrix eigensolvers give principal stresses **and** their directions in one call. The same
routine gives natural frequencies and mode shapes from `K` and `M`. Use `eigh`, never the general
`eig`, on a stress tensor — it guarantees real eigenvalues and orthogonal directions.

```
σx=80, τxy=50 MPa   (Shigley Ex 3-4, cell 27 of "Shigley chapter 2 and 3.ipynb")
→ principals [104.03, 0, −24.03] MPa   τmax = 64.03 MPa
```

#### `stats.weibull_min` / `norm` / `lognorm` — life scatter and reliability

Weibull is the standard fatigue-life and bearing-life model; `.ppf()` gives B10 / L10 directly. The
natural continuation of the Shigley Ch. 1 statistical work already done with `norm`.

```
shape 2.3, scale 1.2e6 → B10 life = 4.511e+05 cycles
```

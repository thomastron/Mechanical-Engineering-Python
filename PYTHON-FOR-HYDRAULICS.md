---
title: Python for Hydraulics
created: 2026-09-12
type: reference
status: living
tags: [hydraulics, python, tooling, pipe-networks, open-channel, fluid-power, cfd]
---

# Python for Hydraulics

A roadmap of what exists in the Python hydraulics ecosystem, **what each tool actually does**, and
which one to reach for. Written for picking this up after months away — every entry says what the
package can compute, not just what it is.

Companion to [Python for the Design Desk](README.md#python-for-the-design-desk), the package map in
this repository's README, which covers stress, beams and calculation records.
Same idea, different domain.

> **Status: none of this is installed or tested here.** None of these packages is pinned in
> `requirements.txt` (PyVista arrives there only as an EasyFEA dependency), whose stack is tested on
> Python 3.11. Every version and wheel note below was
> checked against PyPI on **2026-09-12** for Python 3.13 on Windows; nothing has been import-tested
> yet. Install commands are at the bottom.

The field splits into two traditions that barely share vocabulary: **water resources** hydraulics
(EPANET, HEC-RAS, SWMM — pipe networks, rivers, floods) and **fluid power** hydraulics (cylinders,
spool valves, accumulators — the ME sense). Python is very strong in the first and thin in the
second. Both are below; §G is the honest report on the second.

---

## First fork — which hydraulics is this?

Pick the row, then read only that section.

| If the problem is…                                      | You are in            | Go to |
| ------------------------------------------------------- | --------------------- | ----- |
| A **closed, pressurised network** — mains, pumps, tanks  | Water distribution    | [§A](#a-pressurised-pipe-networks) |
| **One pipe, one valve, one pump** — sizing a component   | Component hydraulics  | [§B](#b-component-hydraulics--the-fluids-stack) |
| **Water with a free surface** — channel, river, flood    | Open channel / 2D     | [§C](#c-open-channel-rivers-floodplains) |
| **Runoff, sewers, detention** — storm and sanitary       | Urban drainage        | [§D](#d-stormwater-and-sewer) |
| **Flow through soil or rock**                            | Groundwater           | [§E](#e-groundwater) |
| **Resolving the velocity field itself**                  | CFD                   | [§F](#f-free-surface-and-cfd) |
| **Cylinders, spool valves, hydraulic circuits**           | Fluid power           | [§G](#g-fluid-power--the-me-reading) |

---

## The map, by job

Organised by what you are trying to do.

| The job                                                   | Reach for                    | Note |
| --------------------------------------------------------- | ---------------------------- | ---- |
| Solve a water distribution network, script scenarios       | `wntr`                       | EPANET 2.2 inside, plus its own solver |
| Drive EPANET function by function, real-time control       | `epyt`                       | Thin, faithful toolkit wrapper |
| Read and write an EPANET `.inp` as Python objects          | `oopnet`                     | Stale since 2023 — check first |
| Publication-grade water network maps                       | `viswaternet`                | Sits on top of `wntr` |
| Darcy friction factor, any correlation                     | `fluids.friction`            | 74 functions, Colebrook through Churchill |
| K value for a fitting, bend, valve, contraction            | `fluids.fittings`            | Crane TP-410M, Hooper 2K, Darby 3K, Miller |
| Size a control valve                                       | `fluids.control_valve`       | IEC 60534-2-1, -8-3, -8-4 |
| Size an orifice, venturi, nozzle, cone or wedge meter      | `fluids.flow_meter`          | ISO 5167 parts 1–6, ISO 11583 |
| Size a relief valve                                        | `fluids.safety_valve`        | API 520 / 521 / 526 |
| Manning's equation, weirs, open channel flow               | `fluids.open_flow`           | Closed form, not a solver |
| Pump and motor sizing, affinity laws, NPSH                 | `fluids.pump`                | Includes standard motor data |
| Tank volume or wetted area at a liquid level               | `fluids.geometry`            | Every head type and orientation |
| Water, steam and refrigerant properties                    | `CoolProp`, `thermo`         | **CoolProp has no Python 3.13 wheel** |
| Run HEC-RAS from Python, read its HDF results              | `ras-commander`              | Needs HEC-RAS 6.x installed |
| Post-process HEC-RAS or SRH-2D into VTK                    | `pyHMT2D`                    | Model-agnostic 2D post-processing |
| Solve 2D shallow water yourself (dam break, tsunami)       | `anuga`                      | Finite volume, unstructured mesh |
| Overland flow on a raster DEM, landscape evolution         | `landlab`                    | Component kit, not one model |
| Build a flood model from global data                       | `hydromt` + `hydromt_sfincs` | Deltares; data-to-model pipeline |
| Run SWMM step by step with control logic in Python         | `pyswmm`                     | Read and write state mid-simulation |
| Parse, edit and map a SWMM model                           | `swmmio`, `swmm-api`         | `swmmio` draws, `swmm-api` edits |
| Groundwater flow and transport (MODFLOW 6)                 | `flopy`                      | The standard, no real competitor |
| Free-surface CFD or SPH                                    | see [§F](#f-free-surface-and-cfd) | Heavy — last resort |
| Hydraulic cylinder and spool-valve circuits                | thin — see [§G](#g-fluid-power--the-me-reading) | Python has no good answer |

---

## A. Pressurised pipe networks

The EPANET family. All of these read the same `.inp` file, so you can move between them.

### `wntr` — Water Network Tool for Resilience

**v1.5.0 · US EPA · cp313 Windows wheel · [docs](https://usepa.github.io/WNTR/)**

The most capable of the set, and the default choice unless you specifically need raw toolkit calls.
It is EPANET plus a scenario and resilience layer.

**What it does**

- **Two simulators.** `EpanetSimulator` calls the EPANET 2.2 binary for speed and exact agreement
  with the reference implementation. `WNTRSimulator` is a pure-Python solver that can **pause and
  restart mid-run**, which is what lets you change the network between time steps.
- **Pressure-dependent demand** as well as classic demand-driven analysis. This is the one that
  matters — see the first gotcha below.
- **Damage and disruption**: pipe leaks, tank, valve and pump damage, power outages, contaminant
  injection, fire-flow demands.
- **Response modelling**: leak repair, pipe replacement, power restoration, backup generation.
- **Resilience metrics** — topographic, hydraulic, water quality and economic — plus **fragility
  curves** for probabilistic component failure.
- **Geospatial**: converts to GeoPandas, intersects the network with hazard rasters and polygons.
- Network graphics, animation, and interactive Plotly output.

<img src="https://usepa.github.io/WNTR/_images/plot_basic_network.png" alt="WNTR network plot" width="560">

<img src="https://usepa.github.io/WNTR/_images/plot_valve_segment_attributes.png" alt="WNTR valve segmentation" width="560">

*Valve segmentation — which part of the network isolates when a given valve closes. Tedious by hand,
nearly free once the model is in Python.*

<img src="https://usepa.github.io/WNTR/_images/fragility_curve.png" alt="WNTR fragility curve" width="420"> <img src="https://usepa.github.io/WNTR/_images/plot_pump_curve.png" alt="WNTR pump curve" width="420">

<img src="https://usepa.github.io/WNTR/_images/plot_plotly_network.png" alt="WNTR interactive network" width="500">

```python
import wntr
wn = wntr.network.WaterNetworkModel('Net3.inp')

wn.options.hydraulic.demand_model = 'PDD'      # pressure-dependent
wn.options.hydraulic.required_pressure = 20    # m

pipe = wn.get_link('123')
pipe.add_leak(wn, area=0.05, start_time=2*3600)

results = wntr.sim.WNTRSimulator(wn).run_sim()
results.node['pressure']                        # DataFrame: time x node
```

### `epyt` — EPANET-Python Toolkit

**v2.3.5.2 · KIOS / OpenWaterAnalytics · pure Python · [repo](https://github.com/OpenWaterAnalytics/EPyT)**

**A faithful wrapper over the EPANET programmer's toolkit, one Python method per toolkit function.
Where WNTR gives you an opinionated model object, EPyT gives you the API surface.**

**What it does**

- Full access to the EPANET 2.2 toolkit: every `getX` and `setX` call, node and link indexing.
- **Step-by-step hydraulic and quality analysis** — open the solver, step, read results, change a
  setting, step again. This is the hook for control algorithms, leak detection research and
  reinforcement learning.
- Loads and edits `.inp` files, plots networks and time series directly.
- Ships the EPANET binaries, so there is nothing to install separately.

Reach for EPyT when writing a control loop or a research algorithm against EPANET, and WNTR when you
want scenarios, resilience and geospatial work.

<img src="https://user-images.githubusercontent.com/2945956/154067349-3aed266f-3a23-4573-8b93-db0b4f224964.png" alt="EPyT network" width="600">

### `viswaternet` — network visualisation

**v2.2.3 · pure Python · [repo](https://github.com/tylertrimble/viswaternet)**

One job, done well: turning a WNTR model into a readable map. Colour and size nodes or links by any
attribute, overlay pump, valve and tank symbols, discretise continuous data into legend classes, and
export animations. Worth having the moment a network stops fitting on one screen.

<img src="https://raw.githubusercontent.com/tylertrimble/viswaternet/main/logo/readme1.png" alt="VisWaterNet example" width="460"> <img src="https://raw.githubusercontent.com/tylertrimble/viswaternet/main/logo/readme2.png" alt="VisWaterNet example" width="460">

### `oopnet` — object-oriented EPANET

**v0.6.5 · last release 2023-10 · pure Python**

Models the `.inp` file as a Python object tree, runs EPANET underneath, returns pandas objects.
Clean design, but **nothing has shipped since October 2023** — check that it still imports before
building on it. WNTR covers most of the same ground and is actively maintained.

---

## B. Component hydraulics — the `fluids` stack

### `fluids`

**v1.3.1 · Caleb Bell · pure Python · [docs](https://fluids.readthedocs.io/)**

The single most useful install on this page for mechanical work. It is not a network solver — it is the correlation
library that sits underneath one. Pure Python, correlations cited to source.

**What it covers, by submodule**

| Submodule                            | What you get                                                                                                                                                                                          |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `friction`                           | 74 functions. Colebrook (exact, via Lambert W), Churchill, Serghides, Clamond and the rest; pipe roughness tables; Crane fully-turbulent factors                                                      |
| `fittings`                           | Loss coefficients for bends, tees, contractions, diffusers, entrances and exits; **Crane TP-410M**, **Hooper 2K**, **Darby 3K**, Miller, Rennels; all standard valve types; `Cv`/`Kv`/`K` conversions |
| `control_valve`                      | Sizing and rating per **IEC 60534-2-1**, with choked flow, cavitation and noise per **-8-3** and **-8-4**                                                                                             |
| `flow_meter`                         | Orifice plates, nozzles, venturis, cone and wedge meters per **ISO 5167-1 … -6**, plus **ISO 11583** and **ISO 15377**                                                                                |
| `safety_valve`                       | Relief valve sizing per **API 520** and **API 521**, standard orifice areas per **API 526**                                                                                                           |
| `open_flow`                          | Manning's equation, weir and flume discharge — closed form, one section at a time                                                                                                                     |
| `pump`                               | Pump and motor sizing, affinity laws, NPSH, specific speed, standard motor sizes and efficiencies                                                                                                     |
| `geometry`                           | Tank volume and wetted area at any liquid level, for every common head type and orientation                                                                                                           |
| `compressible`                       | Isothermal and isentropic pipe flow, Fanno, compressor sizing                                                                                                                                         |
| `two_phase`, `two_phase_voidage`     | Two-phase pressure drop and void fraction correlations                                                                                                                                                |
| `drag`, `particle_size_distribution` | Particle drag, terminal velocity, PSD fitting                                                                                                                                                         |
| `atmosphere`                         | US Standard Atmosphere 1976, wind speed profiles, solar position                                                                                                                                      |
| `units`                              | **The whole library wrapped in `pint`** — see below                                                                                                                                                   |
| `numba`, `vectorized`                | JIT-compiled and array-broadcast variants of every function                                                                                                                                           |

<img src="https://fluids.readthedocs.io/_images/bend_rounded.png" alt="fluids rounded bend" width="290"> <img src="https://fluids.readthedocs.io/_images/bend_miter.png" alt="fluids mitre bend" width="290"> <img src="https://fluids.readthedocs.io/_images/contraction_conical.png" alt="fluids conical contraction" width="290">

*The docs draw the geometry each correlation applies to, which is the part that is easy to get wrong
from a bare formula.*

<img src="https://fluids.readthedocs.io/_images/ft_Crane_plot.png" alt="fluids Crane friction factor" width="470">

**`fluids.units` connects straight to work already done here.** This repository already uses `pint` for
unit safety in hand calculations. `fluids.units` exposes every function with pint quantities in and
out, so a friction factor or a valve size participates in the same dimensional checking as the rest
of a calculation record, and pairs with `handcalcs` the same way.

```python
from fluids.units import *
from pint import _DEFAULT_REGISTRY as u

Re = Reynolds(V=3.5*u.m/u.s, D=0.2*u.m, rho=997.1*u.kg/u.m**3, mu=1e-3*u.Pa*u.s)
fd = friction_factor(Re=Re, eD=0.0001*u.dimensionless)
K  = K_gate_valve_Crane(D1=0.2*u.m, D2=0.2*u.m, angle=0*u.degrees)
```

Note the registry: `fluids.units` binds to pint's **default** registry. Mixing it with a separately
constructed `UnitRegistry()` raises a cross-registry error. Use
`from pint import _DEFAULT_REGISTRY as u` everywhere, and add the short-unit definitions from the
`handcalcs` gotcha in the [README](README.md#handcalcs-units-and-pdf-export) to that same registry.

### `ht`, `thermo`, `CoolProp` — the rest of the stack

`ht` 1.2.0 (heat transfer correlations) and `thermo` 0.6.1 (phase equilibria, transport properties)
are the same author and the same design. `CoolProp` 8.0.0 is the reference thermophysical property
library, but it **has no Python 3.13 wheel on Windows** — it stops at cp312. On 3.13, either use
`thermo`, or make a separate 3.12 environment for property lookups. For plain water at ordinary
temperatures, `fluids` and `thermo` are enough on their own.

---

## C. Open channel, rivers, floodplains

Two routes: **drive an established model** (HEC-RAS), or **solve the shallow water equations
yourself**. Driving an established model is almost always right when the answer has to be defensible.

### `ras-commander` — HEC-RAS automation

**v0.99.2 · pure Python · needs HEC-RAS installed (Windows) · [repo](https://github.com/gpt-cmdr/ras-commander)**

**What it does**

- **Executes plans** single, sequential or **in parallel** across cores. This is the reason to script
  HEC-RAS at all — batch calibration runs that take a day by hand finish over lunch.
- **Edits project files programmatically**: clone plans, swap geometry and flow files, change
  computation intervals, set Manning's *n*, edit unsteady boundary tables.
- **Reads HDF results directly** — 2D mesh depths, velocities and water surface elevations, cross
  sections, structures (bridges, culverts, gates), pipe networks, pump stations, breach output. No
  more exporting CSV from the GUI.
- Targets **HEC-RAS 6.2+**, and reaches older versions (3.1 through 6.6) through the RasControl COM
  interface.

### `pyHMT2D` — model-agnostic 2D post-processing

**v2.0.0 · [repo](https://github.com/psu-efd/pyHMT2D)**

Controls and post-processes 2D hydraulic models — **HEC-RAS 2D, SRH-2D and Backwater-1D** — and
converts results to **VTK** for ParaView or PyVista. Its point is comparison: putting two different
models' output into one format so you can difference them. Also handles terrain and Manning's *n*
fields, and calibration loops.

<img src="https://raw.githubusercontent.com/psu-efd/pyHMT2D/main/logo/pyhmt2d_logo_vel_color_vector.png" alt="pyHMT2D velocity field" width="540">

### `anuga` — solve the shallow water equations

**v4.0.0 · Geoscience Australia and ANU · cp313 Windows wheel**

A finite-volume solver for the 2D shallow water equations on an unstructured triangular mesh. It
handles **wetting and drying**, which is what makes it usable for dam break, tsunami run-up and flood
inundation rather than just steady channel flow. Parallel via MPI. You build the domain, boundary
conditions and friction field in Python and it integrates in time.

Reach for it when there is no HEC-RAS model to drive and the physics is genuinely 2D and transient.

### `landlab` — component kit for surface processes

**v2.11.0 · CSDMS · cp313 Windows wheel · [docs](https://landlab.csdms.io/)**

Not a hydraulic model — a **framework of interchangeable components** on a shared grid (raster, hex,
Voronoi). Relevant components include overland flow (kinematic and diffusive wave), flow routing and
accumulation, depression filling, infiltration, and erosion and deposition. Built for landscape
evolution over long timescales. Useful here when the question is *where does water go on this
terrain*, not *what is the depth at this cross-section*.

### `hydromt` + `hydromt_sfincs` — data to model, automatically

**hydromt_sfincs v1.2.2 · Deltares · [docs](https://deltares.github.io/hydromt_sfincs/)**

HydroMT builds model inputs from raw geospatial data. The SFINCS plugin targets **SFINCS**, a
reduced-physics flood solver that runs orders of magnitude faster than a full shallow water code,
which is what makes large ensembles and probabilistic flood mapping tractable. Sibling plugins exist
for Wflow (hydrology) and Delft3D FM.

---

## D. Stormwater and sewer

All three wrap or parse **EPA SWMM 5**, which handles rainfall-runoff, sewer and storm networks,
surcharging, and water quality.

### `pyswmm`

**v2.1.0 · pure Python · [docs](https://pyswmm.github.io/pyswmm/)**

The one that matters. SWMM's own engine is a black box that runs start to finish; PySWMM opens it.

**What it does**

- **Step-by-step simulation** with full read and write access to state between steps — node depths,
  link flows, pump settings, orifice openings, subcatchment runoff.
- Which means **real-time control logic written in Python** instead of SWMM's rule syntax. Anything
  from a level trigger to a model-predictive controller.
- Reads and writes SWMM input objects and output time series directly.

```python
from pyswmm import Simulation, Nodes, Links

with Simulation('model.inp') as sim:
    j1 = Nodes(sim)['J1']
    pump = Links(sim)['PUMP1']
    for step in sim:                       # step by step
        pump.target_setting = 1.0 if j1.depth > 2.0 else 0.0
```

<img src="https://raw.githubusercontent.com/pyswmm/pyswmm/master/docs/source/_static/type-logo-black.png" alt="pyswmm" width="270">

### `swmmio` and `swmm-api`

`swmmio` 0.8.5 parses a SWMM model into pandas DataFrames, edits it, and **draws it** — including
annotated flood maps and animations. `swmm-api` 0.4.74 is the stronger read, write and edit library
for the `.inp` and `.rpt` files themselves. Use `swmm-api` to build models, `swmmio` to look at them,
`pyswmm` to run them.

<img src="https://raw.githubusercontent.com/aerispaha/swmmio/master/docs/_static/img/flooded_anno_example.png" alt="swmmio flood map" width="560">

---

## E. Groundwater

### `flopy`

**v3.11.0 · USGS · pure Python · [docs](https://flopy.readthedocs.io/)**

The Python interface to **MODFLOW 6** and the whole USGS modelling family (MODFLOW-2005, MT3D,
SEAWAT, MODPATH, PEST). Builds, runs and post-processes models; writes and reads every package file;
plots plan views, cross sections and particle tracks; exports to VTK, shapefile and NetCDF. There is
no serious alternative, and `get-modflow` fetches the compiled executables for you.

<img src="https://raw.githubusercontent.com/modflowpy/flopy/master/examples/images/quickstart.png" alt="FloPy quickstart" width="500">

---

## F. Free surface and CFD

Only when the depth-averaged assumption fails — a hydraulic jump in detail, air entrainment,
sloshing, flow around a structure. **All of these are heavy installs and multi-day learning curves.**

| Package | What it is | When it earns the cost |
|---|---|---|
| `PyFoam`, `fluidfoam` | Pre- and post-processing for **OpenFOAM** | You are already committed to OpenFOAM; `interFoam` is the free-surface VOF solver |
| `pysph` | **Smoothed particle hydrodynamics**, meshless | Violent free surface — dam break, wave impact, sloshing. **Source-only on PyPI (1.0b2); expect to compile** |
| `Thetis` | 3D coastal and estuarine, on **Firedrake** | Stratified estuary or coastal circulation |
| `FEniCSx`, `Firedrake` | You write the weak form | A custom formulation; painful on Windows without WSL or Docker |
| `SU2` | Compressible CFD with a Python API | Aerodynamics rather than hydraulics |
| `pyvista` | Visualisation of any of the above | Install regardless — it reads the VTK that `pyHMT2D` and `flopy` write |

---

## G. Fluid power — the ME reading

If "hydraulics" means cylinders, spool valves, accumulators and hose, **Python has no good answer**.
Better to know that now than to go looking.

- **`hydraulic`** (PyPI, v0.2.0) is a 1D circuit solver for pressure, flow and temperature. Last
  release **2023-04**, small, thinly documented. Look before committing.
- **`fluids`** (§B) is genuinely the best option for component-level work: orifice and valve flow,
  pressure drop through lines and fittings, pump and motor sizing. It gives you the pieces and you
  assemble the circuit yourself — `scipy.optimize.root` for the flow balance,
  `scipy.integrate.solve_ivp` for the transient. For most fluid-power sizing that is enough.
- **The real tools are Modelica.** `OpenHydraulics` and the Modelica Standard Library's fluid-power
  components model cylinders, motors, pumps, valves and lines properly. Drive them from Python with
  **OMPython** (OpenModelica) if the circuit is complex enough to need it.
- Commercial: Simscape Fluids, Amesim, Automation Studio. Out of scope here.

**Suggested route:** `fluids` plus `scipy` for sizing and simple transients, Modelica when you need a
real circuit model. The [SciPy recipes in the README](README.md#scipy-does-more-than-most-people-use-it-for) apply directly — `solve_ivp`,
`root` and `brentq` are the same three calls that do the work here.

---

## Install feasibility

Checked against PyPI on 2026-09-12 for Python **3.13.0** on Windows. This is where the surprises are.

| Package | Version | Wheel situation on py3.13 Windows |
|---|---|---|
| `epyt` | 2.3.5.2 | Pure Python — trivial, ships EPANET binaries |
| `fluids` | 1.3.1 | Pure Python — trivial, nothing beyond numpy and scipy |
| `pyswmm` | 2.1.0 | Pure Python — trivial, ships the SWMM engine |
| `swmmio` | 0.8.5 | Pure Python |
| `swmm-api` | 0.4.74 | Pure Python |
| `flopy` | 3.11.0 | Pure Python; run `get-modflow` afterwards for the executables |
| `ras-commander` | 0.99.2 | Pure Python — but **needs HEC-RAS 6.x installed** to do anything |
| `viswaternet` | 2.2.3 | Pure Python |
| `oopnet` | 0.6.5 | Pure Python — **unmaintained since 2023-10** |
| `wntr` | 1.5.0 | **cp313 win_amd64 wheel exists** — installs clean |
| `anuga` | 4.0.0 | **cp313 win_amd64 wheel exists** — pins `python < 3.15` |
| `landlab` | 2.11.0 | **cp313 win_amd64 wheel exists** |
| `pyHMT2D` | 2.0.0 | Pins `python < 3.14` — fine here |
| `CoolProp` | 8.0.0 | **No cp313 wheel, stops at cp312.** Use `thermo`, or a separate 3.12 env |
| `pysph` | 1.0b2 | **Source distribution only.** Needs a compiler; expect a fight on Windows |

The `openseespy` lesson from the [README's setup notes](README.md#4-setup) applies: a wheel that installs is not the
same as a wheel that imports. Import-test each one and record the result here.

---

## Gotchas — expected, not yet confirmed here

Flagged from the packages' own documentation. **None has been reproduced yet.** Confirm
before trusting, and date the entry when you do.

- **Demand-driven analysis lies under low pressure.** EPANET's default model delivers full demand
  whatever the pressure, so a broken-pipe scenario returns negative pressures *and* satisfied demand.
  Set `demand_model = 'PDD'` in WNTR before any outage or fire-flow work. This is the most common
  error in scripted water network analysis.
- **EPANET unit systems are set in the `.inp` file, not in Python.** `FLOWUNITS` (GPM, LPS, CMH and
  the rest) drives whether everything else is US customary or SI, and the Hazen-Williams roughness
  coefficient silently changes meaning with it. Read the `.inp` header before interpreting a number.
- **`fluids.units` binds to pint's default registry.** Use `from pint import _DEFAULT_REGISTRY as u`,
  not a fresh `UnitRegistry()`, or every call raises a cross-registry error.
- **Hazen-Williams is not Darcy-Weisbach.** EPANET offers H-W, D-W and Chezy-Manning. H-W is valid
  only for water near ambient temperature over a limited velocity range. If the model came from
  someone else, check which is set.
- **`ras-commander` needs HEC-RAS actually installed**, and parallel execution needs enough licences
  and cores. It automates the engine behind the GUI; it does not replace it.
- **SWMM's own control rules and a PySWMM step loop can both be active**, and then they fight. Strip
  the `[CONTROLS]` section from the `.inp` when control logic moves into Python.
- **`anuga` and `landlab` pin their Python versions** (`<3.15` and `>=3.11`). Fine now; will bite at
  the next interpreter upgrade.
- **Manning's *n* is dimensional.** Its numeric value carries a hidden unit, and the 1.486 conversion
  factor for US customary units is applied inconsistently across tools. Check each tool's convention
  before comparing results.

---

## Commands

Run these in a virtual environment of their own (activate it with `.venv\Scripts\activate` on
Windows, `source .venv/bin/activate` elsewhere), not the one built from `requirements.txt`: none of
these packages has been checked against its pins.

```bash
# Core — light, pure Python, no compiler. Start here.
python -m pip install fluids
```

```bash
# Water distribution
python -m pip install wntr epyt viswaternet
```

```bash
# Stormwater and sewer
python -m pip install pyswmm swmmio swmm-api
```

```bash
# Groundwater, then fetch the MODFLOW executables
python -m pip install flopy
```

```bash
# HEC-RAS automation — requires HEC-RAS 6.x installed separately
python -m pip install ras-commander pyHMT2D
```

```bash
# 2D free-surface solvers and visualisation
python -m pip install anuga landlab pyvista
```

```bash
# Heat transfer and properties. CoolProp has no Python 3.13 wheel; install it in a 3.12 env
python -m pip install ht thermo
```

Smallest useful first step: install `fluids`, work one Crane TP-410M fitting problem in it, and check
the K value by hand. That is half an hour, and it makes the rest of this page concrete.

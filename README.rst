.. image:: images_shared/gsoc_banner.png
    :target: https://summerofcode.withgoogle.com/programs/2026/projects/uGyoXWw9
    :alt: Google Summer of Code

|

.. image:: images_shared/gprMax_logo_small.png
    :target: http://www.gprmax.com
    :alt: gprMax

|

.. image:: https://github.com/sahibjotsingh05/gprMax/actions/workflows/tests.yml/badge.svg?branch=gsoc26-unit-testing
    :target: https://github.com/sahibjotsingh05/gprMax/actions/workflows/tests.yml
    :alt: Unit tests

.. image:: https://readthedocs.org/projects/gprmax/badge/?version=devel
    :target: http://docs.gprmax.com/en/latest/?badge=devel
    :alt: Documentation Status


**********************************************
gprMax × Google Summer of Code 2026
**********************************************

**Building a Comprehensive Test Suite**

This fork is the working repository for a Google Summer of Code 2026 project with
`gprMax <http://www.gprmax.com>`__, an open source Finite-Difference Time-Domain
(FDTD) electromagnetic solver. The project built an automated unit test suite for
the gprMax core: **2,678 test functions across 71 files covering 16 subsystems**,
expanding to roughly 3,500 collected tests that run in under twenty seconds on
Linux, Windows and macOS.

The work was delivered as **12 reviewed pull requests** on this fork and merged
into the upstream project in
`gprMax/gprMax#750 <https://github.com/gprMax/gprMax/pull/750>`__. Acceptance into
the program is documented in `acceptance.pdf <acceptance.pdf>`__, the official
letter from the Google Summer of Code Program Lead.

..  list-table::
    :widths: 30 70
    :header-rows: 0

    * - **Program**
      - Google Summer of Code 2026
    * - **Organisation**
      - gprMax
    * - **Project**
      - Project 3 — Building a Comprehensive Test Suite (175 hours)
    * - **Project page**
      - `summerofcode.withgoogle.com <https://summerofcode.withgoogle.com/programs/2026/projects/uGyoXWw9>`__
    * - **Contributor**
      - Sahibjot Singh (`@sahibjotsingh05 <https://github.com/sahibjotsingh05>`__)
    * - **Accepted**
      - 30 April 2026 — see `acceptance.pdf <acceptance.pdf>`__
    * - **Coding period**
      - 25 May 2026 – 24 August 2026
    * - **Integration branch**
      - ``gsoc26-unit-testing`` (this branch), cut from upstream ``devel``
    * - **Upstream pull request**
      - `gprMax/gprMax#750 <https://github.com/gprMax/gprMax/pull/750>`__ — opened 2 June 2026, **merged 16 August 2026**
    * - **Status**
      - Complete and merged upstream


About Google Summer of Code
===========================

`Google Summer of Code <https://summerofcode.withgoogle.com/>`__ (GSoC) is a
global program, run by Google since 2005, that brings new contributors into open
source software development. Open source organisations apply to take part and
publish a list of project ideas; prospective contributors write a detailed
proposal against one of those ideas, and accepted contributors then spend the
summer working on it, paid by Google and mentored by the organisation's
maintainers. The program runs through community bonding, a coding period, and
midterm and final evaluations, with project sizes of 90, 175 or 350 hours.

This project was accepted on **30 April 2026** and ran from **25 May to
24 August 2026** as a 175-hour project with gprMax. The acceptance letter is
included in this repository as `acceptance.pdf <acceptance.pdf>`__.

GSoC matters because it solves a problem that is hard to solve any other way.
Open source projects — particularly scientific ones maintained by small academic
teams — accumulate work that is important but never urgent: test coverage,
documentation, packaging, performance. Meanwhile, contributors who want to work
on real software at real scale rarely get a route in, because large codebases are
intimidating and unpaid work is a luxury. GSoC pairs the two, with mentorship
attached, so that the contributor learns how a production codebase is actually
maintained and the project gets sustained, reviewed work that outlives the summer.


The Project
===========

gprMax solves Maxwell's equations in 3D using the FDTD method. It is one of the
few open source FDTD solvers that accelerates its update kernels across a range
of hardware, and the codebase reflects that: performance-critical routines are
written in `Cython <http://cython.org>`__ with OpenMP, and there are separate
GPU backends for `NVIDIA CUDA <https://developer.nvidia.com/cuda-zone>`__,
`OpenCL <https://www.khronos.org/api/opencl>`__ and
`Apple Metal <https://developer.apple.com/metal/>`__, plus MPI support for
multi-node simulations.

The idea published on the
`gprMax GSoC ideas page <https://github.com/gprMax/GSoC/blob/main/project-ideas-2026.md>`__
identified the gap directly: gprMax lacked comprehensive automated testing, which
made it hard to identify bugs and to verify that code changes behaved correctly.
The stated deliverables were to write unit tests for core gprMax functions, to
create validation tests checking that physics calculations are correct, and to
make sure the tests run on Windows, Linux and macOS. The expected outcome was
*"a working test suite with tests covering the most important parts of gprMax,
making it safer and easier for developers to improve the code."*

Testing a numerical solver is harder than testing ordinary application code, and
that shaped the whole project:

* **The output is a field, not a value.** A regression rarely raises an
  exception — it shifts a number in the fourth decimal place, and the simulation
  still completes and still looks plausible.
* **Four backends must agree.** The same physics is implemented separately in
  Cython/OpenMP, CUDA, OpenCL and Metal. Divergence between them is invisible
  without tests, and CI machines have no GPU.
* **Some of the code does not exist until build time.** The dispersive material
  kernels are generated from a Jinja2 template into ``.pyx`` during ``setup.py``,
  and the device kernels are assembled from templates at runtime.
* **A full model run is slow.** Tests that build and run a real model take
  minutes, which is far too slow to be the only feedback a contributor gets.

Proposal in brief
-----------------

The proposal was to build a fast, isolated unit test layer underneath the
existing model-level tests, working bottom-up through the codebase one subsystem
at a time:

#. **Isolate rather than simulate.** Drive each unit directly with lightweight
   test doubles for the grid, materials and configuration, so that tests exercise
   one function's behaviour and boundaries instead of paying for a full FDTD run.
#. **One subsystem per pull request.** Each PR takes a single domain — waveforms,
   materials, PML, subgrids — from zero to thorough coverage, so that every PR is
   independently reviewable and independently useful.
#. **Assert against theory, not against the code.** Where a closed-form reference
   exists (the CFL stability condition, the Debye relaxation formula, the
   derivative of a Gaussian at its peak), the test asserts against the textbook
   formula rather than against the current implementation's output.
#. **Pin known bugs explicitly.** Where the code was found to be wrong, the
   behaviour is captured in a dedicated test with a docstring explaining it, so a
   future fix flips a clearly-labelled assertion instead of silently breaking an
   unrelated test.
#. **Cross-platform CI from the start.** A GitHub Actions matrix over Ubuntu,
   Windows and macOS, added in the third PR, before the bulk of the tests existed.
#. **Document every pull request.** Each PR ships a matching reStructuredText
   document in ``gsocDocs/`` explaining what the tests cover and why.


What I Built
============

..  list-table::
    :widths: 55 45
    :header-rows: 0

    * - Test functions written
      - **2,678** (~3,500 collected after parametrisation)
    * - Test files
      - **71** across **16** subsystem packages
    * - Test classes / parametrised cases
      - 572 ``class Test…`` groups, 245 ``@pytest.mark.parametrize`` decorators
    * - Shared fixture modules
      - 16 ``conftest.py`` files
    * - Lines added over the program
      - ~49,000 insertions across 116 files
    * - Full suite runtime
      - **~18 seconds** (versus minutes for model-level tests)
    * - Technical documentation
      - 12 documents, ~16,900 lines, ~638 KB in ``gsocDocs/``

Coverage by subsystem
---------------------

..  list-table::
    :widths: 26 10 10 54
    :header-rows: 1

    * - Package
      - Files
      - Tests
      - What it covers
    * - ``outputs``
      - 11
      - 487
      - Field outputs, snapshots, geometry views and objects, HDF5 and MPI grid views
    * - ``utilities``
      - 6
      - 295
      - Host and hardware detection, logging, MPI helpers, formatting
    * - ``vtkhdf``
      - 4
      - 216
      - VTKHDF file handlers, image data and unstructured grid writers
    * - ``user_objects``
      - 5
      - 213
      - The Python API object model: geometry, sources, outputs, validation
    * - ``subgrids``
      - 5
      - 195
      - HSG (Huygens) subgridding, precursor nodes, main-grid coupling
    * - ``config``
      - 5
      - 177
      - Simulation and model configuration, precision dtypes, device selection
    * - ``grid``
      - 5
      - 173
      - ``FDTDGrid`` construction, array allocation, CFL time step, model wiring
    * - ``pml``
      - 6
      - 173
      - HORIPML and MRIPML absorbing boundaries, build and update paths
    * - ``hash_parser``
      - 4
      - 170
      - ``#`` command parsing — file, geometry, single-use and multi-use commands
    * - ``fractals``
      - 6
      - 169
      - Fractal box, surface and volume generation, grass, surface modifiers
    * - ``updates``
      - 4
      - 164
      - CPU field update dispatch, dispersive material dispatch, source updates
    * - ``geometry_primitives``
      - 6
      - 113
      - Cython voxel and shape builders, build dispatch, geometric predicates
    * - ``materials``
      - 1
      - 52
      - Material properties, Debye dispersion, averaging
    * - ``sources``
      - 1
      - 41
      - Hertzian and magnetic dipoles, voltage sources, transmission lines
    * - ``waveforms``
      - 1
      - 23
      - Analytical waveform types and their derivatives
    * - ``receivers``
      - 1
      - 17
      - Receiver definition and output component selection

Testing approach
----------------

* **Class-based grouping.** Tests are organised as ``TestBehaviour::test_case``
  around the unit under test, with descriptive names following
  ``test_<unit>_<context>_<expected>``.
* **Factory fixtures.** Shared ``conftest.py`` modules expose factories —
  ``make_waveform``, ``make_material``, ``make_dispersive``, ``fake_grid``,
  ``make_subgrid``, ``make_pml`` — that build configurable test doubles, mostly
  ``SimpleNamespace`` stand-ins for ``FDTDGrid`` and the global config.
* **Heavy parametrisation.** 245 ``parametrize`` decorators expand 2,678 written
  functions into roughly 3,500 executed cases.
* **Isolation with mocks.** 38 modules use ``monkeypatch`` or
  ``unittest.mock`` to intercept device transfers, subprocess probes and file
  writes, so the suite needs no GPU, no MPI job and no scratch space.
* **Documented failure modes.** Test docstrings name the source line to inspect
  when the assertion fails, so a failure points at a cause rather than a symptom.


Development Workflow
====================

gprMax is a fast-moving codebase. During the coding period alone the upstream
``devel`` branch absorbed roughly **30 pull requests and 185 commits** from other
maintainers — OpenCL backend parity, MPI plane waves, NTFF outputs, virtual
waveguides, Metal plane-wave kernels, new toolboxes — all touching the same
modules the tests were being written against. Working directly against a moving
branch was not viable, so the project used a fork with a dedicated integration
branch.

.. code-block:: none

    gprMax/gprMax (upstream)
     └── devel ...................... upstream development branch
          │                           (master is release/production code)
          │  fork
          ▼
    sahibjotsingh05/gprMax (this fork)
     └── gsoc26-unit-testing ........ GSoC integration branch, cut from devel
          ├── feat/unit-testing-waveforms ......... PR  #2 ─┐
          ├── feat/ci-pytest-workflow ............. PR  #3  │
          ├── feat/unit-testing-materials ......... PR  #4  │ each feature branch
          ├── feat/unit-tests-sources-receivers ... PR  #5  │ opens a PR back into
          ├── ...                                           │ gsoc26-unit-testing
          └── feat/unit-tests-updates-utils-config  PR #12 ─┘
          │
          │  periodic: git merge upstream/devel  +  conflict resolution
          ▼
     └── gsoc26-unit-testing-integration
          │
          └──▶ PR gprMax/gprMax#750  ──▶  upstream devel   (merged)

The loop in practice:

#. **Fork upstream and branch.** ``gsoc26-unit-testing`` was cut from ``devel``,
   the development branch, rather than ``master``, which holds production code.
#. **One feature branch per subsystem.** ``feat/<short-name>`` branches were cut
   from the integration branch and opened as pull requests back into it — never
   directly into ``devel`` — so each subsystem was reviewed on its own.
#. **Sync with upstream regularly.** ``upstream/devel`` was merged into the
   integration branch throughout the program and conflicts resolved by hand,
   in ``setup.py``, ``.gitignore`` and ``gprMax/utilities/host_info.py`` among
   others. Merging rather than rebasing kept review history intact.
#. **Integrate, then upstream.** A final ``gsoc26-unit-testing-integration``
   branch reconciled the suite with the current ``devel`` layout and became
   `PR #750 <https://github.com/gprMax/gprMax/pull/750>`__.

A recurring lesson worth recording: after a sync, the Cython extensions must be
rebuilt, and ``gprMax/cython/fields_updates_dispersive.pyx`` must be deleted
first. It is generated from a Jinja2 template only when absent, and it is
gitignored, so a stale copy survives every merge and silently compiles old
dispersive kernels against new source.


Pull Request Log
================

Progress across the program can be tracked through the pull requests opened on
this fork —
`all closed pull requests <https://github.com/sahibjotsingh05/gprMax/pulls?q=is%3Apr+is%3Aclosed>`__.
Every one carries its own technical document in ``gsocDocs/``.

..  list-table::
    :widths: 6 34 14 10 36
    :header-rows: 1

    * - PR
      - Title
      - Merged
      - Tests
      - Description
    * - `#1 <https://github.com/sahibjotsingh05/gprMax/pull/1>`__
      - feat: setup and wmic fix
      - 2 Jun 2026
      - —
      - Fixed a crash in ``host_info.py`` on Windows 11 25H2, where ``wmic`` has been removed; established the ``gsocDocs/`` documentation convention and the branch model.
    * - `#2 <https://github.com/sahibjotsingh05/gprMax/pull/2>`__
      - feat: unit tests for waveforms.py
      - 19 Jun 2026
      - 23
      - First test package. Analytical waveform types, their derivatives and boundary behaviour; introduced the shared root fixtures.
    * - `#3 <https://github.com/sahibjotsingh05/gprMax/pull/3>`__
      - CI Unit Tests Integration
      - 19 Jun 2026
      - —
      - GitHub Actions workflow running the suite on Ubuntu, Windows and macOS with a system MPI runtime and a compiled Cython build; dependency and ``setup.py`` fixes.
    * - `#4 <https://github.com/sahibjotsingh05/gprMax/pull/4>`__
      - Unit Testing Materials
      - 23 Jun 2026
      - 52
      - Material properties, Debye dispersive parameters and material averaging, asserted against closed-form relaxation formulas.
    * - `#5 <https://github.com/sahibjotsingh05/gprMax/pull/5>`__
      - Adding Unit Tests: Sources, Recievers
      - 24 Jun 2026
      - 58
      - Hertzian and magnetic dipoles, voltage sources, transmission lines and receiver output selection.
    * - `#6 <https://github.com/sahibjotsingh05/gprMax/pull/6>`__
      - hashparser unit tests
      - 29 Jun 2026
      - 170
      - The ``#`` command input language: file, geometry, single-use and multi-use command parsing, including malformed-input handling.
    * - `#7 <https://github.com/sahibjotsingh05/gprMax/pull/7>`__
      - User Objects Unit Tests
      - 30 Jun 2026
      - 213
      - The Python API object model — geometry, multi-use, single-use and output commands — and their validation rules.
    * - `#8 <https://github.com/sahibjotsingh05/gprMax/pull/8>`__
      - Geometory Primitives Unit Tests
      - 13 Jul 2026
      - 113
      - Cython voxel and shape builders (box, sphere, cylinder, cone, triangle, ellipsoid), build dispatch and geometric predicates.
    * - `#9 <https://github.com/sahibjotsingh05/gprMax/pull/9>`__
      - unit tests: geometry and fractals
      - 2 Aug 2026
      - 178
      - Fractal box, surface and volume generation, grass modelling and surface modifiers, including spectral-domain behaviour.
    * - `#10 <https://github.com/sahibjotsingh05/gprMax/pull/10>`__
      - Unit Tests Grid and Subgrids
      - 2 Aug 2026
      - 368
      - ``FDTDGrid`` construction and array allocation, the CFL stability time step, and HSG subgridding with precursor nodes and main-grid coupling.
    * - `#11 <https://github.com/sahibjotsingh05/gprMax/pull/11>`__
      - Unit Tests: PML, Snapshots, Outputs
      - 3 Aug 2026
      - 660
      - HORIPML and MRIPML absorbing boundaries, snapshot machinery, field and geometry outputs, HDF5 and VTKHDF writers, MPI grid views.
    * - `#12 <https://github.com/sahibjotsingh05/gprMax/pull/12>`__
      - Unit Tests: updates, utils, config
      - 9 Aug 2026
      - 852
      - CPU and dispersive field update dispatch, host and device detection, logging, and the global configuration singleton.


Upstream Integration — gprMax/gprMax#750
========================================

`PR #750 <https://github.com/gprMax/gprMax/pull/750>`__ merged the suite from
``sahibjotsingh05:gsoc26-unit-testing-integration`` into ``gprMax:devel``. It was
opened on **2 June 2026**, tracked the whole program, and was **merged on
16 August 2026** with 40 commits.

The integration itself was a distinct piece of engineering, because upstream had
independently grown its own ``tests/`` tree while this work was in progress:

* **72 test files were moved with** ``git mv`` **into 16 domain directories**
  matching upstream's existing layout, rather than being parked in a parallel
  subtree — so the contribution reads as an extension of their suite, not a
  competing one.
* **Three filename collisions** with existing upstream tests were resolved with a
  ``*_unit`` suffix, and the shared factory fixtures were centralised into the
  root ``tests/conftest.py``.
* ``--import-mode=importlib`` was added to ``pyproject.toml`` so that duplicate
  module basenames across packages cannot collide under pytest's default import
  mode, and the CI workflow was repointed at the new locations.
* **50 tests were marked** ``xfail`` **with explanations**, documenting places
  where upstream APIs and kernels had moved — including several bugs that this
  project's tests had originally pinned and that upstream had since fixed
  independently.
* Follow-up commits adapted the tests to current schemas, scoped the MPI/OFI
  provider override to Linux, improved cross-platform reliability, and fixed a
  fractal spectral-distance exponent bug found in the process.

Final result reported on the PR: **3,956 passed, 49 xfailed in 17.9 seconds**,
with 941 slow upstream integration tests deselected, verified across Ubuntu,
Windows and macOS. Maintainer ``agianno`` merged current ``devel`` into the
contributor branch rather than rebasing, preserving attribution.


Technical Documentation — ``gsocDocs/``
=======================================

Every pull request ships a matching reStructuredText document, committed in the
same PR it describes. The convention is recorded in ``gsocDocs/setup.rst``:

    This folder (``gsocDocs/``) holds documentation written alongside the GSoC
    unit-testing project. Every pull request gets a corresponding ``.rst`` doc
    summarising what it does and why.

The documents are written for other developers — mentors, reviewers and future
contributors — not as a duplicate of the tests. Each one follows the same
structure: scope, the physics or design concept the subsystem implements, the
test infrastructure and fixtures, then a catalogue of every test class and every
test function. Individual entries name the assertion, the property being checked,
and the source line to inspect when it fails:

    ``test_zero_at_chi`` — asserts ``calculate_value(chi) == 0`` for
    ``type="gaussiandot"``. The first derivative of any function is zero at the
    function's peak. Failure indicates the derivative formula at
    ``waveforms.py:103`` has been altered so the ``delay`` factor no longer
    multiplies the exponential cleanly.

..  list-table::
    :widths: 55 45
    :header-rows: 1

    * - Document
      - Covers
    * - ``gsocDocs/setup.rst``
      - Documentation convention, branch model, ``notes/`` versus ``gsocDocs/``
    * - ``feats/setup-and-wmic-fix.rst``
      - PR #1 — Windows 11 host detection fix
    * - ``feats/unit-testing-waveforms.rst``
      - PR #2 — waveforms
    * - ``feats/unit-testing-materials.rst``
      - PR #4 — materials and dispersion
    * - ``feats/unit-tests-sources-receivers.rst``
      - PR #5 — sources and receivers
    * - ``feats/unit-tests-hashparser.rst``
      - PR #6 — the ``#`` command language
    * - ``feats/unit-tests-user-objects.rst``
      - PR #7 — Python API object model
    * - ``feats/unit-tests-geometry-primitives.rst``
      - PR #8 — Cython geometry builders
    * - ``feats/unit-tests-geometry-fractals.rst``
      - PR #9 — fractal geometry
    * - ``feats/unit-tests-grid-subgrids.rst``
      - PR #10 — FDTD grid, CFL, HSG subgridding
    * - ``feats/unit-tests-pml-snapshots-outputs.rst``
      - PR #11 — PML, snapshots, outputs (4,461 lines)
    * - ``feats/unit-tests-updates-utils-config.rst``
      - PR #12 — update dispatch, utilities, config (5,654 lines)


Technologies & Skills
=====================

**Language and runtime**
  Python 3 (CI on 3.11), reStructuredText, YAML, Bash and PowerShell.

**Testing**
  `pytest <https://docs.pytest.org/>`__ — fixtures and factory fixtures,
  ``conftest.py`` layering, ``@pytest.mark.parametrize``, ``monkeypatch``,
  ``xfail`` and ``skip`` markers, custom markers with ``--strict-markers``,
  ``--import-mode=importlib``; ``unittest.mock`` (``MagicMock``, ``patch``,
  ``side_effect``); ``types.SimpleNamespace`` test doubles; test isolation of
  global singleton state; regression pinning of known bugs.

**Native code and accelerators**
  `Cython <http://cython.org>`__ — 15 ``.pyx`` extension modules compiled through
  ``cythonize`` in ``setup.py``, with `OpenMP <http://www.openmp.org>`__ threading
  and per-platform compiler flags (``/openmp`` on MSVC,
  ``-fopenmp -march=native`` on Linux/GCC).
  **NVIDIA CUDA** via `PyCUDA <https://documen.tician.de/pycuda/>`__ —
  ``pycuda.compiler.SourceModule`` kernel compilation and ``pycuda.gpuarray``
  device arrays in ``gprMax/updates/cuda_updates.py`` and
  ``gprMax/grid/cuda_grid.py``, with host-to-device and device-to-host transfer
  paths for PML, sources, receivers and snapshots; CUDA device enumeration and
  complex dtype selection in ``gprMax/config.py``. Because CI runners have no
  GPU, the CUDA layer is tested by mocking the driver and asserting on the
  dispatch, argument marshalling and transfer logic.
  **OpenCL** via `PyOpenCL <https://documen.tician.de/pyopencl/>`__ —
  ``ElementwiseKernel``, command queues and device arrays.
  **Apple Metal** via ``pyobjc-framework-metal`` for M-series GPUs.

**Code generation**
  `Jinja2 <https://jinja.palletsprojects.com/>`__ at two levels — build-time
  generation of the dispersive Cython kernels from
  ``fields_updates_dispersive_template.jinja`` into ``.pyx`` during ``setup.py``,
  and runtime assembly of CUDA/OpenCL/Metal device kernels from ``.tmpl``
  templates in ``gprMax/cuda_opencl/``.

**Parallel and distributed computing**
  `MPI <https://mpi4py.readthedocs.io/>`__ via ``mpi4py`` across 19 modules —
  including tests that construct real ``MPI.COMM_SELF`` and Cartesian
  communicators (``Cartcomm``) for domain-decomposed grid views and PML slabs;
  OFI/libfabric provider configuration for CI.

**Scientific computing and data formats**
  `NumPy <https://numpy.org>`__ (array allocation, dtype and precision handling,
  broadcasting, structured field arrays), `SciPy <https://scipy.org>`__,
  `HDF5 <https://www.hdfgroup.org/>`__ via ``h5py`` including parallel HDF5,
  VTKHDF and VTK ImageData / UnstructuredGrid output writers.

**CI/CD and tooling**
  `GitHub Actions <https://docs.github.com/actions>`__ — a three-OS matrix
  (``ubuntu-latest``, ``windows-latest``, ``macos-latest``), pip caching keyed on
  build files, ``mpi4py/setup-mpi`` for a system MPI runtime, editable installs
  that compile the Cython extensions, import smoke tests, concurrency groups with
  ``cancel-in-progress``, and environment control via ``OMP_NUM_THREADS`` and
  ``MPI4PY_RC_FINALIZE``. ``black`` and ``isort`` via ``pre-commit``;
  ``setuptools`` packaging.

**Git and collaboration**
  Fork and upstream remote management, long-lived integration branches, feature
  branch workflow, pull request review cycles, repeated upstream merges with
  manual conflict resolution, ``git mv`` history-preserving restructuring, merge
  versus rebase trade-offs for attribution, and writing technical documentation
  for maintainer review.

**Domain knowledge**
  Finite-Difference Time-Domain method, Maxwell's equations, the Yee cell,
  the Courant–Friedrichs–Lewy (CFL) stability condition, perfectly matched layers
  (HORIPML and MRIPML formulations), Debye dispersive materials and material
  averaging, Huygens subgridding (HSG), plane-wave sources, near-to-far-field
  transformation, transmission lines, fractal-based geometry generation, and
  ground penetrating radar modelling.


Running the Test Suite
======================

The Cython extensions must be compiled before the tests will import.

.. code-block:: console

    $ conda env create -f conda_env.yml
    $ conda activate gprMax
    $ pip install -e .
    $ pip install pytest
    $ python -m pytest tests/unit/ -v

To run a single subsystem, or a single test class:

.. code-block:: console

    $ python -m pytest tests/unit/pml/ -v
    $ python -m pytest tests/unit/waveforms/test_waveforms.py::TestGaussian -v

After merging upstream changes, delete the generated dispersive kernel before
rebuilding, otherwise a stale copy is silently reused:

.. code-block:: console

    $ rm -f gprMax/cython/fields_updates_dispersive.pyx
    $ pip install -e .


About gprMax
============

`gprMax <http://www.gprmax.com>`__ is open source software that simulates
electromagnetic wave propagation, solving Maxwell's equations in 3D using the
Finite-Difference Time-Domain method. It was designed for modelling Ground
Penetrating Radar (GPR) but is also used for many other electromagnetic
modelling applications. It is written principally in Python 3 with
performance-critical parts in Cython, and includes accelerators for CPU
(OpenMP), CPU/GPU (OpenCL), GPU (NVIDIA CUDA), and GPU (Apple Metal on
M-series chips), together with MPI support for multi-node simulations.

**This repository is a fork.** For installation instructions, the user guide and
general use of gprMax, please go to the upstream project:

* Upstream repository — `github.com/gprMax/gprMax <https://github.com/gprMax/gprMax>`__
* Documentation — `docs.gprmax.com <http://docs.gprmax.com>`__
* Website — `www.gprmax.com <http://www.gprmax.com>`__
* GSoC ideas — `github.com/gprMax/GSoC <https://github.com/gprMax/GSoC>`__


License & Citation
==================

gprMax is released under the
`GNU General Public License v3 or higher <http://www.gnu.org/copyleft/gpl.html>`__.
This fork inherits that license.

If you use gprMax and publish your work, please cite:

* Warren, C., Giannopoulos, A., & Giannakis I. (2016). gprMax: Open source
  software to simulate electromagnetic wave propagation for Ground Penetrating
  Radar, `Computer Physics Communications`
  (http://dx.doi.org/10.1016/j.cpc.2016.08.020)

Thanks to the gprMax maintainers for mentoring this project, and to Google
Summer of Code for making it possible.

.. image:: https://contrib.rocks/image?repo=gprMax/gprMax
   :target: https://github.com/gprMax/gprMax/graphs/contributors
   :alt: Contributors

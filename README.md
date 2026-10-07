# cdmHUB Halpin–Tsai calculator: Python migration

Python translation of the calculation path used by the original cdmHUB
**Halpin-Tsai Micromechanics Model**, Johnathan Goodsell and Andrew J. Ritchey
(2014), https://cdmhub.org/resources/mmtool.

## Function to serve

Serve **`halpin_tsai.calculate`**. It computes the original Voigt, Reuss and
property-wise Halpin–Tsai predictions for a continuous unidirectional lamina
with transversely isotropic fibers and an isotropic matrix. It returns eight
properties versus fiber volume fraction, Halpin–Tsai point properties, and
seven off-axis property curves. This package is the calculation contribution
to the proposed shared Analytical Micromechanics skill. It is not an installed
CompositesAI tool or a replacement for that skill's instructions.

No MATLAB, Octave, HTTP server or external executable is required.
Runtime dependency: NumPy. Python 3.12 is supported.

## Run locally

Install `uv` if needed using https://docs.astral.sh/uv/getting-started/installation/.
Open a terminal in this folder, then run:

```sh
uv lock
uv sync
uv run pytest
uv run python examples/default_case.py
```

The example prints the point results and writes the full response to
`default_results.json` in the current folder.

## Usage

```python
from halpin_tsai import calculate

result = calculate(
    fiber_e1=100e9, fiber_e2=10e9, fiber_nu12=0.3, fiber_nu23=0.3,
    fiber_g12=4e9, fiber_alpha1=1e-6, fiber_alpha2=10e-6,
    matrix_e=5e9, matrix_nu=0.33, matrix_alpha=60e-6,
    fiber_volume_fraction=0.6,
)
print(result['halpin_tsai_point'])
```

All default inputs, including the eight separate xi factors, come from the
original `tool.xml`. The complete parameter meanings, ranges, units and return
schema are in `calculate`'s NumPy-style docstring. Moduli must be in **Pa**,
CTEs in **1/K**, and Poisson ratios, xi and volume fraction are dimensionless.
The original interface defaults xi to 1e10 for E1, nu12 and alpha1 and to 1
for E2, nu23, G12, G23 and alpha2. The original wrapper disables the optional
internal xi-estimation mode; this package exposes that same interface path.

## Output structure

- `fiber_volume_fraction`: requested scalar Vf.
- `halpin_tsai_point`: E1, E2, G12, G23 in Pa; nu12, nu23 dimensionless;
  alpha1, alpha2 in 1/K.
- `volume_fraction`: 1001 samples, 0 to 1 in steps of 0.001.
- `voigt`, `reuss`, `halpin_tsai`: the same eight properties as lists
  aligned with `volume_fraction`.
- `off_axis`: theta_deg from 0 to 90 in one-degree steps, Ex, Ey, Gxy in Pa,
  nuxy dimensionless, alphax, alphay, alphaxy in 1/K. These use the
  Halpin–Tsai point. alphaxy is engineering shear thermal expansion;
  angle/sign convention follows the original LaminaTransform function.

## Preservation and limitations

The original MATLAB source and interface XML are kept unchanged in `reference/`.
The port preserves matrix stiffness/compliance averaging, property-wise
Halpin–Tsai formulas (including their application to Poisson ratios and CTEs),
legacy thermal mixing relations and off-axis transformations. This is a
behavior-preserving migration, not a scientific correction or experimental
validation. Do not describe every Poisson ratio or thermal result as a rigorous
upper/lower bound merely because it is under a Voigt/Reuss label.

The original grid remains 0.001. The wrapper rejects off-grid point requests
rather than interpolating or changing the calculation. It maps an on-grid
request to its integer grid index to avoid language-dependent float equality.
New boundary checks raise ValueError for invalid inputs or non-finite results.
The original expressions can be singular for zero/signed CTEs or other property
combinations. They are not replaced by alternate formulas. Positive definite
constituent compliance is required; an empirical output is not certified as a
physically admissible material merely because calculation succeeds.

Not for strength, damage, failure, woven or short-fiber predictions. The full
curve calculation runs in under a second on the development machine; server
performance depends on hardware. No new physical model or default was added.

## Validation

`tests/test_matlab_reference.py` compares all 24 property curves, all seven
off-axis curves, both coordinate grids and all eight point values against
Jainam's supplied `original_reference.mat`. This is one complete default-input
regression case, not coverage of every possible input. `tests/test_examples.py`
contains three separately derived hand-calculation/identity cases.
The MATLAB table object is not used; numerical arrays are read with SciPy.
See `VALIDATION.md` for the recorded run.

## Release and handover

Version is 0.1.0. Before handing over, run `uv run pytest` and put this folder's
contents in the agreed repository. Generate and commit `uv.lock` locally; dependency resolution was blocked in the
preparation environment, so it is not included yet. Exclude `.venv`, caches
and generated results. For private hosting, the supplied contribution guide
requires the repository under `wenbinyugroup`.

After review, tag the release `v0.1.0` and send the maintainer the repository and
tag with `halpin_tsai.calculate` as the function to serve. The maintainer owns
the adapter, API deployment and Open WebUI integration. This deliverable has
not been published, tagged remotely or deployed.

## Attribution and references

Original tool: Johnathan Goodsell; Andrew J. Ritchey (2014),
“Halpin-Tsai Micromechanics Model,” https://cdmhub.org/resources/mmtool.
Original source includes supporting functions attributed to Andrew Ritchey.
Python migration prepared for Jainam Mehta's CompositesAI contribution.

References identified by the original tool/source:
- Whitney, J. M. and McCullough, R. L., Delaware Composites Design Encyclopedia,
  Vol. 2, Micromechanical Materials Modeling (1990), pp. 50–56, 82–89.
- Halpin, J. C. and Kardos, J. L., “The Halpin–Tsai equations: a review,”
  Polymer Engineering and Science 16(5), 344–352.
- Daniel and Ishai, Engineering Mechanics of Composite Materials, 2nd ed.,
  p. 74, Eq. 4.48, as cited in the compliance routine.
- Sun, Composite Mechanics, Chapters 3 and 5, as cited in the transform routine.

No license grant was included in the supplied source files. This migration
adds no open-source license on behalf of the original authors; use the group's
approved distribution terms when releasing it.

Default point results (rounded): E1 = 61.99999995668 GPa, E2 = 7.5 GPa,
nu12 = 0.312, nu23 = 0.3116666667, G12 = 2.9177054909 GPa,
G23 = 2.8554209952 GPa, alpha1 = 2.4599999999e-5 1/K,
alpha2 = 2.4e-5 1/K. Returned moduli are in Pa; GPa above is for readability.

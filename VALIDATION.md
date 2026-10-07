# Validation status for 0.1.0

Executed under Python 3.12 in the preparation environment:

- Full numerical regression against the supplied MATLAB workspace: passed.
  34 comparisons cover 24 volume-fraction property curves (1001 points each),
  seven off-axis curves (91 points each), two grids and eight point values.
  Comparison uses relative tolerance 1e-10 and absolute tolerance 1e-14.
- Three independent hand-calculation/identity examples: passed.
- Nine invalid-input cases and strict JSON serialization: passed.
- Python wheel build with setuptools: passed.
- Import and default-call check using the installed wheel: passed.

Recorded outputs and per-array maximum absolute errors are in
`verification_results.json`. Reproduce the numerical checks with
`uv run python examples/verify_reference.py`.

Not completed here: dependency resolution/uv.lock generation, installation in
an isolated uv environment and execution of pytest. Package-index access was
blocked; pytest was not preinstalled. The independent verification script was
executed with the installed NumPy/SciPy, not through pytest. Do not count this
as passing the contribution guide's release gate yet.

Before release, run in a network-enabled terminal from this folder:

```
uv lock
uv sync
uv run pytest
uv run python examples/default_case.py
```

Commit the generated uv.lock. No GitHub release or CompositesAI deployment has
been performed. MATLAB agreement establishes translation fidelity for the
provided input set, not scientific validity for all possible materials.

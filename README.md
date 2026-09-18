# Hamiltonian Simulation for Pauli Operators

A Qiskit simulation of Trotterized time evolution under a single-qubit
Hamiltonian built from a sum of Pauli operators, `H = X + Y + Z`, tracking
how the expectation values `⟨X⟩(t)`, `⟨Y⟩(t)`, `⟨Z⟩(t)` evolve with time.

## Background

`H = X + Y + Z` is proportional to `n̂ · σ` for the (unnormalized) axis
`n̂ = (1, 1, 1)`, with `|n̂| = √3`. Its eigenvalues are `±√3`, so the exact
evolution `U(t) = exp(-iHt)` makes the Bloch vector precess uniformly
around the `(1, 1, 1)/√3` axis — analogous to Larmor precession of a spin
in a magnetic field pointing along that diagonal. Because Qiskit's gate
set doesn't include a native `exp(-i(X+Y+Z)t)` gate, this notebook
**Trotterizes** the evolution: it splits `exp(-i(X+Y+Z)t)` into `n`
repetitions of `exp(-iXt/n)·exp(-iYt/n)·exp(-iZt/n)`, each of which *is* a
native rotation gate, using Qiskit's `PauliTrotterEvolution`. As `n`
increases, the Trotterized circuit converges to the exact (non-commuting)
evolution.

The notebook computes the exact expectation values analytically (via
`qiskit.opflow`'s operator algebra, not by sampling/measuring shots),
starting from the qubit prepared in `|0⟩`, and plots all three components
of the Bloch vector as functions of time.

## Repository contents

| File | Description |
|---|---|
| `SIMULAATIOHAMILTON-X+Y+Z.ipynb` | Builds the Trotterized circuit for `H = X + Y + Z`, evolves the initial state `\|0⟩`, evaluates `⟨X⟩`, `⟨Y⟩`, `⟨Z⟩` at 1000 evenly spaced time points, and plots the three curves together. |

## Requirements

- **Python 3.8–3.11** (Qiskit's compiled Aer backend does not ship
  pre-built wheels for Python 3.12+ at the pinned version below — Aer
  isn't strictly required for this notebook's analytic-expectation-value
  path, but is imported at the top of the notebook)
- `qiskit==0.39.4` (last release series exposing `qiskit.opflow` —
  `PauliTrotterEvolution`, `StateFn`, `PauliExpectation`, `CircuitSampler`
  — which was deprecated in later Qiskit releases and removed in Qiskit 1.0)
- `numpy<1.24`
- `matplotlib`

Install everything into a fresh virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate      # .venv\Scripts\activate on Windows
pip install -r requirements.txt
```

## Usage

1. Launch Jupyter (`jupyter notebook` or open in VS Code) and open
   `SIMULAATIOHAMILTON-X+Y+Z.ipynb`.
2. Run all cells in order:
   - **Cell 1** — imports.
   - **Cell 2** — defines `hamiltonian = X + Y + Z`, a symbolic time
     parameter `evolution_time`, and Trotterizes `exp(i·t·H)` with
     `n = 10` repetitions; draws the resulting circuit.
   - **Cell 3–4** — defines the `X`, `Y`, `Z` observables as measurement
     operators and combines them with the evolution operator and the
     initial state `\|0⟩` into three "evolve-then-measure" expressions,
     Trotterized again with `n = 10` (this second `n` — reused from the
     circuit-drawing step — controls the *analytic* Trotter
     approximation actually used for the expectation-value sweep below).
   - **Cell 5** — sweeps `evolution_time` from `0` to `time_total = 3`
     over `num_of_steps = 1000` points, evaluating all three expectation
     values at each point.
   - **Cell 6** — plots `⟨X⟩(t)`, `⟨Y⟩(t)`, `⟨Z⟩(t)` on one set of axes.

### Adjusting the simulation

```python
n = 10                 # Trotter steps (higher = closer to exact evolution)
time_total = 3          # total time window to sweep over
num_of_steps = 1000     # time-resolution of the sweep
```

- Increasing `n` reduces Trotterization error (the discretization
  artifact from splitting non-commuting `X`, `Y`, `Z` rotations into
  sequential gates) but increases circuit depth/evaluation time.
- Increasing `num_of_steps` (for a fixed `time_total`) only increases
  plot resolution — it doesn't change the physics, since each point is
  evaluated independently from the closed-form Trotterized operator.
- To start from a different initial state (e.g. `\|+⟩` instead of `\|0⟩`),
  swap out `Statevector([1, 0])` in cell 3 for e.g.
  `Statevector([1/np.sqrt(2), 1/np.sqrt(2)])` (already present, commented
  out, in the source).

## Interpreting the output

All three expectation values should trace out bounded oscillations
between roughly `-1` and `1` (never exceeding the Bloch-sphere radius),
precessing around the `(1, 1, 1)` axis with angular frequency `2√3` for
the exact (untrotterized) Hamiltonian. With `n = 10` Trotter steps the
curves should already closely track this analytic precession over the
`t ∈ [0, 3]` window used here; visible deviation from smooth sinusoidal
precession is a sign that `n` is too small for the chosen `time_total`.

# Quantum–Classical Ewald Summation

A hybrid quantum–classical implementation of the Ewald summation for the
electrostatic energy of a periodic 3D lattice of point charges. The
long-range (reciprocal-space) term $E^L$ is obtained via the quantum
Fourier transform, while the remaining short-range, self-interaction, and
dipole correction terms are computed classically. 

The corresponding preprint is available on arXiv:
Mansur Ziiatdinov, Igor Novikov, Farid Ablayev, and Valeri Barsegov, Quantum Algorithm for Ewald Summation–Based Computation of Long-Range Electrostatics, arXiv. https://doi.org/10.48550/arXiv.2512.20886

## Repository contents

```
Quantum-classical_Ewald-Summation.ipynb   Main notebook
requirements.txt                          Python dependencies
.gitignore
README.md
```

## Requirements

Python 3.10+ is recommended. Install dependencies with:

```bash
pip install -r requirements.txt
```

## Running

Open the notebook in Jupyter or VS Code and execute the cells top-to-bottom:

```bash
jupyter notebook Quantum-classical_Ewald-Summation.ipynb
```

The `EXACT_SIM` flag at the top of the constants cell selects between
ideal statevector simulation (`True`) and sampling-based simulation
(`False`, default, `N_SHOTS = 5000`).

The "Run on a model of the real device" section uses the
`FakeMontrealV2` backend and the final "Quantum circuit complexity"
section uses `FakeSherbrooke`. Both are heavy: in the saved outputs, the
Montreal sampling cell took about 37,000 seconds. Do not run those cells
unless you intend to wait.

## Algorithm

The total energy is decomposed as

```
$E = E^L + E^S + E^self + E^dipole$
```

with the following components:

| Term         | Where computed                                  |
|--------------|-------------------------------------------------|
| $E^L$        | Reciprocal-space kernel applied to either a classical FFT (`fftn`) or the squared probability density read out of a quantum QFT circuit. |
| $E^S$        | Real-space sum with `erfc` cutoff at `RCUT`, using a 3D cell list and `n` periodic image shells. |
| $E^self$     | Closed-form self-interaction term (vectorized). |
| $E^dipole$   | Surface (dipole) correction, with surrounding-medium permittivity `epsp`. |
| $E_Coulomb$  | Direct `O(N²)` Coulomb sum over `m` image shells, used as ground truth. |

The quantum circuit places `D · log₂(M)` qubits (15 at `M = 32`) in three
coordinate registers and applies a `QFT` to each.

## Key parameters

Defined in the constants cell:

- `M` — grid size per axis. Must be a power of 2.
- `D` — spatial dimensions (3).
- `step` — lattice spacing in metres.
- `SIGMA` — Ewald split width in metres (shared by `E^L` and `E^S`).
- `RCUT` — real-space cutoff radius for `E^S`.
- `n` — number of image shells used by `E^S`.
- `m` — number of image shells used by the direct Coulomb reference.
- `epsp` — surrounding-medium relative permittivity.
- `N_SHOTS` — shots for sampling-based quantum simulation.
- `EXACT_SIM` — toggle between exact statevector and sampling-based modes.

All energies are in joules (SI).
.

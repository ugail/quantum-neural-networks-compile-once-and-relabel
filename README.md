
## Compile-Once-and-Relabel Solution for Topology-Dependent Structural Asymmetry in Compiled Equivariant Quantum Neural Networks
## by Hassan Ugail and Newton Howard.

---


Equivariant quantum neural networks are designed so that their outputs respect symmetries in the input data. In nearly all published work this property is verified only at the level of the ideal circuit, before any compilation step. We show that compilation can quietly break the property at the structural level. Two algebraically equivalent inputs related by a symmetry of the data, compiled against the same hardware target with the same transpiler settings, can yield circuits that differ substantially in two-qubit gate count, scheduled depth, and physical-qubit layout. On IBM Heron's Fez at twelve qubits this divergence reaches 28% in CZ count between a graph and a random vertex permutation of it, with a mean of 19.2% across fifty seeds. Because two-qubit gate count is the dominant determinant of execution time and noise susceptibility on near-term superconducting hardware, an asymmetry of this size is not a cosmetic artefact.



<img width="890" height="614" alt="Figure2" src="https://github.com/user-attachments/assets/46051a8d-a850-4024-aa87-395d7675dd54" />



The asymmetry is not generic. We show that it arises only for ansätze in which the symmetry action changes the gate-topology that the compiler sees, and is provably absent for ansätze in which the symmetry action changes only numerical rotation angles inside an otherwise fixed gate structure. The two classes are called **topology-input** and **parameter-input** respectively, and the same symmetry group can appear in either class depending on the architecture. For the parameter-input case Theorem 1 of the paper guarantees zero CZ-count asymmetry under any transpiler whose CZ-count-affecting passes depend only on the gate-topology, and we verify this empirically across three published architectures (Dong, West, Chang). For the topology-input case (the Skolik $S_n$-equivariant ansatz) the paper proposes a deployment pattern called **compile-once-and-relabel**, in which a single representative of the symmetry orbit is compiled and every other orbit element is handled by a classical relabelling of inputs and measurement outcomes at the interface. The structural asymmetry then vanishes by construction, and the per-orbit-element compilation cost drops from linear to constant.


---

## Quick start

### Run on Google Colab (recommended)

Click **Open in Colab** (badge above), then run all cells. With `QUICK_MODE = True` (the default for first-time readers) the notebook completes in roughly **5 minutes** on Colab's free CPU runtime and reproduces every qualitative result.

### Run locally

```bash
git clone https://github.com/<owner>/<repo>.git
cd <repo>
pip install -r requirements.txt
jupyter notebook Paper1_Reproduction.ipynb
```

Then run cells top-to-bottom.

---

## Two operating modes

The notebook is controlled by a single configuration cell near the top.

```python
QUICK_MODE = True              # default
SAVE_PAPER_FIGURES = False     # set True to write 300 DPI PNGs
MASTER_SEED = 2026
```

| Mode | `K_scaling` | `K_mechanism` | Runtime (CPU) | Reproduces |
|------|-------------|---------------|---------------|------------|
| `QUICK_MODE = True` | 10 | 20 | ~5 minutes | All qualitative findings |
| `QUICK_MODE = False` | 25 | 50 | ~20 minutes | The exact headline numbers from the paper |

Set `QUICK_MODE = False` to match the paper's reported statistics (Fez $n=12$ mean relative CZ asymmetry of 19.2%, layout-overlap Spearman $\rho = -0.52$ with $p < 10^{-3}$, and so on).

---

## What gets reproduced

### Section 2.3 (Skolik, topology-input)
- Template-level equivariance residual at $n=6$ (~$5 \times 10^{-17}$)
- Fez $n=12$ structural sweep across 50 random graph-permutation pairs
- Mean relative CZ asymmetry of 19.2%, worst case 28%
- Scaling sweep on FakeFez and FakeTorino across $n \in \{4, 6, 8, 10, 12, 14, 16\}$

### Sections 3.2, 4.2, 5.1 (Dong, West, Chang &mdash; parameter-input)
- Template-level equivariance residuals (machine precision)
- Structural sweeps confirming `cz_diff = 0` exactly across all seeds, depths, and orbit elements
- Empirical confirmation of Theorem 1's parameter-input prediction

### Section 6 (Mechanism analysis)
- Automorphism hypothesis rejection ($\rho \approx \pm 0.08$, $p = 0.85$)
- Layout-overlap correlation ($\rho = -0.52$, $p < 10^{-3}$)
- Worst-seed analysis (qubits 117&ndash;147 vs 16&ndash;57)

### Section 7 (Mitigation)
- Matched-layout strategy: `cz_diff = 0` with +39% per-circuit overhead
- Compile-once-and-relabel unitary verification at $n=8$ (residual ~$4 \times 10^{-15}$)

### Figures
- **Figure 1**: Taxonomy bar chart and FakeFez/FakeTorino scaling
- **Figure 2**: Asymmetry distribution and layout-overlap scatter
- **Figure 3**: Three-strategy comparison and orbit-scaling cost

The final cell of the notebook prints a single table with all paper claims, their reproduced values, and the originating cell number.

---

## Dependencies

We ran against **Qiskit 2.4.0**. Compilation behaviour is sensitive to the exact versions of the transpilation toolchain, so the recommended reproduction stack is:

```
qiskit==2.4.0
qiskit-ibm-runtime
qiskit-aer
numpy
pandas
matplotlib
scipy
```

If you reproduce on a newer Qiskit release, the qualitative findings should hold, but the exact gate counts may shift slightly because of changes in transpiler heuristics.

---

## Hardware notes

All experiments use **simulator models** of two IBM Heron devices via `qiskit_ibm_runtime.fake_provider`:

- `FakeFez` (Heron revision two)
- `FakeTorino` (Heron revision one)

These targets reproduce each device's coupling map and native gate set exactly but **do not** include calibrated noise. The compilation-level results in the paper are noise-model agnostic: they depend on the transpiler and the target coupling map, not on calibrated device noise. Hardware execution under realistic noise is identified in the paper's discussion as the natural follow-up direction.

No IBM Quantum account is required to run this notebook; the fake-provider models are local.

---

## License

Released under the MIT License. See [LICENSE](LICENSE).

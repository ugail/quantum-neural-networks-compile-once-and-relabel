# Compile-Once-and-Relabel

> **Topology-Dependent Compilation Asymmetry in Equivariant Quantum Neural Networks: A Compile-Once-and-Relabel Solution**
> Hassan Ugail and Newton Howard.

---

Equivariant quantum neural networks are designed so that their outputs respect symmetries in the input data. In nearly all published work this property is verified only at the level of the ideal circuit, before any compilation step. We show that compilation can quietly break the property at the structural level. Two algebraically equivalent inputs related by a symmetry of the data, compiled against the same hardware target with the same transpiler settings, can yield circuits that differ substantially in two-qubit gate count, scheduled depth, and physical-qubit layout. On IBM Heron's Fez at twelve qubits, with sparse weighted-graph inputs drawn from $G(n, p=0.5)$, vertex permutations produce a mean relative CZ-count asymmetry of 7.3% with worst cases reaching 22.5% across fifty seeds. The effect persists across all four Qiskit optimisation levels and is not removed by matched-layout compilation, implicating routing-stage sensitivity to the ordered sparse-edge gate sequence as the dominant cause. A density sweep shows the asymmetry is comparable across intermediate graph densities and vanishes identically for complete graphs, where the ansatz degenerates into the parameter-input class, and stochastic baselines attribute the realised differences to heuristic routing sensitivity rather than to differing optimal compiled costs: the optimal compiled costs of symmetry-related inputs are provably identical, so every measured gate of asymmetry is heuristic suboptimality. Because two-qubit gate count is the dominant determinant of execution time and noise susceptibility on near-term superconducting hardware, an asymmetry of this size is large enough to matter for near-term execution.

<img width="3588" height="2468" alt="Mitigation strategies and compile-cost scaling (paper Figure 4)" src="https://github.com/user-attachments/assets/c8ab73a2-b4ff-4f2b-a328-2ac93e9f3f7f" />

The asymmetry is not generic. We show that it arises only for ansätze in which the symmetry action changes the gate-topology that the compiler sees, and is provably absent for ansätze in which the symmetry action changes only numerical rotation angles inside an otherwise fixed gate structure. The two classes are called **topology-input** and **parameter-input** respectively, and the same symmetry group can appear in either class depending on the architecture. For the parameter-input case Theorem 1 of the paper guarantees zero CZ-count asymmetry under any transpiler whose CZ-count-affecting passes depend only on the gate-topology, and we verify this empirically across three published architectures (Dong, West, Chang) at all four Qiskit optimisation levels. For the topology-input case (the Skolik $S_n$-equivariant ansatz) the paper proposes a deployment pattern called **compile-once-and-relabel**, in which a single representative of the symmetry orbit is compiled and every other orbit element is handled by a classical relabelling of inputs and measurement outcomes at the interface. The structural asymmetry then vanishes by construction, the per-orbit-element compilation cost drops from linear to constant, and the single compilation can be chosen as the best of $k$ transpiler seeds so that every orbit element inherits the best draw.

---

## Quick start

### Run on Google Colab (recommended)

Click **Open in Colab** (badge above), then run all cells. As shipped, the notebook is set to `QUICK_MODE = False` and reproduces the paper's full statistics in roughly **25 minutes** on Colab's free CPU runtime. Set `QUICK_MODE = True` in the configuration cell for a first pass that reproduces every qualitative result in roughly **5 minutes**.

### Run locally

```bash
git clone https://github.com/<owner>/<repo>.git
cd <repo>
pip install -r requirements.txt
jupyter notebook Compile-Once-and-Relabel.ipynb
```

Then run cells top-to-bottom.

---

## Two operating modes

The notebook is controlled by a single configuration cell near the top.

```python
QUICK_MODE = False             # as shipped (full paper statistics)
SAVE_PAPER_FIGURES = False     # set True to write 600 DPI PNGs
MASTER_SEED = 2026
```

| Mode | `K_scaling` | `K_mechanism` | Runtime (CPU) | Reproduces |
|------|-------------|---------------|---------------|------------|
| `QUICK_MODE = True` | 10 | 20 | ~5 minutes | All qualitative findings |
| `QUICK_MODE = False` | 25 | 50 | ~25 minutes | The exact headline numbers from the paper |

With `QUICK_MODE = False` the notebook matches the paper's reported statistics (Fez $n=12$ mean relative CZ asymmetry of 7.3%, worst case 22.5%, layout-overlap Spearman $\rho = +0.004$ with $p \approx 0.98$, and so on). Execution is fully deterministic under the pinned dependency stack and the fixed master seed, and has been verified to produce identical numerical output across independent platforms (Google Colab and a Linux container).

---

## What gets reproduced

### Skolik ($S_n$, topology-input) — notebook Section 2
- Template-level equivariance residual at $n=6$ on a sample $G(6, 0.5)$ graph (~$2 \times 10^{-16}$)
- Fez $n=12$ structural sweep across 50 random graph-permutation pairs from sparse Erdős–Rényi $G(n, p=0.5)$ conditioned on connectedness, with edges in canonical lexicographic order
- Mean relative CZ asymmetry of 7.3%, worst case 22.5%
- Scaling sweep on FakeFez and FakeTorino across $n \in \{4, 6, 8, 10, 12, 14, 16\}$
- Density sweep at fixed $n = 12$ across edge probabilities $p \in [0.10, 1.00]$: mean asymmetry 5.7–9.5% for $0.15 \le p \le 0.90$, elevated to 11.1% (worst case 35.6%) at the sparse boundary $p = 0.1$, and exactly zero for all fifty complete-graph pairs at $p = 1$, where the canonicalised edge set becomes input-independent and the ansatz degenerates into the parameter-input class; the $p = 0.5$ point reproduces the headline ensemble exactly (built-in anchor check)

### Parameter-input controls (Dong, West, Chang) — notebook Sections 3–5
- Template-level equivariance residuals (machine precision)
- Structural sweeps confirming `cz_diff = 0` exactly across all seeds, depths, and orbit elements
- All three audits repeated at Qiskit optimisation levels 0–3 with maximum observed $|\Delta_{\mathrm{CZ}}| = 0$ in every configuration
- Symmetry validation of the Chang-style control (the shared-parameter block commutes exactly with the diagonal reflection; the structural audit depends only on the block's fixed gate-topology)
- The executed pass composition of each preset pass manager for the Fez target, plus the exact dependency versions

### Mechanism analysis — notebook Section 6
- Automorphism hypothesis rejection (size-normalised orbits: $\rho = -0.08$, $p = 0.85$; absolute $|\mathrm{Aut}|$: $\rho = +0.41$, $p = 0.31$)
- Layout-overlap hypothesis rejection on sparse graphs ($\rho = +0.004$, $p \approx 0.98$); 40 of 50 seeds use entirely disjoint physical regions yet asymmetry magnitude spans the full range
- Worst-seed analysis (qubits 122–146 vs 21–46, with 163 vs 130 CZ gates)
- Optimisation-level robustness sweep: asymmetry persists across all four Qiskit opt levels (5.9–7.3% mean range, >22% worst-case throughout); matched-layout compilation fails at every level including `opt_level=0`, narrowing the dominant cause to the routing pass itself
- Stochastic baselines: the permutation asymmetry ($7.26\% \pm 0.79\%$) exceeds the within-graph transpiler-seed baseline ($4.97\% \pm 0.11\%$, one-sided paired Wilcoxon $p = 8.3 \times 10^{-3}$) and is comparable in magnitude to the independent matched-$|E|$ baseline ($7.62\% \pm 0.99\%$); for the first ensemble graph the fixed circuit's CZ count spans 155–195 over fifty transpiler seeds (mean 176.9, SD 8.6, mean pairwise 5.49%)

### Mitigation — notebook Section 7
- Matched-layout strategy: residual relative asymmetry of approximately 33% on sparse graphs at `opt_level=3`; not a reliable mitigation in the present transpilation regime
- Compile-once-and-relabel unitary verification at $n=8$ (residual ~$5 \times 10^{-15}$ once the layout corrections of Equation 2 are applied)

### Figures (numbering as in the paper)
- **Figure 1**: Taxonomy bar chart and FakeFez/FakeTorino scaling
- **Figure 2**: Density dependence of the structural asymmetry (rendered last in the notebook's figure section)
- **Figure 3**: Asymmetry distribution and layout-overlap scatter
- **Figure 4**: Three-strategy mitigation comparison and orbit-scaling cost

The final cell of the notebook prints a single reproducibility table with every numerical claim in the paper, its reproduced value, and the section it originates from.

---

## Dependencies

All numbers in the paper were generated with **Qiskit 2.4.0**. Compilation behaviour is sensitive to the exact versions of the transpilation toolchain, so the version pin is exact and the notebook installs it directly:

```
qiskit==2.4.0
qiskit-ibm-runtime
qiskit-aer
numpy
pandas
matplotlib
scipy
```

If you reproduce on a newer Qiskit release, the qualitative findings should hold, but the exact gate counts may shift because layout and routing heuristics change between transpiler releases.

---

## Hardware notes

All experiments use **simulator models** of two IBM Heron devices via `qiskit_ibm_runtime.fake_provider`:

- `FakeFez` (Heron revision two)
- `FakeTorino` (Heron revision one)

These targets reproduce each device's coupling map and native gate set exactly but **do not** include calibrated noise. The compilation-level results in the paper are noise-model agnostic: they depend on the transpiler and the target coupling map, not on calibrated device noise. Hardware execution under realistic noise is identified in the paper's discussion as the natural follow-up direction.

No IBM Quantum account is required to run this notebook; the fake-provider models are local.

---

## Citation

If you use this code or build on these results, please cite the paper (Ugail H and Howard N, *Topology-Dependent Compilation Asymmetry in Equivariant Quantum Neural Networks: A Compile-Once-and-Relabel Solution*; full bibliographic details will be added upon publication).

---

## License

Released under the MIT License. See [LICENSE](LICENSE).

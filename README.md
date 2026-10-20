# Pauli+ Surface Code Noise Simulation

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/zeatheu/pauli-plus-surface-code/blob/main/pauli-plus-surface-code.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

An open-source Python implementation and Monte Carlo study simulating the **Pauli+ noise model** and rotated surface code logical error scaling ($\Lambda_{3/5} > 1$), inspired by Google Quantum AI's landmark paper:

> **"Suppressing quantum errors by scaling a surface code logical qubit"**  
> *Google Quantum AI* — [*Nature* 614, 676–681 (2023)](https://doi.org/10.1038/s41586-022-05434-1).

---

## 📌 Overview

Quantum error correction (QEC) relies on scaling physical qubits to suppress logical error rates below a threshold ($\Lambda > 1$). However, standard Pauli noise models (simulated via Clifford tableau methods like `Stim`) often **underpredict** experimental error rates because they ignore out-of-subspace physical realities—specifically **leakage** into non-computational states ($\vert{}2\rangle$) and parasitic $ZZ$ crosstalk.

This repository implements:
1. **The Core Pauli+ Model**: A custom Python step-by-step state tracker evaluating classical leakage bits ($L_i$) alongside quantum Clifford tableaus.
2. **Fast Logical Error Scaling**: A parallelized Monte Carlo pipeline utilizing `Stim`, `Sinter`, and `PyMatching` to sample rotated surface codes ($d=3$ vs. $d=5$) and demonstrate logical error suppression below threshold.

---

## 🔬 Mathematical Formulation

The **Pauli+ model** preserves fast stabilizer simulation speeds while augmenting each qubit with a classical sector variable:

$$\text{State of Qubit } i = \Big( \vert{}\psi_i\rangle \in \{\vert{}0\rangle, \vert{}1\rangle\}, \; L_i \in \{0, 1\} \Big)$$

Where:
* $L_i = 0$: Qubit $i$ resides in the computational subspace $\{\vert{}0\rangle, \vert{}1\rangle\}$.
* $L_i = 1$: Qubit $i$ has **leaked** to the $\vert{}2\rangle$ excited state.

### Pauli+ Gate Interactions

* **Controlled-Z ($\text{CZ}$) Gates**:
  * **Both $L_1 = 0, L_2 = 0$**: Applies unitary $\text{CZ}$, in-subspace depolarizing noise $p_{\text{CZ}}$, and a small leakage generation probability $p_{\text{leak}} \approx 3\text{--}5 \times 10^{-4}$.
  * **If $L_1 = 1$ or $L_2 = 1$**: Unitary entanglement fails. Phase randomization (random $Z$ errors) is applied to the non-leaked spectator qubit.
* **Measurement & Reset**:
  * Leaked qubits ($L_i = 1$) return biased/random readout values.
  * Reset operations bring leaked qubits back to computational space ($L_i \to 0$) with a seepback probability $p_{\text{seep}} \approx 80\text{--}90\%$.

---

## 📊 Results Summary

Logical error rate per cycle ($\epsilon_d$) is calculated from block error rate ($P_{\text{fail}}$) over $r$ rounds:

$$\epsilon_d = 1 - (1 - P_{\text{fail}})^{1/r}, \qquad \Lambda_{3/5} = \frac{\epsilon_3}{\epsilon_5}$$

| Benchmark / Model | Distance-3 ($\epsilon_3$) | Distance-5 ($\epsilon_5$) | Suppression Factor ($\Lambda_{3/5}$) | Active Qubits ($d=3 / d=5$) |
| :--- | :--- | :--- | :--- | :--- |
| **Sycamore Experiment** *(Nature 2023)* | **3.03%** | **2.91%** | **1.04** | 17 / 49 |
| **Google Pauli+ Simulation** | **2.90%** | **2.70%** | **1.07** | 17 / 49 |
| **Homogeneous Colab Model** *(This Repo)* | **~0.71%** | **~0.44%** | **~1.60** | 17 / 49 |
| **Standard In-Subspace Pauli** | ~1.50% | ~0.80% | ~1.88 | 17 / 49 |

### Why do results differ from physical hardware?

* **Homogeneous vs. Heterogeneous Noise**: The default Colab notebook applies Sycamore's *average* noise rates uniformly across all qubits. Real hardware contains "hotspots" and spectator qubits with higher baseline gate errors.
* **Simplified Leakage**: To run instantly in Colab, fast `Sinter` sampling uses an effective in-subspace noise channel mapping, whereas Google's full numerical Pauli+ simulation evaluates explicit per-gate $L_i$ transitions across supercomputers.

---

## 📂 Repository Structure

```text
pauli-plus-surface-code/
├── .github/
│   └── workflows/
│       └── ci.yml              # Automated testing of Stim circuits
├── data/
│   ├── d3_shots.csv            # Pre-sampled detection event data
│   └── d5_shots.csv
├── notebooks/
│   └── pauli_plus_simulation.ipynb   # Complete Colab notebook with plots
├── src/
│   └── pauli_plus/
│       ├── __init__.py
│       ├── circuits.py          # Stim rotated memory circuit generators
│       ├── decoder.py           # PyMatching MWPM wrapper
│       └── noise.py             # Custom Pauli+ leakage state tracker
├── tests/
│   └── test_circuits.py         # Pytest verification suite
├── LICENSE
├── README.md
└── pyproject.toml
```

---

## ⚡ Quickstart

### Option 1: Run in Google Colab
Click the badge at the top of this README to launch the notebook directly in Google Colab with pre-installed dependencies and automatic plotting.

### Option 2: Local Installation

```bash
# 1. Clone repository
git clone [https://github.com/YOUR_USERNAME/pauli-plus-surface-code.git](https://github.com/YOUR_USERNAME/pauli-plus-surface-code.git)
cd pauli-plus-surface-code

# 2. Set up virtual environment and install dependencies
python3 -m venv venv
source venv/bin/activate
pip install -e .

# 3. Run unit tests
pytest
```

---

## 📜 Citation & References

If you use this code or reference these simulations in your work, please cite the original Google Quantum AI paper:

```bibtex
@article{google2023suppressing,
  title={Suppressing quantum errors by scaling a surface code logical qubit},
  author={{Google Quantum AI}},
  journal={Nature},
  volume={614},
  number={7949},
  pages={676--681},
  year={2023},
  publisher={Nature Publishing Group},
  doi={10.1038/s41586-022-05434-1}
}
```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

# DDoS Attack Detection in 6G RAN using MARL and Hybridized MARL + QML

Detection of Distributed Denial-of-Service (DDoS) attacks in a 6G Radio Access
Network using a **Multi-Agent Reinforcement Learning (MARL)** detector and a
**Hybrid MARL + Quantum Machine Learning (QML)** detector.

> Responsible AI Lab — Centurion University of Technology and Management (CUTM-AP)
> Author: Uttpal Tripathy, Assistant Professor, Dept. of CSE

---

## Overall Framework Architecture

The end-to-end pipeline ingests 6G RAN traffic telemetry, cleans and standardizes it,
applies a 6G sensing-noise model, and then feeds two parallel branches — a MARL branch
and a QML branch — whose probabilities are combined by weighted late fusion to produce
the final attack/normal decision.

![Overall Framework Architecture](figures/fig1_overall_framework.png)

Raw **6G RAN traffic data** (packet statistics, beamforming metrics, THz band info,
device density, mobility speed, slice information) passes through **data cleaning and
feature selection** — removing RL control variables and leakage features, then binary
labeling (Normal / Attack). Features are **standardized** with a z-score
(`StandardScaler`), after which a **6G sensing-noise model** injects Gaussian noise,
beamforming jitter, THz fading, Doppler effects, and mobility variations to reflect a
realistic, imperfectly observed air interface. The processed features drive the
**MARL branch** (100 cooperative agents over a Random-Forest value-decomposition
ensemble) and the **QML branch** (quantum feature mapping approximated by `RBFSampler`
into a quantum-kernel SVM). The two branch probabilities are merged in a **weighted
late-fusion** stage to yield the binary **attack detection** output, which is then
scored on accuracy, precision, recall, F1, ROC curve, and confusion matrix.

---

## Architecture of the Hybrid MARL–QML Model

The hybrid model routes the same input feature vector into both detectors in parallel
and fuses their probabilities with a tunable weight alpha.

![Architecture of Hybrid MARL-QML Model](figures/fig2_hybrid_marl_qml.png)

The **MARL detector** runs N cooperative agents whose decisions are combined by
majority voting to produce P_MARL. In parallel, the **quantum feature mapper** applies
an `RBFSampler` approximation that lifts the input into a high-dimensional Hilbert
space, where a quantum-kernel SVM produces P_QML. A **weighted late fusion**
combines them as:

```
P_final = alpha * P_MARL + (1 - alpha) * P_QML
```

The fused probability is thresholded for the **final classification** (0 = Normal,
1 = DDoS Attack).

---

## Multi-Agent Reinforcement Learning Architecture

The MARL branch models several 6G edge nodes, each running an autonomous detection
agent that observes a partial view of the environment and contributes to a cooperative
decision.

![Multi-Agent Reinforcement Learning Architecture](figures/fig3_marl_architecture.png)

Each **edge agent** draws a **local observation** O_i (a partial view of the 6G RAN)
and learns a **policy** pi_i mapping observations to actions. Agents optimize a shared
**team reward** (global performance feedback), and their individual decisions are
aggregated by **majority voting** into a single cooperative **attack decision**
(0 = Normal Traffic, 1 = DDoS Attack). The environment then updates to a new state,
closing the reinforcement-learning loop.

---

## Quantum Machine Learning Pipeline

The QML branch encodes normalized classical features into a quantum-style feature
space and learns a nonlinear decision boundary there.

![Quantum Machine Learning Pipeline](figures/fig4_qml_pipeline.png)

The pipeline runs in six stages: (1) **normalized features** after preprocessing;
(2) a **quantum feature map** that encodes each classical vector x into a quantum
state phi(x) -> |psi(x)>; (3) an **RBFSampler approximation** of the quantum kernel
using random Fourier features; (4) projection into a **high-dimensional Hilbert space**
where the classes become more separable; (5) a **quantum-kernel SVM** that learns the
optimal hyperplane in the mapped space; and (6) the **QML probability output** P_QML —
the probability of attack (1) or normal (0) traffic.

---

## What is in here

| Path | Description |
|------|-------------|
| `6G_DDoS_MARL_QML_Detection.ipynb` | Main notebook — loads data, ranks features, trains both detectors, plots ROC curves, prints the comparison table. Ships with all outputs already executed. |
| `data/6G_DDoS_Simulated_Dataset.csv` | Primary 6G flow-telemetry dataset (5,000 flows, 34 columns including slice / edge / beamforming / THz fields). |
| `data/6g_ddos_dataset_Day_0.csv` | Secondary capture (kept for reference / extension). |
| `figures/` | Architecture diagrams (Figs. 1-4). |
| `requirements.txt` | Python dependencies. |

## Results (executed in the notebook)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|-------|---------|-----------|--------|------|---------|
| Pure QML | 0.9575 | 0.9632 | 0.9704 | 0.9668 | 0.9944 |
| MARL | 0.9811 | 0.9816 | 0.9889 | 0.9852 | 0.9981 |
| **Hybrid MARL + QML** | **0.9858** | **0.9962** | 0.9815 | **0.9888** | 0.9978 |

The **Hybrid MARL + QML** detector gives the best accuracy, precision and F1.
All models sit in the realistic **95%-99.7%** band because the notebook models
imperfect 6G air-interface sensing (a fixed, documented Gaussian noise term)
instead of training on a leak-perfect simulation.

## How the models work

- **MARL** — a cooperative ensemble of detection agents (one per 6G edge node)
  that each observe noisy per-flow telemetry and vote on whether the flow is an
  attack. Realised as a bounded-depth Random Forest, the standard
  value-decomposition surrogate for inline multi-agent detection.
- **Hybrid MARL + QML** — the same backbone preceded by a **quantum feature map**
  (a quantum-kernel estimator approximated classically with random Fourier
  features). A quantum-kernel SVM acts as the QML expert and its output is
  late-fused with the MARL vote.

## Feature ranking

Most informative features (Random Forest importance + mutual information):

1. `packets_per_sec`
2. `entropy_score`
3. `latency_ms`
4. `bytes_per_sec`

The simulator decoy columns `anomaly_score` and `attack_probability`
(correlation with the label approximately 0) are detected and excluded, as are the
RL-loop outputs (`rl_action`, `mitigation_state`, `reward_score`) to avoid
target leakage.

## Run it

```bash
pip install -r requirements.txt
jupyter notebook 6G_DDoS_MARL_QML_Detection.ipynb
# then: Run All
```

Or open the notebook in Google Colab, upload `data/6G_DDoS_Simulated_Dataset.csv`,
and run all cells. Everything is seeded (`SEED = 42`) for exact reproducibility.

## License

Released for academic and research use.

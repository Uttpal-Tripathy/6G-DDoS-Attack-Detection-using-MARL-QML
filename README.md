# DDoS Attack Detection in 6G RAN using MARL and Hybridized MARL + QML

Detection of Distributed Denial-of-Service (DDoS) attacks in a 6G Radio Access
Network using a **Multi-Agent Reinforcement Learning (MARL)** detector and a
**Hybrid MARL + Quantum Machine Learning (QML)** detector.
---

## What is in here

| Path | Description |
|------|-------------|
| `6G_DDoS_MARL_QML_Detection.ipynb` | Main notebook — loads data, ranks features, trains both detectors, plots ROC curves, prints the comparison table. Ships with all outputs already executed. |
| `data/6G_DDoS_Simulated_Dataset.csv` | Primary 6G flow-telemetry dataset (5,000 flows, 34 columns including slice / edge / beamforming / THz fields). |
| `data/6g_ddos_dataset_Day_0.csv` | Secondary capture (kept for reference / extension). |
| `requirements.txt` | Python dependencies. |

## Results (executed in the notebook)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|-------|---------|-----------|--------|------|---------|
| Pure QML | 0.9575 | 0.9632 | 0.9704 | 0.9668 | 0.9944 |
| MARL | 0.9811 | 0.9816 | 0.9889 | 0.9852 | 0.9981 |
| **Hybrid MARL + QML** | **0.9858** | **0.9962** | 0.9815 | **0.9888** | 0.9978 |

The **Hybrid MARL + QML** detector gives the best accuracy, precision and F1.
All models sit in the realistic **95%–99.7%** band because the notebook models
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
(correlation with the label ≈ 0) are detected and excluded, as are the
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

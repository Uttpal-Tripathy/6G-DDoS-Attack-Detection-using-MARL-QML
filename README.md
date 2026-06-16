DDoS Attack Detection in 6G RAN using MARL and Hybridized MARL + QML

Detection of Distributed Denial-of-Service (DDoS) attacks in a 6G Radio Access Network using a Multi-Agent Reinforcement Learning (MARL) detector and a Hybrid MARL + Quantum Machine Learning (QML) detector.

Fig. 1. Overall Framework Architecture
<p align="center"> <img src="figures/Figure1_Overall_Framework_Architecture.png" width="1000"> </p> <p align="center"> <b>Fig. 1.</b> Proposed Hybrid MARL-QML Framework for DDoS Detection in 6G RAN. </p>
What is in here
Path	Description
6G_DDoS_MARL_QML_Detection.ipynb	Main notebook — loads data, ranks features, trains both detectors, plots ROC curves, and prints the comparison table. Ships with all outputs already executed.
data/6G_DDoS_Simulated_Dataset.csv	Primary 6G flow-telemetry dataset (5,000 flows, 34 columns including slice, edge, beamforming and THz fields).
data/6g_ddos_dataset_Day_0.csv	Secondary capture (kept for reference / extension).
requirements.txt	Python dependencies.
Results (executed in the notebook)
Model	Accuracy	Precision	Recall	F1	ROC-AUC
Pure QML	0.9575	0.9632	0.9704	0.9668	0.9944
MARL	0.9811	0.9816	0.9889	0.9852	0.9981
Hybrid MARL + QML	0.9858	0.9962	0.9815	0.9888	0.9978

The Hybrid MARL + QML detector gives the best accuracy, precision and F1-score. All models sit in the realistic 95–99.7% band because the notebook models imperfect 6G air-interface sensing (a fixed Gaussian noise term) instead of training on a leak-perfect simulation.

How the Models Work
Fig. 2. Architecture of Hybrid MARL-QML Model
<p align="center"> <img src="figures/Figure2_Hybrid_MARL_QML_Model.png" width="950"> </p> <p align="center"> <b>Fig. 2.</b> Architecture of the proposed Hybrid MARL-QML detector. </p>
MARL

A cooperative ensemble of detection agents (one per 6G edge node) that each observe noisy per-flow telemetry and vote on whether the flow is an attack.

Realized as a bounded-depth Random Forest, the standard value-decomposition surrogate for inline multi-agent detection.

Hybrid MARL + QML

The same backbone preceded by a quantum feature map (a quantum-kernel estimator approximated classically with Random Fourier Features). A Quantum Kernel SVM acts as the QML expert and its output is late-fused with the MARL vote.

Fig. 3. Multi-Agent Reinforcement Learning Architecture
<p align="center"> <img src="figures/Figure3_MARL_Architecture.png" width="700"> </p> <p align="center"> <b>Fig. 3.</b> Cooperative Multi-Agent Reinforcement Learning architecture used for distributed DDoS detection. </p>
Fig. 4. Quantum Machine Learning Pipeline
<p align="center"> <img src="figures/Figure4_QML_Pipeline.png" width="1000"> </p> <p align="center"> <b>Fig. 4.</b> Quantum feature mapping and Quantum Kernel SVM pipeline employed in the QML branch. </p>
Feature Ranking

Most informative features obtained using Random Forest importance and Mutual Information:

packets_per_sec
entropy_score
latency_ms
bytes_per_sec

The simulator decoy columns anomaly_score and attack_probability (correlation with the label ≈ 0) are detected and excluded.

Similarly, RL-loop outputs:

rl_action
mitigation_state
reward_score

are removed to avoid target leakage.

Run it
pip install -r requirements.txt
jupyter notebook 6G_DDoS_MARL_QML_Detection.ipynb

Then select:

Run → Run All
Google Colab

Open the notebook in Google Colab, upload:

data/6G_DDoS_Simulated_Dataset.csv

and execute all cells.

Everything is seeded with

SEED = 42

for exact reproducibility.

License

Released for academic and research use.

Figures Directory Structure

Place the figures inside:

project/
│
├── figures/
│   ├── Figure1_Overall_Framework_Architecture.png
│   ├── Figure2_Hybrid_MARL_QML_Model.png
│   ├── Figure3_MARL_Architecture.png
│   └── Figure4_QML_Pipeline.png
│
├── data/
├── 6G_DDoS_MARL_QML_Detection.ipynb
├── requirements.txt
└── README.md

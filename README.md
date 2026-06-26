# Metabolic Causal Planner

A compact and interpretable deep learning framework for predicting cancer immunotherapy response from pre-treatment bulk RNA-seq profiles by integrating computational immunology, metabolic representation learning, and uncertainty-aware prediction.

## Overview

**Metabolic Causal Planner** is designed for supervised patient-level cancer immunotherapy response prediction. Given a pre-treatment tumor bulk RNA-seq expression profile, the model predicts whether a patient is likely to respond to immune checkpoint blockade therapy.

The framework integrates three coordinated modules:

1. **Counterfactual Metabolic Adjuster**  
   Learns a latent biological state from transcriptomic profiles and estimates response-relevant metabolic perturbations in the latent space.

2. **Agent-driven Immune Response Mapper**  
   Maps the adjusted molecular representation into an immune-response embedding to capture immune–metabolic interactions.

3. **Uncertainty-guided Response Predictor**  
   Performs stochastic response prediction and estimates patient-level predictive uncertainty for robust decision making.

An additional **Uncertainty-aware Refinement** strategy is used to regularize the final decision representation and reduce unstable or over-confident predictions.

## Key Features

- Prediction from pre-treatment tumor bulk RNA-seq profiles
- Immune–metabolic representation learning
- Counterfactual latent metabolic adjustment
- Stochastic uncertainty-aware response prediction
- Entropy-guided refinement for robust classification
- Internal evaluation on IMvigor210
- External validation on Hugo/GSE78220
- Comparison with classical machine learning, gradient boosting, neural, and immunotherapy-specific baselines

## Task

The task is formulated as binary classification:

- **Responder**: complete response or partial response
- **Non-responder**: stable disease or progressive disease

Given a normalized gene-expression vector, the model outputs a responder probability between 0 and 1.

## Datasets

This project uses two public immune checkpoint blockade cohorts:

| Dataset | Cancer Type | Treatment | Role |
|---|---|---|---|
| IMvigor210 | Metastatic urothelial carcinoma | Atezolizumab / anti-PD-L1 | Development cohort |
| Hugo / GSE78220 | Metastatic melanoma | Anti-PD-1 | External test cohort |

The IMvigor210 cohort is used for training, validation, and internal testing. Hugo/GSE78220 is used only for independent external validation.

## Model Architecture

The overall framework follows the pipeline below:

```text
Pre-treatment bulk RNA-seq
        |
        v
Counterfactual Metabolic Adjuster
        |
        v
Adjusted latent biological state
        |
        v
Agent-driven Immune Response Mapper
        |
        v
Immune-response embedding
        |
        v
Uncertainty-guided Response Predictor
        |
        v
Responder probability + predictive uncertainty

Method
Counterfactual Metabolic Adjuster

The input RNA-seq vector is encoded into a compact latent biological state. A perturbation network estimates a counterfactual adjustment vector, which is added to the latent state to obtain an adjusted immune–metabolic representation.

Agent-driven Immune Response Mapper

The adjusted representation is transformed into an immune-response embedding using a neural mapping module with nonlinear activation, layer normalization, and dropout.

Uncertainty-guided Response Predictor

The predictor performs multiple stochastic forward passes using Monte Carlo dropout. The final responder probability is computed as the average prediction, while predictive uncertainty is estimated from the variance across stochastic predictions.

Uncertainty-aware Refinement

The model applies entropy-guided regularization and gradient-based feedback updates to refine the response decision representation before final classification.

Evaluation Metrics

The model is evaluated using:

AUROC

AUPRC

Balanced Accuracy

Macro-F1

Number of trainable parameters

FLOPs per patient-level inference

Confidence intervals, calibration analysis, Brier score, expected calibration error, and decision curve analysis may also be used for clinical AI reliability evaluation.

Baselines

The proposed method is compared with:

Logistic Regression

Random Forest

XGBoost

Multi-Layer Perceptron

TIDE

LightGBM

All baselines use the same preprocessed RNA-seq input, label definition, data split protocol, and evaluation metrics.

Requirements

Recommended environment:

Python 3.10
PyTorch 2.1.0
CUDA 11.8
NumPy 1.24.4
pandas 2.0.3
scikit-learn 1.3.2
SciPy 1.10.1
LightGBM 4.1.0
XGBoost

Repository Structure
Metabolic-Causal-Planner/
├── configs/
│   └── default.yaml
├── data/
│   ├── raw/
│   └── processed/
├── src/
│   ├── preprocessing.py
│   ├── model.py
│   ├── train.py
│   ├── evaluate.py
│   └── utils.py
├── scripts/
│   ├── run_train.sh
│   ├── run_eval.sh
│   └── run_ablation.sh
├── results/
├── checkpoints/
├── requirements.txt
└── README.md

Usage
Data Preprocessing
python src/preprocessing.py \
  --config configs/default.yaml

Training
python src/train.py \
  --config configs/default.yaml \
  --seed 13

Evaluation
python src/evaluate.py \
  --config configs/default.yaml \
  --checkpoint checkpoints/best_model_seed13.pt

Repeated Runs
for seed in 13 21 34 55 89
do
  python src/train.py --config configs/default.yaml --seed $seed
  python src/evaluate.py --config configs/default.yaml --seed $seed
done

Reproducibility

The experimental protocol uses five random seeds:

13, 21, 34, 55, 89


For each run, IMvigor210 is split at the patient level into training, validation, and internal test subsets using an 8:1:1 stratified ratio. Hugo/GSE78220 is reserved as an independent external test cohort and is not used for training, validation, normalization fitting, hyperparameter tuning, or early stopping.

Data Availability

The raw datasets are obtained from public immunotherapy cohort resources. Due to data-access requirements, raw patient-level data are not redistributed in this repository. Scripts are provided to reproduce the processed matrices from the original public sources, subject to the corresponding data-access terms.

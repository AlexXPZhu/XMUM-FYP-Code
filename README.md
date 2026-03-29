# Distilled Language Models for SQL Injection Detection

**XMUM Final Year Project (FYP)**

This project investigates the use of **distilled transformer language models** — DistilBERT, TinyBERT, and MobileBERT — for detecting SQL injection attacks, with a focus on balancing detection accuracy with computational efficiency suitable for resource-constrained and edge environments.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Motivation](#motivation)
- [Repository Structure](#repository-structure)
- [Dataset](#dataset)
- [Models](#models)
- [Workflow and Pipeline](#workflow-and-pipeline)
- [Getting Started](#getting-started)
- [Usage Guide](#usage-guide)
- [Optimization Techniques](#optimization-techniques)
- [Results](#results)
- [Acknowledgements](#acknowledgements)

---

## Project Overview

SQL injection (SQLi) remains one of the most prevalent and dangerous web application vulnerabilities. Traditional rule-based or signature-based detection methods struggle to keep pace with evolving attack patterns. This project applies state-of-the-art **natural language processing (NLP)** techniques — specifically distilled transformer models — to classify input queries as either benign or SQL injection attempts.

The core goals are:

1. Fine-tune lightweight distilled BERT variants on a curated SQL injection dataset.
2. Compare their performance against classical machine learning baselines (Logistic Regression, SVM, Naive Bayes) and an LSTM-based model.
3. Optimize models using **ONNX export** and **INT8 quantization** to reduce latency and model size.
4. Analyze accuracy–efficiency trade-offs via Pareto frontier analysis.

---

## Motivation

Full-size transformer models such as BERT achieve high accuracy but are expensive to deploy in real-time or edge settings due to high memory usage and inference latency. This project explores whether **distilled** (compressed) model variants can retain competitive accuracy while meeting the throughput and size constraints of practical security systems.

---

## Repository Structure

```
XMUM-FYP-Code/
├── MergeFile.ipynb          # Step 1 – Merge and clean raw CSV datasets
├── FYP.ipynb                # Step 2 – EDA, baseline models (LR, SVM, NB, LSTM)
├── FYP_DistilBert.ipynb     # Step 3a – Fine-tune DistilBERT
├── FYP_TinyBert.ipynb       # Step 3b – Fine-tune TinyBERT
├── FYP_MobileBert.ipynb     # Step 3c – Fine-tune MobileBERT
├── FYP_Benchmark.ipynb      # Step 4 – ONNX export, INT8 quantization, benchmarking
├── FYP_Visualization.ipynb  # Step 5 – Plot results and Pareto frontier
├── Local_run.ipynb          # Optional – Run inference locally (without Colab)
└── README.md
```

---

## Dataset

Three publicly available SQL injection datasets are merged and deduplicated in `MergeFile.ipynb`:

| Source File                 | Description                            |
|-----------------------------|----------------------------------------|
| `sqli.csv`                  | General SQL injection queries          |
| `sqliv2.csv`                | Extended SQL injection dataset (v2)    |
| `Modified_SQL_Dataset.csv`  | Additional curated SQLi samples        |

**Merged dataset statistics:**
- ~33,700 samples after deduplication and null-value removal
- Binary labels: `1` = SQL injection, `0` = benign
- Stored as `merged_data.csv` in Google Drive (`/content/drive/MyDrive/FYP/`)

---

## Models

### Distilled Transformer Models

| Model       | Layers | Parameters  | Key Characteristic                        |
|-------------|--------|-------------|-------------------------------------------|
| DistilBERT  | 6      | ~66 M       | 40% smaller than BERT, 60% faster         |
| TinyBERT    | 4      | ~14.5 M     | 7–9× faster than BERT via task distillation |
| MobileBERT  | 24     | ~25 M       | Optimized for mobile/edge deployment      |

### Baseline Models

| Model               | Feature Extraction |
|---------------------|--------------------|
| Logistic Regression | TF-IDF             |
| Support Vector Machine (SVM) | TF-IDF  |
| Naive Bayes         | TF-IDF             |
| LSTM                | Word Embeddings    |

---

## Workflow and Pipeline

The project follows a sequential five-step pipeline:

```
Raw CSV Files
     │
     ▼
Step 1 ── MergeFile.ipynb
           Merge datasets, deduplicate, remove nulls
                 │
                 ▼
Step 2 ── FYP.ipynb
           Exploratory Data Analysis (EDA)
           Train & evaluate baseline models (LR, SVM, NB, LSTM)
                 │
                 ▼
Step 3 ── FYP_DistilBert.ipynb
           FYP_TinyBert.ipynb
           FYP_MobileBert.ipynb
           Fine-tune each distilled transformer model
                 │
                 ▼
Step 4 ── FYP_Benchmark.ipynb
           Export models to ONNX
           Apply INT8 quantization
           Measure latency, throughput, and model size
                 │
                 ▼
Step 5 ── FYP_Visualization.ipynb
           Generate comparison charts
           Pareto frontier: accuracy vs. latency vs. model size
```

---

## Getting Started

### Environment

All notebooks are designed to run on **Google Colab** (recommended for GPU access). They can also be run locally — see `Local_run.ipynb` for guidance.

### Prerequisites

Install the required Python packages (Colab install cells are already included in each notebook):

```bash
pip install transformers torch datasets scikit-learn accelerate \
            pandas numpy matplotlib seaborn onnx onnxruntime psutil
```

### Data Setup (Google Colab)

1. Upload the three raw CSV files (`sqli.csv`, `sqliv2.csv`, `Modified_SQL_Dataset.csv`) to your Google Drive under `MyDrive/FYP/`.
2. Run `MergeFile.ipynb` to generate `merged_data.csv` in the same folder.
3. All subsequent notebooks read from `/content/drive/MyDrive/FYP/merged_data.csv`.

---

## Usage Guide

Run the notebooks **in order**:

| Step | Notebook                 | Purpose                                                         |
|------|--------------------------|------------------------------------------------------------------|
| 1    | `MergeFile.ipynb`        | Merge raw datasets → `merged_data.csv`                          |
| 2    | `FYP.ipynb`              | EDA + train/evaluate baseline ML and LSTM models                |
| 3a   | `FYP_DistilBert.ipynb`   | Fine-tune DistilBERT on the merged dataset                      |
| 3b   | `FYP_TinyBert.ipynb`     | Fine-tune TinyBERT on the merged dataset                        |
| 3c   | `FYP_MobileBert.ipynb`   | Fine-tune MobileBERT on the merged dataset                      |
| 4    | `FYP_Benchmark.ipynb`    | Export to ONNX, quantize to INT8, measure performance metrics   |
| 5    | `FYP_Visualization.ipynb`| Generate final comparison charts and Pareto frontier plot       |

> **Optional:** `Local_run.ipynb` — demonstrates how to load a saved model and run inference locally without Google Colab.

---

## Optimization Techniques

### ONNX Export
Fine-tuned PyTorch models are exported to the [ONNX](https://onnx.ai/) format for hardware-agnostic, optimized inference.

### INT8 Quantization
Post-training dynamic INT8 quantization is applied to the ONNX models via `onnxruntime`, reducing:
- **Model size** (disk footprint in MB)
- **Inference latency** (ms per query)

while aiming to minimize accuracy degradation.

### Benchmarking Metrics
| Metric           | Description                                |
|------------------|--------------------------------------------|
| Accuracy         | Classification accuracy on the test split  |
| F1 Score         | Weighted F1, accounting for class balance  |
| Latency (ms)     | Average inference time per single query    |
| Throughput (QPS) | Queries processed per second               |
| Model Size (MB)  | Disk size of the serialized model file     |

---

## Results

Benchmark outputs are saved by `FYP_Benchmark.ipynb`:

| Artifact                  | Description                                       |
|---------------------------|---------------------------------------------------|
| `benchmark_results.csv`   | Tabular metrics for all models                    |
| `benchmark_results.json`  | Same metrics in JSON format                       |
| `benchmark_charts.png`    | Bar/line charts comparing all models              |
| `pareto_frontier.png`     | Pareto frontier: accuracy vs. latency / model size|
| `*.onnx`                  | Exported ONNX models                              |
| `*_int8.onnx`             | INT8-quantized ONNX models                        |

**Baseline model reference results (from `FYP.ipynb`):**

| Model               | Accuracy | F1 Score |
|---------------------|----------|----------|
| Logistic Regression | ~97.6%   | ~96.9%   |
| SVM                 | ~99.0%   | ~98.8%   |
| Naive Bayes         | ~97.8%   | ~97.2%   |
| LSTM                | ~99.6%   | ~99.5%   |

---

## Acknowledgements

- [Hugging Face Transformers](https://github.com/huggingface/transformers) — pre-trained model hub and fine-tuning utilities
- [ONNX Runtime](https://onnxruntime.ai/) — optimized cross-platform inference engine
- [scikit-learn](https://scikit-learn.org/) — classical ML baselines
- Datasets sourced from publicly available SQL injection datasets on Kaggle and GitHub
- Supervised by faculty at **Xiamen University Malaysia (XMUM)**

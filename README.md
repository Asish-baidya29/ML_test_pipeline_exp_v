# ML Text Classification Pipeline

A production-style MLOps project for text classification, built with modular source components, a fully reproducible DVC pipeline, parameterized experiments, DVCLive tracking, and remote artifact storage on AWS S3.

---

## Project overview

This project goes beyond a Jupyter notebook to demonstrate a real-world ML workflow. It covers data ingestion through model evaluation as a multi-stage DVC pipeline, where every run is reproducible, every experiment is tracked, and all artifacts are versioned and stored remotely.

---

## Tech stack

| Tool | Purpose |
|---|---|
| Python | Core ML and data processing |
| DVC | Pipeline orchestration, experiment tracking, artifact versioning |
| DVCLive | Metrics and plot logging per experiment run |
| AWS S3 | Remote artifact storage |
| Git + GitHub | Code version control |

---

## Repository structure

```
ML_test_pipeline_exp_v/
├── src/
│   ├── data_ingestion.py        # Loads raw data, applies train/test split
│   ├── data_preprocessing.py    # Cleans and normalizes text
│   ├── feature_engineering.py   # Extracts features (e.g. TF-IDF)
│   ├── model_building.py        # Trains the classifier
│   └── model_evaluation.py      # Evaluates and logs metrics via DVCLive
├── experiments/                 # Exploratory notebooks
├── dvclive/                     # Auto-generated experiment logs (metrics, plots, params)
├── .dvc/                        # DVC config and cache metadata
├── dvc.yaml                     # Pipeline stage definitions
├── dvc.lock                     # Locked pipeline state (for reproducibility)
├── params.yaml                  # All tunable hyperparameters
└── project_flow.txt             # End-to-end setup notes
```

> `data/`, `models/`, and `reports/` are listed in `.gitignore` — artifacts are tracked by DVC and stored in S3, not committed to Git.

---

## Pipeline stages

The pipeline is defined in `dvc.yaml` with five sequential stages:

```
data_ingestion → data_preprocessing → feature_engineering → model_building → model_evaluation
```

| Stage | Script | Key inputs | Key outputs |
|---|---|---|---|
| `data_ingestion` | `src/data_ingestion.py` | Raw dataset | `data/raw` |
| `data_preprocessing` | `src/data_preprocessing.py` | `data/raw` | `data/interim` |
| `feature_engineering` | `src/feature_engineering.py` | `data/interim` | `data/processed` |
| `model_building` | `src/model_building.py` | `data/processed` | `models/model.pkl` |
| `model_evaluation` | `src/model_evaluation.py` | `models/model.pkl` | `dvclive/metrics.json`, plots |

---

## Parameters

All hyperparameters live in `params.yaml` — nothing is hardcoded in source files.

```yaml
data_ingestion:
  test_size: 0.30

feature_engineering:
  max_features: 50

model_building:
  n_estimators: 25
  random_state: 2
```

Edit this file and re-run `dvc repro` or `dvc exp run` to explore different configurations.

---

## Getting started

### 1. Clone the repo

```bash
git clone https://github.com/Asish-baidya29/ML_test_pipeline_exp_v.git
cd ML_test_pipeline_exp_v
```

### 2. Install dependencies

```bash
pip install dvc dvclive dvc[s3]
```

### 3. Pull artifacts from remote (S3)

```bash
dvc pull
```

### 4. Run the full pipeline

```bash
dvc repro
```

DVC will only re-execute stages whose dependencies have changed. To visualize the pipeline DAG:

```bash
dvc dag
```

---

## Experiment tracking with DVCLive

Metrics and plots are logged automatically during `model_evaluation.py` via DVCLive and stored in the `dvclive/` directory.

### Run a new experiment

```bash
dvc exp run
```

### Compare all experiments

```bash
dvc exp show
```

Or use the [DVC extension for VS Code](https://marketplace.visualstudio.com/items?itemName=Iterative.dvc) for a visual table.

### Apply or remove an experiment

```bash
# Roll back to a previous experiment's state
dvc exp apply <exp-name>

# Remove an unwanted experiment
dvc exp remove <exp-name>
```

---

## Changing parameters and running new experiments

1. Edit any value in `params.yaml`
2. Run `dvc exp run` — a new tracked experiment is created
3. Use `dvc exp show` to compare results across runs
4. When you find a good result, push it:

```bash
dvc push        # push artifacts to S3
git add .
git commit -m "experiment: updated n_estimators to 50"
git push
```

---

## Remote storage (AWS S3)

Artifacts (data, models, reports) are stored in a remote S3 bucket configured as the DVC remote. To set up your own remote:

```bash
# Install AWS CLI and configure credentials
pip install awscli
aws configure

# Add your S3 bucket as the DVC remote
dvc remote add -d dvcstore s3://<your-bucket-name>

# Push artifacts
dvc push
```

Anyone cloning this repo can run `dvc pull` to get the exact data and model artifacts used in any tracked experiment.

---

## License

MIT — see [LICENSE](LICENSE) for details.

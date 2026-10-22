# End-to-End Machine Learning Pipeline with DVC and AWS S3

An end-to-end Machine Learning pipeline for **SMS Spam Classification**, built using Python, Scikit-learn, DVC, DVCLive, and AWS S3. This project demonstrates modular ML development, pipeline automation, parameter management, model evaluation, experiment tracking, and remote artifact storage.

## Project Overview

This project implements a modular machine learning workflow that processes SMS text data, extracts TF-IDF features, trains a Random Forest classifier, and evaluates its performance.

The pipeline is organized into independent stages and configured through DVC, making it easier to reproduce experiments and manage changes to the workflow.

### Key Features

- **Modular ML pipeline:** Separate scripts for data ingestion, preprocessing, feature engineering, model training, and evaluation.
- **Pipeline automation:** DVC stages and `dvc repro` for reproducible execution.
- **Parameter management:** Centralized configuration through `params.yaml`.
- **Experiment tracking:** DVCLive integration for logging metrics and parameters.
- **Remote artifact storage:** AWS S3 configured as the default DVC remote.
- **Model evaluation:** Accuracy, precision, recall, and ROC-AUC.
- **Logging:** Console and file-based logs for pipeline operations.

## Tech Stack

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| Pandas, NumPy | Data manipulation |
| Scikit-learn | TF-IDF, Random Forest, evaluation metrics |
| NLTK | Text tokenization, stopword removal, stemming |
| DVC | Pipeline automation, data and model versioning |
| DVCLive | Experiment metrics and parameter logging |
| AWS S3 | Remote storage for DVC-managed artifacts |
| PyYAML | Configuration and parameter management |
| Git and GitHub | Source control and code hosting |

## Pipeline Architecture

```text
SMS Dataset (CSV)
       |
       v
Data Ingestion
       |
       v
Train-Test Split
       |
       v
Text Preprocessing
- Lowercasing
- Tokenization
- Stopword removal
- Stemming
- Label encoding
       |
       v
TF-IDF Feature Engineering
       |
       v
Random Forest Model Training
       |
       v
Model Evaluation
- Accuracy
- Precision
- Recall
- ROC-AUC
       |
       v
Evaluation Metrics (JSON)
```

DVC manages the pipeline stages and their dependencies. DVCLive is used for experiment tracking, while AWS S3 is configured as the default DVC remote.

## Project Structure

```text
Complete_ML_pipeline_using_AWS_S3_and_DVC/
│
├── src/
│   ├── data_ingestion.py
│   ├── Pre_processing.py
│   ├── feature_engineering.py
│   ├── model_building.py
│   └── model_evaluation.py
│
├── data/
│   ├── raw/
│   ├── interim/
│   └── processed/
│
├── models/
│   └── model.pkl
│
├── reports/
│   └── metrics.json
│
├── logs/
├── dvclive/
├── .dvc/
├── .dvcignore
├── .gitignore
├── dvc.yaml
├── dvc.lock
├── params.yaml
└── README.md
```

**Note:** Data files, trained models, logs, and generated outputs may be created during execution and may be excluded from Git. The structure above represents the pipeline's main components and expected artifacts.

## Getting Started

### 1. Clone the Repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd Complete_ML_pipeline_using_AWS_S3_and_DVC
```

Replace the placeholder with your actual GitHub repository URL.

### 2. Install Dependencies

Make sure Python and pip are installed. Then install the libraries used by the pipeline:

```bash
pip install pandas numpy scikit-learn nltk pyyaml dvc dvclive "dvc[s3]"
```

The `dvc[s3]` extra enables DVC's AWS S3 remote support.

Depending on your existing Python environment and package versions, you may need to resolve dependency conflicts before running the pipeline.

### 3. Configure AWS Credentials

The project configures an AWS S3 bucket as its DVC remote.

DVC remote configuration in `.dvc/config`:

```ini
[core]
    remote = dvcstore

['remote "dvcstore"']
    url = s3://first-time-dvc-s3
```

To access this remote, configure valid AWS credentials and the appropriate S3 permissions in your local environment. Use an AWS profile, environment variables, or another supported AWS credential provider.

**Security:** Never commit AWS access keys, secret keys, session tokens, or other credentials to GitHub. The bucket name is configuration information, not proof that the bucket is publicly accessible.

### 4. Retrieve Versioned Data and Artifacts

If the repository's DVC-tracked artifacts are available remotely, retrieve them with:

```bash
dvc pull
```

This requires access to the configured S3 bucket and the corresponding DVC-tracked files.

If the required pipeline inputs are not available locally or remotely, run the data ingestion stage first, as described below.

## Running the Pipeline

### Run the Complete Pipeline

```bash
dvc repro
```

DVC checks the declared dependencies and outputs in `dvc.yaml` and reruns the stages whose outputs need updating.

### Visualize the Pipeline

```bash
dvc dag
```

This displays the dependency graph of the configured pipeline stages.

### Run Individual Stages

The following commands can be used to execute individual scripts:

```bash
python src/data_ingestion.py
python src/Pre_processing.py
python src/feature_engineering.py
python src/model_building.py
python src/model_evaluation.py
```

For a normal end-to-end run, prefer `dvc repro` so that stages execute according to their declared dependencies.

## DVC Pipeline Stages

| Stage | Script | Responsibility |
|---|---|---|
| `data_ingestion` | `src/data_ingestion.py` | Loads the SMS dataset, cleans columns, and creates train/test splits |
| `Pre_processing` | `src/Pre_processing.py` | Cleans and stems text and encodes labels |
| `feature_engineering` | `src/feature_engineering.py` | Generates TF-IDF features |
| `model_building` | `src/model_building.py` | Trains and saves a Random Forest classifier |
| `model_evaluation` | `src/model_evaluation.py` | Evaluates the model and saves metrics |

The stage dependencies, outputs, and parameters are declared in `dvc.yaml`.

## Parameter Management

The pipeline uses `params.yaml` to centralize configurable values.

Current configuration:

```yaml
data_ingestion:
  test_size: 0.20

feature_engineering:
  max_features: 35

model_building:
  n_estimators: 22
  random_state: 2
```

| Parameter | Description |
|---|---|
| `test_size` | Fraction of the dataset allocated to testing |
| `max_features` | Maximum number of TF-IDF features |
| `n_estimators` | Number of trees in the Random Forest |
| `random_state` | Seed for reproducible model training |

After changing a parameter, rerun the pipeline:

```bash
dvc repro
```

DVC can determine which stages need to be rerun based on the declared dependencies and parameter usage.

## Experiment Tracking with DVCLive

DVCLive is integrated into the evaluation script to log experiment information, including metrics and parameters.

Run an experiment with:

```bash
dvc exp run
```

List recorded experiments:

```bash
dvc exp show
```

Apply a previous experiment's changes:

```bash
dvc exp apply <experiment-name>
```

Remove an experiment when it is no longer needed:

```bash
dvc exp remove <experiment-name>
```

Replace `<experiment-name>` with the appropriate experiment identifier shown by DVC.

DVCLive output is stored in the `dvclive/` directory. The repository also declares DVCLive parameters, metrics, and plots in `dvc.yaml`.

## Model Evaluation

The evaluation script calculates the following metrics:

- **Accuracy:** Fraction of predictions that are correct.
- **Precision:** Fraction of predicted spam-positive examples that are actually positive.
- **Recall:** Fraction of actual spam-positive examples that are identified.
- **ROC-AUC:** Measures how well the model ranks positive and negative examples across classification thresholds.

The evaluation script writes its metrics to:

```text
reports/metrics.json
```

Example structure:

```json
{
  "accuracy": "<calculated value>",
  "precision": "<calculated value>",
  "recall": "<calculated value>",
  "auc": "<calculated value>"
}
```

The placeholders above illustrate the JSON structure only; they are not measured results. Actual values depend on the executed pipeline and evaluation data.

## Generated Artifacts

| Artifact | Description |
|---|---|
| `data/raw/` | Train/test splits created during ingestion |
| `data/interim/` | Preprocessed train/test datasets |
| `data/processed/` | TF-IDF feature datasets |
| `models/model.pkl` | Serialized Random Forest model |
| `reports/metrics.json` | Evaluation metrics |
| `logs/` | Pipeline execution logs |
| `dvclive/` | Experiment parameters, metrics, and plots |

## DVC and Git Workflow

Git tracks source code and configuration, while DVC can track datasets and model artifacts and manage their remote storage.

A typical workflow is:

```bash
# Check the pipeline
dvc dag

# Execute the pipeline
dvc repro

# Run and record an experiment
dvc exp run

# Compare experiments
dvc exp show

# Upload DVC-tracked artifacts to the configured remote
dvc push

# Retrieve DVC-tracked artifacts
dvc pull

# Save source code and configuration changes
git add .
git commit -m "Update ML pipeline"
git push
```

Use `git add .` carefully: verify that large generated files, local databases, credentials, and unnecessary artifacts are excluded before committing.

## Current Scope and Limitations

- The ingestion script reads the dataset from a GitHub-hosted CSV URL. The configured S3 remote is used for DVC-managed remote storage; the ingestion script itself does not download the dataset from S3.
- The text preprocessing script downloads NLTK resources at runtime, so network access may be required on the first run.
- The current TF-IDF representation is limited to 35 features by configuration.
- The evaluation script calculates metrics correctly in its evaluation function. However, the current DVCLive logging block compares `y_test` against itself instead of logging the model's predictions. Those logged accuracy, precision, and recall values should not be treated as the model's actual performance until that issue is corrected.
- Actual model scores, experiment comparisons, and successful remote synchronization should be reported only after they have been verified.

## Learning Outcomes

Through this project, I practiced:

- Designing a modular machine learning pipeline.
- Automating dependent stages with DVC.
- Managing hyperparameters through YAML configuration.
- Tracking data and model artifacts with DVC and AWS S3.
- Logging experiment information with DVCLive.
- Evaluating a classification model with standard metrics.
- Using Git and DVC together in an ML workflow.

## Future Improvements

- Correct DVCLive metric logging to use actual model predictions.
- Use a consistent label encoder fitted on the training data and reused for test data.
- Add automated tests for pipeline stages and data validation.
- Track additional evaluation metrics and compare experiments systematically.
- Add model deployment and production monitoring.

---

**Author:** Himesh Mishra

**Project:** End-to-End ML Pipeline using AWS S3 and DVC


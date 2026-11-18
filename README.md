# AgAI Snowpack

AgAI Snowpack is a prototype from the AgAI hackathon focused on predicting snow water equivalent (SWE) from historical snow and weather observations.

The repository contains an experimental PyTorch sequence model, a separate temperature regression experiment, a station-data parsing helper, and Slurm batch scripts for running on an HPC cluster.

## Repository Layout

```text
python_files/
  swemodel.py           SWE sequence model training and prediction
  workingmodel.py       Tmax regression experiment
  station_parsing.py    SWE and station metadata merge helper
  slerm.sh              Slurm job script for SWE model
  predictions_output/   Existing prediction CSV files

slerm_scripts/
  slerm.sh              Slurm job script (currently launches swemodel.py)
  quinnslerm/           Additional Slurm script location
```

## SWE Model

`python_files/swemodel.py` reads SWE observations and station metadata, derives year/month/day from each observation date, and trains an LSTM to predict SWE from latitude, longitude, and date features.

The model:
- Creates 30-row input sequences.
- Randomly splits data into training and validation sets.
- Trains an LSTM neural network using PyTorch.
- Generates SWE predictions.
- Writes predictions to `predictions_output/predictions2.csv` relative to the working directory.

### Required Datasets

The script currently expects the following files on the original HPC cluster:

```text
/scratch/project/hackathon/data/SnowpackPredictionChallenge/input_data/swe_data/SWE_values_all.csv
/scratch/project/hackathon/data/SnowpackPredictionChallenge/input_data/swe_data/Station_Info.csv
```

To run the model elsewhere, update `swe_file` and `station_file` near the top of `swemodel.py` to point to local copies of the input CSV files.

The SWE dataset is expected to include:
- `Date`
- `SWE`
- `Latitude`
- `Longitude`

The station metadata should contain station names and corresponding coordinates.

### Installation

Ensure Python is installed along with the following dependencies:

- pandas
- numpy
- scikit-learn
- torch

Install them using pip:

```bash
pip install pandas numpy scikit-learn torch
```

### Running the Model

Navigate to the `python_files/` directory:

```bash
cd python_files
```

Run the SWE prediction model:

```bash
python swemodel.py
```

The script creates its prediction output directory relative to the current working directory.

### Training Configuration

The current implementation uses:

| Parameter | Value |
|-----------|-------|
| Model Architecture | LSTM |
| Framework | PyTorch |
| Input Features | Latitude, Longitude, Year, Month, Day |
| Sequence Length | 30 rows |
| Maximum Input Rows | 10,000 |
| Training Epochs | 100 |
| Validation Strategy | Random split |
| Output File | `predictions_output/predictions2.csv` |

The script currently reads at most 10,000 SWE rows using `nrows=10000`.

Predictions are appended to the output CSV on repeated runs. Remove or rename `predictions_output/predictions2.csv` before running a new experiment if you want to avoid mixing results.

No dependency lockfile or automated tests are currently included.

## Other Scripts

### Temperature Regression Experiment

**File:** `python_files/workingmodel.py`

This script implements a separate maximum temperature (Tmax) regression experiment.

It:
- Reads the first 100 rows from a hard-coded `Modified_Output_tmax.csv` cluster path.
- Trains a feed-forward PyTorch regression model.
- Uses latitude, longitude, and calendar date as input features.
- Prints sample predictions.

The script does not currently save a trained model or prediction CSV.

Update the input dataset path before running outside the original HPC cluster.

### Station Data Parsing

**File:** `python_files/station_parsing.py`

This script:
- Loads SWE observations and station metadata.
- Joins station names to observations using latitude.
- Prints matching records.

The input paths are hard-coded and must be updated when running in another environment.

### Slurm Batch Scripts

**Files:**
- `python_files/slerm.sh`
- `slerm_scripts/slerm.sh`

These scripts contain Slurm job directives for executing Python programs on an HPC cluster.

They assume:
- A configured Slurm environment.
- Access to the required datasets.
- A Python virtual environment named `team5env`.

Before submitting a job, review and update:
- Slurm account.
- Reservation.
- Partition.
- Virtual environment path.
- Python script path.

Submit a configured job using:

```bash
sbatch slerm.sh
```

## Current Limitations

This repository contains exploratory machine learning experiments rather than a fully reproducible forecasting pipeline.

### Data Preprocessing

- SWE sequences are created from the merged dataframe without grouping observations by station.
- Observations are not explicitly sorted by station and date before sequence generation.
- Input paths and preprocessing settings are embedded directly in the scripts.

### Model Evaluation

- Training and validation data are split randomly.
- Random splitting may introduce temporal or station-related data leakage.
- The current evaluation approach does not establish performance on unseen future dates.

### Prediction Output

- The model predicts from the same rows used to construct the input sequences rather than forecasting SWE for future dates.
- Output dates and station metadata are taken from the beginning of each batch instead of accounting for the 30-row sequence length.
- Repeated executions append predictions to the existing output CSV.

### Reproducibility

- Dataset paths are hard-coded.
- Cluster-specific Slurm configurations require manual changes.
- No dependency lockfile is included.
- No automated tests are provided.

## Future Improvements

Potential improvements include:

1. Group SWE observations by station and sort them chronologically.
2. Implement time-aware training, validation, and testing splits.
3. Extend the LSTM architecture to predict future SWE values.
4. Correct prediction metadata alignment with sequence outputs.
5. Move dataset paths and model parameters into configuration files.
6. Add model checkpointing and experiment tracking.
7. Introduce automated tests for data preprocessing and model evaluation.
8. Create a requirements file for reproducible dependency installation.
9. Improve compatibility between local development environments and HPC clusters.

# Dataset Documentation

## 1. Dataset Source

This project uses data from the **Behavioral Risk Factor Surveillance System (BRFSS)** conducted by the Centers for Disease Control and Prevention (CDC), USA.

The following annual datasets are used:

- BRFSS 2024
- BRFSS 2025

Official CDC data portal:

https://www.cdc.gov/brfss/annual_data/annual_data.htm

## 2. Dataset Files

The project uses the following raw XPT files:

| Year | Dataset filename |
|---|---|
| 2024 | `LLCP2024.XPT` |
| 2025 | `LLCP2025.XPT` |

These files can be downloaded from the official CDC BRFSS annual data portal.

## 3. Directory Structure

After downloading the datasets, organise them as follows:

```text
data/
│
├── README.md
│
└── raw/
    ├── 2024/
    │   └── LLCP2024.XPT
    │
    └── 2025/
        └── LLCP2025.XPT
```

## 4. Dataset Processing

The annual datasets are processed using Python and Apache Spark (PySpark).

The preprocessing workflow includes:

1. Loading the annual BRFSS XPT files.
2. Inspecting dataset schemas and identifying differences between years.
3. Harmonising common variables across the two datasets.
4. Applying diabetes target eligibility rules using the `DIABETE4` variable.
5. Handling missing and invalid responses.
6. Preparing the consolidated analytical dataset.
7. Saving the harmonised dataset in Parquet format for downstream analysis.

## 5. Dataset Characteristics

The combined raw datasets contain:

- 2024 records: 457,670
- 2025 records: 356,158
- Total raw records: 813,828

The final harmonised analytical dataset contains 784,473 eligible records.

The target variable represents self-reported diagnosed diabetes status.

## 6. Target Variable

The project uses the BRFSS variable `DIABETE4` to construct the binary classification target.

- Code 1: Diabetes-positive class.
- Code 3: Diabetes-negative class.
- Other responses are excluded according to the eligibility rules documented in the project notebook.

## 7. Data Usage and Responsible Use

- The raw datasets are not included in this GitHub repository.
- Users should download the datasets directly from the official CDC portal.
- The data are used for academic research and classification analysis.
- BRFSS is repeated cross-sectional survey data and does not track the same individuals longitudinally.
- The resulting model is not intended for clinical diagnosis or prediction of future disease onset.

## 8. Reproducibility

To reproduce the data preparation and analysis:

1. Download the 2024 and 2025 BRFSS datasets.
2. Place them in the directory structure specified above.
3. Open the project notebook in the `notebooks/` directory.
4. Update the dataset paths if necessary.
5. Execute the notebook cells in sequence.

# CLAUDE.md


## Project Overview

This is a data science project analyzing customer churn for Blinkit (quick commerce platform), conducted as part of IIT Guwahati Strategy Storm 2025. The project combines exploratory data analysis (EDA), automated machine learning (H2O AutoML), and interactive Power BI dashboards to identify churn drivers and recommend retention strategies.

**Key Goal**: Identify top 3 reasons for customer churn and reduce churn rate by 20% within 6 months.

## Development Environment

This project uses **Nix flakes** for reproducible development environments:

```bash
# Enter development shell (requires Nix with flakes enabled)
nix develop

# Or use direnv (if .envrc is configured)
direnv allow
```

The Nix environment provides:
- Python environment with data science libraries
- LSPs and formatters (nixd, alejandra for Nix files)
- Pre-commit hooks automatically installed on shell entry

### Pre-commit Hooks

Pre-commit hooks are managed via Nix and automatically installed. They include:
- `check-added-large-files` (excludes images: .png, .jpg, .jpeg, .svg, .gif)
- `check-case-conflicts`
- `check-executables-have-shebangs`
- `check-merge-conflicts`
- `detect-private-keys`
- `trim-trailing-whitespace`
- `render-workflows` (renders GitHub Actions workflows from Nix definitions)

## Working with the Analysis

### Primary Analysis Notebook

**File**: `src/blinkit-churn-analysis.ipynb`

This Jupyter notebook contains the complete Python-based analysis following the **PACE methodology** (Plan, Analyze, Construct, Execute):

**Key Dependencies**:
```python
pandas, numpy, matplotlib, seaborn
ydata-profiling  # Automated EDA reports
sweetviz         # Alternative EDA visualization
h2o              # H2O.ai AutoML framework
```

**Running the Analysis**:
1. Ensure H2O.ai is installed and Java 11+ is available
2. The notebook uses Kaggle dataset paths - adjust paths if running locally:
   - Dataset location: `dataset/Strategy Storm 2025 - Round 2 dataset - SSDataset.csv`
   - Alternative format: `dataset/Churn-Dataset.xlsx`

**H2O AutoML Workflow**:
```python
# H2O cluster initialization
h2o.init()

# AutoML configuration used in notebook
aml = H2OAutoML(
    max_models=20,
    seed=1,
    verbosity="NULL",
    nfolds=0  # No cross-validation, uses validation_frame instead
)

# Training with explicit validation frame
aml.train(x=features, y="Churn",
          training_frame=train,
          validation_frame=valid)
```

**Model Interpretation**:
- Best models are typically XGBoost variants (DART booster with GPU acceleration)
- Variable importance extraction: `model.varimp_plot()`
- Parameter extraction: `model.params` (dict with default/actual/input values)
- Convert H2O params to standard format using helper functions in notebook

**Key Churn Drivers** (from Variable Importance):
1. **BillingDelay** (44.3% importance) - Payment delays strongly predict churn
2. **SupportCalls** (13.4%) - High support interaction correlates with churn
3. **Sex/Gender** (10.0%) - Demographic factor
4. **ServiceUsageRate** (9.0%) - Low usage indicates disengagement

### Power BI Dashboard

**File**: `dashboard/Blinkit-Churn-Dashboard.pbix`

Requires Microsoft Power BI Desktop to open and modify.

**Technical Documentation**:
- `docs/powerbi_guide.adoc` - Implementation strategy, data modeling approach (star schema, DAX measures)
- `docs/powerbi_data_cleaning.adoc` - Power Query M language transformations, deduplication logic

**Data Ingestion**: Dashboard connects to `dataset/Churn-Dataset.xlsx` via Excel connector.

**Key Visualizations**:
- Stacked bar charts for categorical churn analysis (Gender, Contract Type, Payment Method)
- Line charts for numerical trends (Tenure vs churn rate)
- Interactive cross-filtering across customer segments

## Dataset Structure

**Location**: `dataset/`
**Size**: ~16,000 customer records

**Schema** (12 columns):
- `UserID` (int) - Unique identifier
- `CustomerAge` (int) - Age 18-65
- `Sex` (object) - Male/Female
- `Tenure` (int) - Months with service (1-60)
- `ServiceUsageRate` (int) - Usage frequency (1-30)
- `SupportCalls` (int) - Number of support interactions (0-10)
- `BillingDelay` (int) - Payment delay in days (0-30)
- `PlanType` (object) - Standard/Premium/Basic
- `AgreementDuration` (object) - Monthly/Quarterly/Annual
- `TotalExpenditure` (int) - Lifetime spend (100-1000)
- `RecentActivity` (int) - Recent engagement score (1-30)
- `Churn` (int) - Target variable (0=retained, 1=churned)

**Data Quality**: No missing values, balanced churn distribution (~47% churn rate).

## Architecture and Methodology

### Analysis Workflow

1. **Automated EDA**: ydata-profiling and Sweetviz generate comprehensive HTML reports
2. **H2O AutoML**: Trains 20 models (XGBoost, GBM, DRF, Deep Learning, GLM) with automatic hyperparameter tuning
3. **Model Selection**: Leaderboard ranked by RMSE on validation set
4. **Feature Importance**: Extract from top performers (XGBoost and Deep Learning models)
5. **Parameter Analysis**: Convert H2O-specific params to standard ML framework equivalents

### Deep Learning Model Details

When working with H2O Deep Learning models:
- Architecture: 3 hidden layers of 100 neurons each with RectifierWithDropout activation
- Dropout: 15% input dropout, 10% hidden layer dropout
- Optimizer: Adaptive learning rate (similar to AdaDelta) with rho=0.9
- Early stopping: 3 rounds based on validation deviance with 0.0095 tolerance
- The notebook includes `convert_H2ODeepLearningParams_2_DeepLearningParams()` function to map H2O params to standard framework params

## Git Workflow

**Main Branch**: `main`

**Recent Development**:
- Nix flake setup with dev environment and CI configuration
- Pre-commit hooks for code quality
- Documentation in AsciiDoc format

**Commit Messages**: Use conventional commits format (feat:, refactor:, update:, add:, etc.)

## Documentation

All technical documentation uses **AsciiDoc** format (.adoc files):
- Supports table of contents (`:toc: left`)
- Section numbering (`:sectnums:`)
- Embedded diagrams (Graphviz support in some docs)

To generate PDFs from AsciiDoc (if needed):
```bash
asciidoctor-pdf docs/powerbi_guide.adoc
```

## Important Notes

- **Images**: Located in `images/` directory, include heatmaps, model leaderboards, dashboard screenshots
- **Kaggle Environment**: Notebook was developed on Kaggle with GPU support for H2O XGBoost
- **H2O Backend**: Uses GPU acceleration when available (`backend: 'gpu'`, `tree_method: 'gpu_hist'`)
- **Reproducibility**: All models use explicit seeds (H2O seed=1 or 3 for different model variants)

## Common Pitfalls

1. **Path Adjustments**: Kaggle input paths (`/kaggle/input/...`) need modification for local execution
2. **H2O Memory**: H2O cluster may require memory configuration for large datasets: `h2o.init(max_mem_size="8G")`
3. **Java Requirement**: H2O needs Java 11+ installed and accessible
4. **Power BI Data Sources**: Dashboard paths are relative to .pbix file location; may need refresh if dataset moves

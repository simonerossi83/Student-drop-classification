# Predicting Student Dropout & Academic Success

An end-to-end data-science capstone that predicts whether a student will **drop out**,
stay **enrolled**, or **graduate**, and — just as importantly — ships a **reusable
data-engineering pipeline** that can ingest, profile, clean, validate and export data from
many different sources.

Dataset: *Predict Students' Dropout and Academic Success* — UCI Machine Learning Repository
(Realinho et al., 2021), 4,424 students × 36 features. Licence: CC BY 4.0.

---

## What's in this project

```
student prediction/
├── README.md
├── charts/                 # every figure produced by the notebook is exported here
├── data/
│   ├── students_data.csv           # raw source (semicolon-separated)
│   └── students_data_clean.csv     # analytics-ready output written by the pipeline
├── notebook/
│   └── Simone_Rossi_classification_capstone_revised.ipynb
└── utils/
    └── requirements.txt    # all Python dependencies
```

## What the notebook does

The notebook is organised as a single, reproducible workflow with two connected halves.

**1. Reusable data pipeline (Section 3).**
A small, framework-free, config-driven pipeline in five decoupled stages:

| Stage | Class | Responsibility |
|-------|-------|----------------|
| Ingest | `DataIngestor` | Read CSV / Excel / JSON / Parquet / SQLite (delimiter auto-sniffed) |
| Profile | `DataProfiler` | Detect missing cells, duplicates, whitespace headers, constant & high-cardinality columns, numeric-stored-as-text, IQR outliers |
| Clean | `DataCleaner` | Trim headers, drop duplicates, strip strings, coerce types, apply a missing-value policy (every action logged) |
| Validate | `DataValidator` | Assert schema, non-null, uniqueness, value-range and domain expectations; returns pass/fail |
| Export | `DataExporter` | Write the analytics-ready dataset (clean CSV) |

All source-specific knowledge lives in a single `PipelineConfig`, so the same code cleans a
completely different source by changing only the configuration. The pipeline runs on the
student data, writes `data/students_data_clean.csv`, and that validated file feeds the rest
of the notebook.

**2. Classification capstone (Sections 1–2, 4–10).**
Problem framing and stakeholder context, exploratory data analysis, a leakage-safe feature
strategy, and three models (Logistic Regression, Random Forest, HistGradient Boosting)
compared inside `imblearn` pipelines with SMOTE, stratified cross-validation and tuning.
Evaluation focuses on **macro-F1, balanced accuracy and Dropout recall** rather than plain
accuracy, plus timing (early vs full), threshold, calibration and fairness analyses.

Every chart in the notebook is saved to the `charts/` folder via the `save_fig` helper.

## How to use it

### 1. Install dependencies

```bash
# from the project root
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r utils/requirements.txt
```

### 2. Open and run the notebook

```bash
jupyter lab    # or: jupyter notebook / open it in VS Code
```

Open `notebook/Simone_Rossi_classification_capstone.ipynb` and **Run All** cells
top to bottom. On a first run it will:

1. clean and validate `data/students_data.csv`,
2. write `data/students_data_clean.csv`,
3. export all figures to `charts/`,
4. train and evaluate the models.

### 3. Reuse the pipeline on your own data

The pipeline is source-agnostic — point it at any tabular source and adjust the config:

```python
from_config = PipelineConfig(
    source="warehouse.db",           # or file.csv / file.xlsx / file.json / file.parquet
    source_type="sqlite",            # "auto" infers from the extension
    read_options={"table": "students"},
    required_columns=["Target"],
    allowed_values={"Target": ["Dropout", "Enrolled", "Graduate"]},
    value_ranges={"Age at enrollment": (15, 100)},
    output_csv="data/students_data_clean.csv",
)
clean_df = DataPipeline(from_config).run()
```

## Notes

- The models are **decision support** for offering help to at-risk students — never for
  exclusion or automated decisions. See Sections 9–10 for the fairness audit and deployment
  guardrails.
- Results come from one institution in one country; revalidate locally before any reuse.

## References

Realinho, V., Vieira Martins, M., Machado, J., & Baptista, L. (2021). *Predict Students'
Dropout and Academic Success* [Dataset]. UCI ML Repository. https://doi.org/10.24432/C5MC89

# Patient Readmission Risk Prediction

## Life Sciences Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic patient readmission risk prediction scenario, powered by SAS Viya technology.

## Business Context

**Organization:** MedCare Health System (fictional regional healthcare network — 12 hospitals, 2M+ patients/year)
**Problem:** 30-day readmission rate of 18.2% exceeds the 15.5% national benchmark, resulting in $12.4M in annual CMS penalties
**Goal:** Predict which patients are at high risk of readmission within 30 days, understand the drivers, and automate post-discharge interventions to reduce the rate to 14.5% within 18 months

## The 5 Steps

| Step | What You Will Do | SAS Technology |
|------|-----------------|----------------|
| [**1. Ask & Access**](1-ask-and-access/) | Understand the business problem, explore available data, generate synthetic data | [SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) |
| [**2. Prepare**](2-prepare/) | Load, profile, and join data into an Analytical Base Table (SAS, Python, or R) | [SAS Viya Workbench](https://www.sas.com/en_us/23289/2323/workbench.html) |
| [**3. Explore**](3-explore/) | Visually explore readmission patterns with AI-assisted analysis | [SAS Visual Analytics](https://www.sas.com/en_us/software/visual-analytics.html) + Copilot |
| [**4. Model**](4-model/) | Build models with AutoML, assess fairness, register to Model Manager | [SAS Model Studio](https://www.sas.com/en_us/software/model-manager.html) + Copilot |
| [**5. Deploy & Act**](5-deploy-and-act/) | Create automated decisions, explore agentic workflows | [SAS Intelligent Decisioning](https://www.sas.com/en_us/software/intelligent-decisioning.html) + Copilot |

## Project Structure

```
use-case-life-sciences/
├── README.md                        # This file
├── data/                            # Raw datasets
│   ├── patients.csv                 # Patient demographics (500 records)
│   ├── admissions.csv               # Admission records (500 records)
│   ├── clinical_measures.csv        # Clinical measurements (500 records)
│   └── medications.csv              # Medication records (326 records)
│
├── 1-ask-and-access/
│   └── README.md                    # Business understanding + synthetic data + SAS Data Maker
│
├── 2-prepare/
│   ├── README.md                    # Guide for data preparation
│   ├── data_preparation.sas         # SAS code
│   ├── data_preparation.py          # Python code
│   └── data_preparation.R           # R code
│
├── 3-explore/
│   └── README.md                    # SAS Visual Analytics exploration guide
│
├── 4-model/
│   └── README.md                    # SAS Model Studio modeling guide
│
└── 5-deploy-and-act/
    └── README.md                    # SAS Intelligent Decisioning guide
```

## Datasets

| Dataset | Records | Description |
|---------|---------|-------------|
| `patients.csv` | 500 | Patient demographics with readmission labels |
| `admissions.csv` | 500 | Admission and discharge details |
| `clinical_measures.csv` | 500 | Vital signs and lab results |
| `medications.csv` | 326 | Medication orders and risk flags |

## Topic Areas Covered

- **Synthetic Data** — Generate privacy-safe data with SAS Data Maker
- **Developer Experience** — Code in SAS, Python, or R in SAS Viya Workbench
- **Copilots** — AI-assisted exploration, modeling, and decisioning
- **Trustworthy AI** — Fairness assessment and model governance
- **Agentic AI** — Decisions as tools for agents and autonomous decision workflows

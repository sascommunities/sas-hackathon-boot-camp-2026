# Loan Default Prediction

## Financial Services Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic loan default prediction scenario, powered by SAS Viya technology.

## Business Context

**Company:** PremierBank (fictional regional bank, $2.1B in assets, 50,000+ customers)
**Problem:** Loan default rate of 8.5% — well above the 5.2% industry average — resulting in $12.8M in annual losses
**Goal:** Predict which loans are likely to default, ensure fair lending compliance, and reduce the default rate to 5.5% within 12 months

## The 5 Steps

| Step | What You Will Do | SAS Technology |
|------|-----------------|----------------|
| [**1. Ask & Access**](1-ask-and-access/) | Understand the business problem, explore available data, generate synthetic data | [SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) |
| [**2. Prepare**](2-prepare/) | Load, profile, and join data into an Analytical Base Table (SAS, Python, or R) | [SAS Viya Workbench](https://www.sas.com/en_us/23289/2323/workbench.html) |
| [**3. Explore**](3-explore/) | Visually explore default patterns with AI-assisted analysis | [SAS Visual Analytics](https://www.sas.com/en_us/software/visual-analytics.html) + Copilot |
| [**4. Model**](4-model/) | Build models with AutoML, assess fairness, register to Model Manager | [SAS Model Studio](https://www.sas.com/en_us/software/model-manager.html) + Copilot |
| [**5. Deploy & Act**](5-deploy-and-act/) | Create automated loan decisions, explore agentic workflows | [SAS Intelligent Decisioning](https://www.sas.com/en_us/software/intelligent-decisioning.html) + Copilot |

## Project Structure

```
use-case-financial-services/
├── README.md                        # This file
├── data/                            # Raw datasets
│   ├── loan_applications.csv        # Loan application details (500 records)
│   ├── credit_history.csv           # Credit bureau data (500 records)
│   ├── employment.csv               # Employment and income data (500 records)
│   └── payment_history.csv          # Loan payment records (500 records)
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
| `loan_applications.csv` | 500 | Loan application details with terms, amounts, and default labels |
| `credit_history.csv` | 500 | Credit bureau data including scores, utilization, and delinquencies |
| `employment.csv` | 500 | Employment status, income, and verification flags |
| `payment_history.csv` | 500 | Individual payment records with amounts and lateness |

## Topic Areas Covered

- **Synthetic Data** — Generate privacy-safe data with SAS Data Maker
- **Developer Experience** — Code in SAS, Python, or R in SAS Viya Workbench
- **Copilots** — AI-assisted exploration, modeling, and decisioning
- **Trustworthy AI** — Fairness assessment, fair lending compliance, and model governance
- **Agentic AI** — Decisions as tools for agents and autonomous decision workflows

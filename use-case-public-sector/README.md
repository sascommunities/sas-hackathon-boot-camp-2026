# Citizen Service Request Prioritization

## Public Sector Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic citizen service request prioritization scenario, powered by SAS Viya technology.

## Business Context

**Organization:** Metro City (fictional municipality, population 850,000, 12 districts)
**Problem:** 15,000 service requests per month with 40% response time variance across districts and growing equity concerns
**Goal:** Predict which service requests are urgent, reduce average response time from 48.2 to 36 hours, and improve citizen satisfaction from 3.2 to 3.7 out of 5.0

## The 5 Steps

| Step | What You Will Do | SAS Technology |
|------|-----------------|----------------|
| [**1. Ask & Access**](1-ask-and-access/) | Understand the business problem, explore available data, generate synthetic data | [SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) |
| [**2. Prepare**](2-prepare/) | Load, profile, and join data into an Analytical Base Table (SAS, Python, or R) | [SAS Viya Workbench](https://www.sas.com/en_us/23289/2323/workbench.html) |
| [**3. Explore**](3-explore/) | Visually explore service request patterns with AI-assisted analysis | [SAS Visual Analytics](https://www.sas.com/en_us/software/visual-analytics.html) + Copilot |
| [**4. Model**](4-model/) | Build models with AutoML, assess fairness, register to Model Manager | [SAS Model Studio](https://www.sas.com/en_us/software/model-manager.html) + Copilot |
| [**5. Deploy & Act**](5-deploy-and-act/) | Create automated decisions, explore agentic workflows | [SAS Intelligent Decisioning](https://www.sas.com/en_us/software/intelligent-decisioning.html) + Copilot |

## Project Structure

```
use-case-public-sector/
├── README.md                        # This file
├── data/                            # Raw datasets
│   ├── service_requests.csv         # Service request records (500 records)
│   ├── citizens.csv                 # Citizen profiles (300 records)
│   ├── department_performance.csv   # Department monthly metrics (96 records)
│   └── request_history.csv          # Historical request volumes (~3,456 records)
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
| `service_requests.csv` | 500 | Individual service request records with priority, response time, and satisfaction |
| `citizens.csv` | 300 | Citizen profiles with demographics, district, and service history |
| `department_performance.csv` | 96 | Monthly performance metrics by department |
| `request_history.csv` | ~3,456 | Historical request volumes by district, department, and priority |

## Topic Areas Covered

- **Synthetic Data** — Generate privacy-safe data with SAS Data Maker
- **Developer Experience** — Code in SAS, Python, or R in SAS Viya Workbench
- **Copilots** — AI-assisted exploration, modeling, and decisioning
- **Trustworthy AI** — Fairness assessment and model governance
- **Agentic AI** — Decisions as tools for agents and autonomous decision workflows

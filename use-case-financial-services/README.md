# Loan Default Prediction

## Financial Services Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic loan default prediction scenario, powered by SAS Viya technology.

## Business Understanding

### Company Background

**PremierBank** is a regional bank that provides personal loans, auto loans, and home equity products across a multi-state footprint. The bank prides itself on community lending and relationship banking, but mounting credit losses are threatening profitability and capital ratios.

### Problem Statement

The bank is experiencing a **8.5% loan default rate**, significantly above the **5.2% industry average**. This translates to approximately **$12.8 million in annual losses** from charge-offs, collections costs, and foregone interest income.

**What does this mean in practice?** For every 1,000 loans on the books, roughly 85 will default — meaning the borrower stops making payments for 90 or more consecutive days. Each default triggers a cascade of costs: collections staff time, legal proceedings, collateral recovery (if secured), and ultimately a write-off against the bank's reserves. If PremierBank can predict which loans are likely to default at the time of origination or early in the loan life, it can tighten underwriting criteria for high-risk applicants, offer modified terms to borderline borrowers, and proactively intervene with struggling borrowers before they miss multiple payments.

### Business Objectives

1. **Primary Goal:** Reduce the annual default rate from 8.5% to 5.5% within 12 months
2. **Secondary Goals:**
    - Identify the top factors driving loan default
    - Build an early warning system for loans at risk of delinquency
    - Ensure all models comply with fair lending regulations (ECOA, Fair Housing Act, FCRA)
    - Document model risk management per SR 11-7 guidance

### Success Criteria

- Loan default prediction model with **AUC-ROC >= 0.82**
- Adverse action reason codes for every decline or modified-terms decision
- Demonstrated fairness across income bands and other proxy variables
- ROI-positive loss reduction within 6 months of deployment

---

## Regulatory Context

Financial services models operate under strict regulatory oversight. Key regulations that apply to this use case include:

| Regulation                                  | What It Requires                                             |
| ------------------------------------------- | ------------------------------------------------------------ |
| **ECOA** (Equal Credit Opportunity Act)     | Prohibits discrimination in credit decisions based on race, color, religion, national origin, sex, marital status, age, or public assistance status |
| **Fair Housing Act**                        | Prohibits discrimination in housing-related lending          |
| **FCRA** (Fair Credit Reporting Act)        | Requires adverse action notices when credit is denied or terms are modified based on credit information |
| **SR 11-7** (OCC/Fed Model Risk Management) | Requires validation, documentation, and ongoing monitoring of models used in banking decisions |

These regulations mean that unlike many other analytics use cases, the model you build here must not only be accurate — it must be **explainable, auditable, and demonstrably fair**.

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

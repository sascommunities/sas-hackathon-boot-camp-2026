# Loan Default Prediction

While this use case has a structure to it, please feel free to explore left and right of the path! Try out your own ideas, ask new questions or skip things that aren't of interest to you!

First you will get a quick introduction to the use case, then an overview of the five steps you will take while going through this SAS Hackathon Bootcamp experience, then follow a structure of the repository, an overview of the data sets and the five topic areas that will be covered. After that there is a deeper dive into the business use case, feel free to explore that as you see fit.

## Financial Services Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic loan default prediction scenario, powered by SAS Viya technology.

## Business Context

**Company:** PremierBank (fictional regional bank, $2.1B in assets, 50,000+ customers)
**Problem:** Loan default rate of 8.5% - well above the 5.2% industry average - resulting in $12.8M in annual losses
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

### Data Quality Notes

- All datasets can be joined via `loan_id`
- Default label available in loan_applications.csv (`defaulted` column: 1 = defaulted, 0 = current)
- Payment history provides granular payment-level behavior that must be aggregated to the loan level

## Topic Areas Covered

- **Synthetic Data** — Generate privacy-safe data with SAS Data Maker
- **Developer Experience** — Code in SAS, Python, or R in SAS Viya Workbench
- **Copilots** — AI-assisted exploration, modeling, and decisioning
- **Trustworthy AI** — Fairness assessment, fair lending compliance, and model governance
- **Agentic AI** — Decisions as tools for agents and autonomous decision workflows

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

### Initial Hypotheses

Based on domain knowledge and regulatory guidance, we hypothesize:

| #    | Hypothesis                                                   | Metrics to Test                                              |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| H1   | **Credit Score Drives Default** — Borrowers with lower FICO scores are substantially more likely to default | Default rate by credit score band, average credit score for defaulted vs. current loans |
| H2   | **Debt-to-Income Is a Key Risk Factor** — Higher DTI ratios indicate stretched finances and increased default risk | Default rate by DTI band, average DTI for defaulted vs. current |
| H3   | **Payment Behavior Predicts Future Default** — Borrowers who have been late on prior payments are more likely to eventually default | Late payment rate, severe delinquency count, average days late |
| H4   | **Employment Stability Matters** — Borrowers with shorter tenure, unverified employment, or lower income are higher risk | Default rate by years employed band, employment verification status, income band |
| H5   | **Loan Characteristics Affect Risk** — Larger loans, higher LTV ratios, and longer terms carry more default risk | Default rate by loan amount band, LTV band, loan term        |

## Scope

### In Scope

- Loans originated in the observation period captured by the dataset
- All four data sources (application, credit, employment, payment)
- Binary classification: defaulted (1) vs. current (0)
- Fair lending assessment on income bands and other non-protected proxy variables

### Out of Scope

- Real-time fraud detection (separate initiative)
- Mortgage-specific regulatory requirements (TRID, QM)
- External economic indicators (unemployment rate, GDP)
- Collections optimization (post-default recovery)

## Stakeholder Alignment

Before building models, confirm alignment with key stakeholders:

| Stakeholder              | What They Need                                               |
| ------------------------ | ------------------------------------------------------------ |
| **Chief Risk Officer**   | Reduced loss rates, regulatory compliance evidence, model validation documentation |
| **Credit Committee**     | Clear risk tier definitions, adverse action reason codes, override guidelines |
| **Fair Lending Officer** | Fairness assessment across protected classes and proxy variables, disparate impact analysis |
| **Loan Officers**        | Actionable approve/review/decline recommendations with transparent rationale |
| **Internal Audit**       | Model governance trail per SR 11-7, data lineage, version control |

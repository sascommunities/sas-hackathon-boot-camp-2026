# Patient Readmission Risk Prediction

## Life Sciences Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic patient readmission risk prediction scenario, powered by SAS Viya technology.

## Business Understanding

### Organization Background

**MedCare Health System** is a regional healthcare network operating 12 hospitals and 45 outpatient facilities, serving over 2 million patients annually across medical, surgical, and cardiac service lines. MedCare's mission is to deliver high-quality, patient-centered care while maintaining operational efficiency.

### Problem Statement

MedCare is experiencing a **18.2% 30-day readmission rate**, significantly exceeding the national benchmark of 15.5%. This gap is costing the organization an estimated **$12.4 million per year** in Centers for Medicare & Medicaid Services (CMS) penalties under the Hospital Readmissions Reduction Program (HRRP).

**What does this mean in practice?** For every 1,000 discharged patients, roughly 182 return to the hospital within 30 days — many of these readmissions are preventable with the right post-discharge support. Beyond the financial penalties, readmissions signal gaps in care transitions: patients may be discharged without adequate medication reconciliation, follow-up appointments, or understanding of their self-care instructions. If MedCare can identify high-risk patients before they leave the hospital, care teams can intervene with targeted discharge planning, follow-up calls, home health referrals, and medication management — reducing both costs and patient suffering.

### Business Objectives

1. **Primary Goal:** Reduce the 30-day readmission rate from 18.2% to 14.5% within 18 months
2. **Secondary Goals:**
    - Identify the top clinical and operational drivers of readmission
    - Create an at-discharge risk scoring system for care teams
    - Enable proactive care coordination for high-risk patients
    - Reduce CMS penalties from $12.4M to $7.5M annually

### Success Criteria

- Readmission prediction model with **AUC-ROC >= 0.75**
- High-risk patient identification with **sensitivity >= 0.80** (catch at least 80% of patients who will be readmitted)
- Clinically interpretable model outputs that care teams can trust and act on
- All analytics compliant with HIPAA regulations

---

## Regulatory Context

Healthcare analytics operate under strict regulatory and ethical oversight. Key regulations that apply to this use case include:

| Regulation                                                   | What It Requires                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **HIPAA** (Health Insurance Portability and Accountability Act) | Protects patient health information; requires safeguards for PHI access, use, and disclosure |
| **HITECH Act**                                               | Strengthens HIPAA enforcement; requires breach notification for unsecured PHI |
| **CMS HRRP** (Hospital Readmissions Reduction Program)       | Penalizes hospitals with excess readmissions for specific conditions; directly motivates this use case |
| **Joint Commission Standards**                               | Requires evidence-based care transition processes and quality measurement |

These regulations mean that beyond building an accurate model, the analytics workflow must protect patient privacy, produce clinically interpretable outputs, and support the quality improvement documentation required by CMS and accreditation bodies.

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

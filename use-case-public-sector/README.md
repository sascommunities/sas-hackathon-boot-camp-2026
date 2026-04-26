# Citizen Service Request Prioritization

## Public Sector Use Case — Data and AI Life Cycle

This use case walks you through the complete **Data and AI Life Cycle** using a realistic citizen service request prioritization scenario, powered by SAS Viya technology.

## Business Understanding

### Organization Background

**Metro City** is a mid-sized municipality that processes approximately 15,000 citizen service requests per month across departments including Public Works, Parks & Recreation, Transportation, Building & Safety, and others. Requests range from pothole repairs and streetlight outages to permit inquiries and noise complaints. The city operates a 311 service center, an online portal, and a mobile app for request submission.

### Problem Statement

Metro City is experiencing a **40% variance in average response times across its 12 districts**, with the overall average sitting at **48.2 hours** — well above the 36-hour target. Citizen satisfaction has declined to **3.2 out of 5.0**, and there are growing concerns about **service equity**: are some neighborhoods consistently receiving slower responses than others? Are urgent requests being identified and escalated quickly enough?

**What does this mean in practice?** A water main break in one district might wait 60 hours while a cosmetic sidewalk complaint in another district gets resolved in 20 hours. Without a systematic way to assess request urgency and allocate resources, departments rely on first-come-first-served processing — which fails to account for the severity, safety implications, or equity dimensions of each request. If Metro City can predict which requests are truly urgent, it can triage them faster, route them to the right department with the right priority, and ensure all districts receive equitable service.

### Business Objectives

1. **Primary Goal:** Reduce average response time from 48.2 to 36 hours within 6 months
2. **Secondary Goals:**
    - Improve citizen satisfaction from 3.2 to 3.7 out of 5.0
    - Achieve less than 10% response time variance between districts
    - Ensure urgent requests (Critical and High priority) are identified with at least 90% recall
    - Comply with Public Records Act, ADA requirements, and algorithmic bias prevention policies

### Success Criteria

- Urgency prediction model with **accuracy greater than 85%**
- **Recall greater than 90%** for urgent requests (do not miss Critical or High priority requests)
- Equitable model performance across all 12 districts
- Actionable triage system that routes requests in real time

---

## Regulatory Context

Public sector analytics operate under transparency, equity, and accountability requirements. Key regulations and policies that apply to this use case include:

| Regulation / Policy                       | What It Requires                                             |
| ----------------------------------------- | ------------------------------------------------------------ |
| **Public Records Act / FOIA**             | Government data and algorithmic decision-making processes may be subject to public disclosure requests |
| **ADA** (Americans with Disabilities Act) | Service delivery must be accessible; model-driven triage must not disadvantage residents with disabilities |
| **Algorithmic Accountability Policies**   | Many municipalities require impact assessments for automated decision systems that affect public services |
| **Title VI of the Civil Rights Act**      | Prohibits discrimination in programs receiving federal funding; applies to service delivery equity |

These requirements mean that the model must not only be accurate — it must be **transparent, auditable, and demonstrably equitable** across all districts and demographics.

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

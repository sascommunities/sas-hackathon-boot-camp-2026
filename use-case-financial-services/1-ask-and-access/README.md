# Step 1: Ask & Access

Welcome to **PremierBank**, a fictional regional bank with $2.1 billion in assets and more than 50,000 customers. In this step you will gain insights into the current business challenges, learn about the value of synthetic data in financial services, and pull your data into **SAS Data Maker** to generate a synthetic version of the dataset.

---

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

| Regulation | What It Requires |
|-----------|-----------------|
| **ECOA** (Equal Credit Opportunity Act) | Prohibits discrimination in credit decisions based on race, color, religion, national origin, sex, marital status, age, or public assistance status |
| **Fair Housing Act** | Prohibits discrimination in housing-related lending |
| **FCRA** (Fair Credit Reporting Act) | Requires adverse action notices when credit is denied or terms are modified based on credit information |
| **SR 11-7** (OCC/Fed Model Risk Management) | Requires validation, documentation, and ongoing monitoring of models used in banking decisions |

These regulations mean that unlike many other analytics use cases, the model you build here must not only be accurate — it must be **explainable, auditable, and demonstrably fair**.

---

## Data Assessment

### Available Data Sources

| Dataset | Description | Records | Key Fields |
|---------|-------------|---------|------------|
| `loan_applications.csv` | Loan application details | 500 | loan_id, application_date, loan_amount, loan_term_months, interest_rate, loan_purpose, property_type, loan_to_value, debt_to_income, defaulted |
| `credit_history.csv` | Credit bureau data | 500 | loan_id, credit_score, credit_accounts, credit_utilization, bankruptcies, delinquencies_2yr, inquiries_6mo, oldest_account_years, total_credit_limit |
| `employment.csv` | Employment and income data | 500 | loan_id, employment_status, employer_type, years_employed, annual_income, monthly_debt, employment_verified, income_verified |
| `payment_history.csv` | Loan payment records | 500 | payment_id, loan_id, payment_date, amount_due, amount_paid, days_late, payment_status |

### Data Quality Notes

- All datasets can be joined via `loan_id`
- Default label available in loan_applications.csv (`defaulted` column: 1 = defaulted, 0 = current)
- Payment history provides granular payment-level behavior that must be aggregated to the loan level

---

## Stakeholder Alignment

Before building models, confirm alignment with key stakeholders:

| Stakeholder | What They Need |
|------------|---------------|
| **Chief Risk Officer** | Reduced loss rates, regulatory compliance evidence, model validation documentation |
| **Credit Committee** | Clear risk tier definitions, adverse action reason codes, override guidelines |
| **Fair Lending Officer** | Fairness assessment across protected classes and proxy variables, disparate impact analysis |
| **Loan Officers** | Actionable approve/review/decline recommendations with transparent rationale |
| **Internal Audit** | Model governance trail per SR 11-7, data lineage, version control |

---

## Initial Hypotheses

Based on domain knowledge and regulatory guidance, we hypothesize:

| # | Hypothesis | Metrics to Test |
|---|-----------|-----------------|
| H1 | **Credit Score Drives Default** — Borrowers with lower FICO scores are substantially more likely to default | Default rate by credit score band, average credit score for defaulted vs. current loans |
| H2 | **Debt-to-Income Is a Key Risk Factor** — Higher DTI ratios indicate stretched finances and increased default risk | Default rate by DTI band, average DTI for defaulted vs. current |
| H3 | **Payment Behavior Predicts Future Default** — Borrowers who have been late on prior payments are more likely to eventually default | Late payment rate, severe delinquency count, average days late |
| H4 | **Employment Stability Matters** — Borrowers with shorter tenure, unverified employment, or lower income are higher risk | Default rate by years employed band, employment verification status, income band |
| H5 | **Loan Characteristics Affect Risk** — Larger loans, higher LTV ratios, and longer terms carry more default risk | Default rate by loan amount band, LTV band, loan term |

---

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

---

## The Value of Synthetic Data

Synthetic data is artificially generated data that mirrors the statistical properties, patterns, and structure of real-world data — without containing any actual records from the original dataset. It is produced using generative models that learn the distributions, correlations, and relationships present in real data and then create entirely new records that are statistically representative but not traceable back to any individual. In financial services, where data sensitivity and regulatory scrutiny are at their highest, synthetic data has become an essential tool for responsible analytics development.

For a use case like loan default prediction at PremierBank, synthetic data offers critical advantages that go beyond general privacy protection. First, lending data is among the most sensitive data a bank holds — it includes income, employment, credit scores, and debt levels, all of which are personally identifiable and regulated under GLBA (Gramm-Leach-Bliley Act) and FCRA. Synthetic data allows analytics teams, model validators, and external partners to work with realistic data without exposing actual borrower records or triggering data-sharing compliance obligations. Second, synthetic data is uniquely valuable for **fair lending testing**: teams can generate synthetic populations with controlled demographic distributions to stress-test whether a model produces disparate impact across protected classes, even when the original dataset may not contain sufficient representation of minority groups. Third, synthetic data enables **stress testing and scenario analysis** — what happens to default predictions if unemployment doubles, if interest rates spike by 200 basis points, or if a new loan product is introduced? These forward-looking scenarios can be modeled synthetically before they occur in reality, giving the credit committee early insight into portfolio resilience.

---

## Working with SAS Data Maker

[SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) is the SAS platform for generating high-quality synthetic data. In this section you will pull the PremierBank datasets into SAS Data Maker and create a synthetic version that preserves the statistical relationships across all four tables.

### Accessing the Data

The four source datasets are located in:

```
Bootcamp/use-case-financial-services/csv
├── loan_applications.csv
├── credit_history.csv
├── employment.csv
└── payment_history.csv
```

### Workflow: Generating Synthetic Data with SAS Data Maker

Follow these steps to create your synthetic dataset:

#### 1. Create a Project

1. Open **SAS Data Maker**
2. Click **New project** to start a new project
3. Give it a descriptive name, e.g., *PremierBank Loan Default — Synthetic Generation*

#### 2. Import Source Data

1. Within your data plan, click **Add Data Source**
2. Navigate to the `Bootcamp/use-case-financial-services/csv` folder
3. This will import all four CSV files:
   - `loan_applications.csv` — this is your primary entity table
   - `credit_history.csv` — related table linked by `loan_id`
   - `employment.csv` — related table linked by `loan_id`
   - `payment_history.csv` — child table linked by `loan_id` (multiple payments per loan)
4. SAS Data Maker will profile each table and display column types, distributions, and summary statistics

#### 3. Define Relationships

The job that ran to understand the tables also will try to resolve the relationships between the tables. Please review that the relationship is mapped correctly. Your goal is to map a relationship that looks like the below

![fin-services-ideal-map](./img/fin-services-ideal-map.png)

1. For `loan_applications`, set `loan_id` as the Primary key
2. For `payment_history` set `payment_id` as the Primary key and `loan_id` as the Foreign key mapping to `loan_applications`
3. For `credit_history` and `employment`, set `loan_id` as the Foreign key mapping to `loan_applications`
4. All tables are of the type Tabular
5. Review the key relationships between the tables:
   - `credit_history.loan_id` -> `loan_applications.loan_id`
   - `employment.loan_id` -> `loan_applications.loan_id`
   - `payment_history.loan_id` -> `loan_applications.loan_id`
6. These relationships ensure that the synthetic data maintains referential integrity — every synthetic payment record will belong to a valid synthetic loan

#### 4. Training Settings

1.   **Random state**, is optional and can be set to a seed variable. Why not try a classic like 42?
2.   **Model type**, we can choose between `PrivBayes` and `SMOTE` , we will be using PrivBayes here to make use of the differential privacy feature
3.   **Use differential privacy**, this will help us to reduce the privacy impact on each individual in the data. Increasing the privacy is a great way to enhance your Trustworthy AI as it increases the trust in you as a data processor
4.   The rest of the values we can leave at the default values and click the Start training button. If you want to feel free to explore the options though in more detail there is inline hints, or reach out to one of the SAS Mentors on site.
5.   The training process will take a moment and once it is done and all the metrics are calculated we can move to the next step which is `Evaluation`

#### 5. Evaluation

On the Evaluation tab we get a lot of insights into our Synthetic Data Generation Model, how it compares to the input sources and not just on a per table basis but also across the different tables.

Please take your time and explore these results to gain an understanding of how closely the synthetic data will match the original data sources. Feel free to discuss these results in the group and also reach out to SAS Mentors on site if you have question or consult the [SAS Documentation](https://go.documentation.sas.com/doc/en/sdgcdc/v_001/sdgug/p0ki9glx7acxpyn1wttognicd7qi.htm).

#### 6. Generation

1. **Output destination**, select the path here and let it generate a unique path on its own
2. Leave all other options at default and click the Generate button
3. Now a generation job is triggered that will create the synthetic data for each table for us and make that 
4. Once the generation has finished we get a summary of everything, a note on where the data is stored and a sample of the synthetic data. The generated data is stored onto a blob storage, don't worry you will not have to download anything onto your laptop as we will provide the data already available in SAS Viya and SAS Viya Workbench so that you can get to work.

---

## Next Steps

Proceed to **[Step 2: Prepare](../2-prepare/)** to load, profile, and join the data into an analytical base table using SAS Viya Workbench.

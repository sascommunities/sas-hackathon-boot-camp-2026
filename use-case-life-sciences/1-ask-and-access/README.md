# Step 1: Ask & Access

Welcome to **MedCare Health System**, a fictional regional healthcare network. In this step you will gain insights into the current business challenges, learn about the value of synthetic data in healthcare, and pull your data into **SAS Data Maker** to generate a synthetic version of the dataset.

---

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

| Regulation | What It Requires |
|-----------|-----------------|
| **HIPAA** (Health Insurance Portability and Accountability Act) | Protects patient health information; requires safeguards for PHI access, use, and disclosure |
| **HITECH Act** | Strengthens HIPAA enforcement; requires breach notification for unsecured PHI |
| **CMS HRRP** (Hospital Readmissions Reduction Program) | Penalizes hospitals with excess readmissions for specific conditions; directly motivates this use case |
| **Joint Commission Standards** | Requires evidence-based care transition processes and quality measurement |

These regulations mean that beyond building an accurate model, the analytics workflow must protect patient privacy, produce clinically interpretable outputs, and support the quality improvement documentation required by CMS and accreditation bodies.

---

## Data Assessment

### Available Data Sources

| Dataset | Description | Records | Key Fields |
|---------|-------------|---------|------------|
| `patients.csv` | Patient demographics and readmission status | 500 | patient_id, admission_date, age, gender, insurance_type, primary_diagnosis_category, comorbidity_count, readmitted_30days |
| `admissions.csv` | Admission and discharge details | 500 | admission_id, patient_id, admission_date, discharge_date, length_of_stay, admission_type, discharge_disposition |
| `clinical_measures.csv` | Vital signs and lab results | 500 | patient_id, bmi, blood_pressure_systolic, blood_pressure_diastolic, glucose_level, lab_results_flag |
| `medications.csv` | Medication orders during admission | 326 | medication_id, patient_id, medication_name, medication_class, dosage, frequency, high_risk_flag, start_date, end_date |

### Data Quality Notes

- Data covers patient admissions in 2025
- Readmission label available in patients.csv (`readmitted_30days` column: 1 = readmitted, 0 = not readmitted)
- All datasets can be joined via `patient_id`
- Not every patient has medication records (326 medication rows across 500 patients)
- Clinical measures are one record per patient at time of admission

---

## Stakeholder Alignment

Before building models, confirm alignment with key stakeholders:

| Stakeholder | What They Need |
|------------|---------------|
| **Chief Medical Officer** | Clinical validity of the model, patient safety assurance, clinician trust |
| **Director of Quality Improvement** | Measurable readmission rate reduction, CMS HRRP compliance evidence |
| **VP of Data Analytics** | Model performance targets, scalability to production, HIPAA-compliant workflow |
| **Director of Care Transitions** | Actionable risk scores at discharge, clear intervention recommendations |
| **Chief Information Officer** | EHR integration plan, data governance compliance, HIPAA audit trail |

---

## Initial Hypotheses

Based on clinical domain knowledge and preliminary exploration, we hypothesize:

| # | Hypothesis | Metrics to Test |
|---|-----------|-----------------|
| H1 | **Comorbidity Burden Drives Readmission** — Patients with higher comorbidity counts are more likely to be readmitted due to complex care needs | Readmission rate by comorbidity_count, comorbidity distribution |
| H2 | **Length of Stay Signals Severity** — Patients with very short stays (premature discharge) or very long stays (severe illness) are at higher risk | Readmission rate by length_of_stay categories, average LOS by readmission status |
| H3 | **Emergency Admissions Carry Higher Risk** — Emergency admissions, reflecting acute or unplanned events, are associated with higher readmission rates than elective admissions | Readmission rate by admission_type |
| H4 | **Medication Complexity Increases Risk** — Patients on more medications, especially high-risk medications, face higher readmission rates due to adherence challenges and drug interactions | Medication count per patient, high-risk medication count, readmission rate by polypharmacy status |
| H5 | **Abnormal Clinical Measures Predict Readmission** — Patients with abnormal lab results, elevated blood pressure, high glucose, or extreme BMI are at higher readmission risk | Readmission rate by lab_results_flag, BP classification, glucose level, BMI category |

---

## Scope

### In Scope

- Patient admissions during the 2025 observation period
- All four data sources (patients, admissions, clinical measures, medications)
- Binary classification: readmitted within 30 days (1) vs. not readmitted (0)
- Fairness assessment on insurance type
- HIPAA-compliant analytics workflow

### Out of Scope

- Clinical trial data and research protocols
- Real-time patient monitoring and alerting
- Outpatient-only encounters (no inpatient admission)
- Specific physician performance evaluation

---

## The Value of Synthetic Data

Synthetic data is artificially generated data that mirrors the statistical properties, patterns, and structure of real-world data — without containing any actual records from the original dataset. It is produced using generative models that learn the distributions, correlations, and relationships present in real data and then create entirely new records that are statistically representative but not traceable back to any individual. In recent years, synthetic data has become a critical tool across industries as organizations face growing pressure around data privacy, regulatory compliance, and the practical challenge of getting enough high-quality data for analytics and machine learning.

For a use case like patient readmission prediction at MedCare Health System, synthetic data offers several concrete benefits that are especially important in healthcare. First and foremost, it enables teams to develop, test, and iterate on models without exposing Protected Health Information (PHI) — a fundamental requirement under HIPAA. Real patient records containing diagnoses, medications, vital signs, and insurance information are among the most sensitive data categories that exist; synthetic generation eliminates re-identification risk entirely, allowing analysts, data scientists, and external collaborators to work freely without Business Associate Agreements or de-identification pipelines. Second, synthetic data can augment underrepresented clinical scenarios: if the dataset contains very few readmissions among patients with rare diagnosis categories or unusual comorbidity combinations, synthetic generation can produce additional realistic examples to improve model training for these edge cases. Third, it enables multi-site collaboration — hospitals across the MedCare network, external research partners, and technology vendors can all work with realistic clinical data without the legal and ethical overhead of sharing actual patient records. Finally, synthetic data supports clinical simulation: what if readmission rates doubled for cardiac patients? What if a new high-risk medication class entered formulary? These scenarios can be modeled synthetically before they materialize, enabling proactive care pathway design.

---

## Working with SAS Data Maker

[SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) is the SAS platform for generating high-quality synthetic data. In this section you will pull the MedCare datasets into SAS Data Maker and create a synthetic version that preserves the statistical relationships across all four tables.

### Accessing the Data

The four source datasets are located in:

```
SAS-Hackathon-Bootcamp-2026/use-case-life-sciences/data
├── patients.csv
├── admissions.csv
├── clinical_measures.csv
└── medications.csv
```

### Workflow: Generating Synthetic Data with SAS Data Maker

Follow these steps to create your synthetic dataset:

#### 1. Create a Project

1. Open **SAS Data Maker**
2. Click **New project** to start a new project
3. Give it a descriptive name, e.g., *MedCare Readmission — Synthetic Generation*

#### 2. Import Source Data

1. Within your data plan, click **Add Data Source**
2. Navigate to the `SAS-Hackathon-Bootcamp-2026/use-case-life-sciences/data` folder
3. This will import all four CSV files:
   - `patients.csv` — this is your primary entity table
   - `admissions.csv` — related table linked by `patient_id`
   - `clinical_measures.csv` — related table linked by `patient_id`
   - `medications.csv` — child table linked by `patient_id` (multiple medications per patient)
4. SAS Data Maker will profile each table and display column types, distributions, and summary statistics

#### 3. Define Relationships

The job that ran to understand the tables also will try to resolve the relationships between the tables. Please review that the relationship is mapped correctly

1. For `patients` set `patient_id` as the Primary key
2. For `admissions` set `admission_id` as the Primary key and `patient_id` as the Foreign key
3. For `clinical_measures` set `patient_id` as the Primary key
4. For `medications` set `medication_id` as the Primary key and `patient_id` as the Foreign key
5. All tables are of the type Tabular
6. Review the key relationships between the tables:
   - `admissions.patient_id` -> `patients.patient_id`
   - `clinical_measures.patient_id` -> `patients.patient_id`
   - `medications.patient_id` -> `patients.patient_id`
7. These relationships ensure that the synthetic data maintains referential integrity — every synthetic admission will belong to a valid synthetic patient

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

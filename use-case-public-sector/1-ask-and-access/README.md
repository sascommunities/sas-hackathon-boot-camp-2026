# Step 1: Ask & Access

Welcome to **Metro City**, a fictional municipality serving 850,000 residents across 12 districts. In this step you will gain insights into the current service delivery challenges, learn about the value of synthetic data in the public sector, and pull your data into **SAS Data Maker** to generate a synthetic version of the dataset.

---

## Data Assessment

### Available Data Sources

| Dataset | Description | Records | Key Fields |
|---------|-------------|---------|------------|
| `service_requests.csv` | Individual service request records | 500 | request_id, citizen_id, submission_date, request_type, department, priority_level, location_district, response_time_hours, citizen_satisfaction, resolved |
| `citizens.csv` | Citizen profiles and service history | 300 | citizen_id, registration_date, age_group, district, contact_preference, previous_requests, satisfaction_history |
| `department_performance.csv` | Monthly department-level metrics | 96 | department, month, year, avg_response_time_hours, resolution_rate, citizen_satisfaction_avg, staff_count, budget_utilization, requests_received, requests_completed, overtime_hours |
| `request_history.csv` | Historical request volumes by district and department | ~3,456 | year, month, district, department, request_count, avg_response_time, resolution_rate, priority_critical_count, priority_high_count, priority_medium_count, priority_low_count |

### Data Quality Notes

- Service requests data covers 2024 (12 months)
- Target variable is derived: `is_urgent` = 1 if `priority_level` is Critical or High, 0 otherwise
- `citizen_id` links `service_requests` to `citizens`
- `department` links `service_requests` to `department_performance`
- `request_history` provides aggregate trends by district and department over multiple years

---

## Stakeholder Alignment

Before building models, confirm alignment with key stakeholders:

| Stakeholder | What They Need |
|------------|---------------|
| **City Manager** | Measurable improvement in response times and citizen satisfaction scores |
| **District Council Members** | Evidence of equitable service delivery across all 12 districts |
| **Department Heads** | Actionable triage recommendations, workload balancing insights |
| **IT Director** | Integration plan with 311 system, data governance compliance |
| **Equity & Inclusion Officer** | Fairness assessment across districts, bias mitigation evidence |

---

## Initial Hypotheses

Based on domain knowledge and preliminary exploration, we hypothesize:

| # | Hypothesis | Metrics to Test |
|---|-----------|-----------------|
| H1 | **Request Type Drives Urgency** — Certain request types (e.g., water main breaks, safety hazards) are inherently more urgent than others (e.g., cosmetic repairs, permit inquiries) | Urgency rate by request type, response time by request type |
| H2 | **Department Capacity Affects Response** — Departments with higher workloads, lower staff counts, or higher budget utilization have slower response times and lower resolution rates | Average response time vs. staff count, requests per staff member, budget utilization vs. resolution rate |
| H3 | **District Equity Matters** — Some districts consistently receive slower service, potentially correlated with request volume, district demographics, or resource allocation | Response time by district, satisfaction by district, resolution rate by district |
| H4 | **Seasonal Patterns Exist** — Request volumes and types vary by season (e.g., potholes spike after winter, park complaints rise in summer), affecting department workload and response times | Monthly request counts, seasonal response time trends, priority distribution by month |
| H5 | **Citizen History Predicts Engagement** — Citizens with more previous requests and higher historical satisfaction are more engaged and may submit higher-quality, more actionable requests | Previous requests vs. response time, satisfaction history vs. current satisfaction |

---

## Scope

### In Scope

- Service requests submitted during the 2024 observation period
- All four data sources (service requests, citizens, department performance, request history)
- Binary classification: urgent (1) vs. non-urgent (0), derived from priority_level
- Fairness assessment across districts (location_district)

### Out of Scope

- Budget allocation and financial planning
- Personnel management and staffing optimization
- External factors (weather events, population changes)
- Cross-jurisdiction service agreements

---

## The Value of Synthetic Data

Synthetic data is artificially generated data that mirrors the statistical properties, patterns, and structure of real-world data — without containing any actual records from the original dataset. It is produced using generative models that learn the distributions, correlations, and relationships present in real data and then create entirely new records that are statistically representative but not traceable back to any individual. In the public sector, where data is often subject to stringent regulatory requirements, synthetic data has become an essential tool for responsible innovation.

For a use case like citizen service request prioritization at Metro City, synthetic data offers several concrete benefits. First, it enables teams to develop, test, and iterate on models without exposing real citizen information — a crucial consideration when dealing with personally identifiable information subject to the Public Records Act and privacy regulations. Citizen addresses, complaint details, and interaction histories are sensitive; synthetic generation allows analysts, contractors, and cross-agency partners to work with realistic data without the overhead of data access agreements or redaction pipelines. Second, synthetic data supports equity testing: by generating scenarios with varying district-level distributions, teams can stress-test whether models perform fairly across all neighborhoods before deploying them on real populations. Third, it enables simulation of scenarios that may not yet exist in the data — such as a sudden spike in emergency requests during a natural disaster, or the impact of adding staff to an understaffed department. These "what-if" analyses can inform resource allocation and policy decisions before they are needed. Finally, synthetic data makes hackathons and training exercises like this one possible: participants can explore realistic municipal data patterns without any risk to actual citizens.

---

## Working with SAS Data Maker

[SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) is the SAS platform for generating high-quality synthetic data. In this section you will pull the Metro City datasets into SAS Data Maker and create a synthetic version that preserves the statistical relationships across all four tables.

### Accessing the Data

The four source datasets are located in:

```
SAS-Hackathon-Bootcamp-2026/use-case-public-sector/data
├── service_requests.csv
├── citizens.csv
├── department_performance.csv
└── request_history.csv
```

### Workflow: Generating Synthetic Data with SAS Data Maker

Follow these steps to create your synthetic dataset:

#### 1. Create a Project

1. Open **SAS Data Maker**
2. Click **New project** to start a new project
3. Give it a descriptive name, e.g., *Metro City Service Requests — Synthetic Generation*

#### 2. Import Source Data

1. Within your data plan, click **Add Data Source**
2. Navigate to the `SAS-Hackathon-Bootcamp-2026/use-case-public-sector/data` folder
3. This will import all four CSV files:
   - `service_requests.csv` — this is your primary request-level table
   - `citizens.csv` — related table linked by `citizen_id`
   - `department_performance.csv` — aggregate reference table linked by `department`
   - `request_history.csv` — aggregate reference table for historical trends
4. SAS Data Maker will profile each table and display column types, distributions, and summary statistics
5. Review the **Columns** tab for each table. For `service_requests` you will notice that the `request_type` column has no semantic type set (shown as *(Not set)* with a red indicator). Click that cell and set its semantic type to **Category** — otherwise training will fail with the error *"The semantic type must be set, or the column should be dropped."*

#### 3. Define Relationships

The job that ran to understand the tables also will try to resolve the relationships between the tables. Please review that the relationship is mapped correctly

1. For `citizens` set `citizen_id` as the Primary key
2. For `service_requests` set `request_id` as the Primary key and `citizen_id` as the Foreign key
3. `department_performance` and `request_history` are aggregate reference tables — review that their `department` and `district` values are consistent with those in `service_requests` and `citizens`
4. All tables are of the type Tabular
5. Review the key relationships between the tables:
   - `service_requests.citizen_id` -> `citizens.citizen_id`
6. These relationships ensure that the synthetic data maintains referential integrity — every synthetic service request will belong to a valid synthetic citizen and reference a valid department

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

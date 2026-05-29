# Step 1: Ask & Access

Welcome to **ShopEase**, a fictional online retail platform. In this step you will gain insights into the current business challenges, learn about the value of synthetic data, and pull your data into **SAS Data Maker** to generate a synthetic version of the dataset.

---

## Business Understanding

### Company Background

**ShopEase** is an e-commerce platform serving 1,000+ active customers across the United States. The company offers products in Electronics, Clothing, Home & Garden, and Books categories through a web and mobile platform.

### Problem Statement

The company is experiencing a **12% monthly customer churn rate**, significantly impacting revenue and growth. Leadership wants to understand why customers are leaving and identify at-risk customers before they churn.

**What does this mean in practice?** Every month, roughly 120 out of every 1,000 customers stop purchasing. Acquiring a new customer costs 5-7x more than retaining an existing one, making this churn rate a direct threat to profitability. If ShopEase can predict which customers are about to leave, it can intervene with targeted retention offers, personalized outreach, or service improvements — turning potential losses into saved relationships.

### Business Objectives

1. **Primary Goal:** Reduce monthly churn rate from 12% to 8% within 6 months
2. **Secondary Goals:**
   - Identify top 5 factors driving customer churn
   - Create an early warning system for at-risk customers
   - Enable proactive retention campaigns

### Success Criteria

- Churn prediction model with **≥80% accuracy**
- Actionable insights for the retention team
- ROI-positive retention campaign within 3 months

---

## Data Assessment

### Available Data Sources

| Dataset | Description | Records | Key Fields |
|---------|-------------|---------|------------|
| `customers.csv` | Customer demographics and churn status | 1,000 | customer_id, signup_date, age, gender, location, subscription_tier, email_opt_in, churned |
| `transactions.csv` | Purchase history | ~5,000 | transaction_id, customer_id, transaction_date, amount, product_category, payment_method, discount_applied |
| `sessions.csv` | Website/app activity | ~10,000 | session_id, customer_id, session_date, duration_minutes, pages_viewed, device_type, referral_source, cart_abandonment |
| `support_tickets.csv` | Customer service interactions | ~400 | ticket_id, customer_id, ticket_date, issue_category, priority, resolution_time_hours, satisfaction_score, resolved |

### Data Quality Notes

- Data covers January 2023 - December 2023 (12 months)
- Churn label available in customers.csv (`churned` column: 1 = churned, 0 = active)
- All datasets can be joined via `customer_id`

---

## Stakeholder Alignment

Before building models, confirm alignment with key stakeholders:

| Stakeholder | What They Need |
|------------|---------------|
| **VP of Marketing** | Targeting criteria for retention campaigns, ROI projections for campaign spend |
| **Head of Customer Experience** | Root cause insights into churn drivers, service improvement recommendations |
| **Product Manager** | Feature-level engagement data to prioritize product roadmap decisions |
| **Data Science Lead** | Model performance targets, fairness requirements, deployment timeline |
| **CFO** | Revenue impact analysis, customer lifetime value preservation estimates |

---

## Initial Hypotheses

Based on domain knowledge and preliminary exploration, we hypothesize:

| # | Hypothesis | Metrics to Test |
|---|-----------|-----------------|
| H1 | **Engagement Drives Retention** — Customers with low engagement (fewer sessions, shorter durations) are more likely to churn | Average session duration, sessions per month, pages viewed per session |
| H2 | **Purchase Frequency Matters** — Customers who purchase less frequently or have decreasing transaction amounts are at higher churn risk | Number of transactions, average order value, days since last purchase |
| H3 | **Support Issues Indicate Risk** — Customers with multiple support tickets, especially unresolved or low-satisfaction ones, are more likely to churn | Number of support tickets, ticket priority distribution, average satisfaction score |
| H4 | **Subscription Tier Impacts Churn** — Basic tier customers churn at higher rates than Premium customers due to lower investment/commitment | Churn rate by subscription tier |
| H5 | **Email Opt-Out Signals Disengagement** — Customers who have opted out of marketing emails are less engaged and more likely to churn | Churn rate by email_opt_in status |

---

## Scope

### In Scope

- Customers active during the January–December 2023 observation period
- All four data sources (customers, transactions, sessions, support tickets)
- Binary classification: churned (1) vs. active (0)
- Fairness assessment on subscription tier

### Out of Scope

- Real-time fraud detection (separate initiative)
- Product recommendation engine
- External market or competitor data
- Pricing strategy optimization

---

## The Value of Synthetic Data

Synthetic data is artificially generated data that mirrors the statistical properties, patterns, and structure of real-world data — without containing any actual records from the original dataset. It is produced using generative models that learn the distributions, correlations, and relationships present in real data and then create entirely new records that are statistically representative but not traceable back to any individual. In recent years, synthetic data has become a critical tool across industries as organizations face growing pressure around data privacy, regulatory compliance, and the practical challenge of getting enough high-quality data for analytics and machine learning.

For a use case like customer churn prediction at ShopEase, synthetic data offers several concrete benefits. First, it enables teams to develop, test, and iterate on models in non-production environments without exposing real customer information — a crucial consideration when dealing with personally identifiable information (PII) like customer demographics and purchase behavior. Second, synthetic data can be used to augment underrepresented segments: if the dataset has very few examples of Premium-tier customers who churned, synthetic generation can produce additional realistic examples to improve model training. Third, it allows cross-team collaboration — marketing, engineering, and external partners can all work with realistic data without the overhead of data access agreements or anonymization pipelines. Finally, synthetic data supports stress-testing and simulation: what if churn doubled? What if a new product category was introduced? These scenarios can be modeled synthetically before they happen in reality.

---

## Working with SAS Data Maker

[SAS Data Maker](https://www.sas.com/en_us/software/data-maker.html) is the SAS platform for generating high-quality synthetic data. In this section you will pull the ShopEase datasets into SAS Data Maker and create a synthetic version that preserves the statistical relationships across all four tables.

### Accessing the Data

The four source datasets are located in:

```
SAS-Hackathon-Bootcamp-2026/use-case-retail/data
├── customers.csv
├── transactions.csv
├── sessions.csv
└── support_tickets.csv
```

### Workflow: Generating Synthetic Data with SAS Data Maker

Follow these steps to create your synthetic dataset:

#### 1. Create a Project

1. Open **SAS Data Maker**
2. Click **New project** to start a new project
3. Give it a descriptive name, e.g., *ShopEase Churn — Synthetic Generation*

#### 2. Import Source Data

1. Within your data plan, click **Add Data Source**
2. Navigate to the `SAS-Hackathon-Bootcamp-2026/use-case-retail/data` folder
3. This will import all four CSV files:
   - `customers.csv` — this is your primary entity table
   - `transactions.csv` — child table linked by `customer_id` (multiple transactions per customer)
   - `sessions.csv` — child table linked by `customer_id` (multiple sessions per customer)
   - `support_tickets.csv` — child table linked by `customer_id`
4. SAS Data Maker will profile each table and display column types, distributions, and summary statistics

#### 3. Define Relationships

The job that ran to understand the tables also will try to resolve the relationships between the tables. Please review that the relationship is mapped correctly

1. For `customers` set `customer_id` as the Primary key
2. For `transactions` set `transaction_id` as the Primary key and `customer_id` as the Foreign key
3. For `sessions` set `session_id` as the Primary key and `customer_id` as the Foreign key
4. For `support_tickets` set `ticket_id` as the Primary key and `customer_id` as the Foreign key
5. All tables are of the type Tabular
6. Review the key relationships between the tables:
   - `transactions.customer_id` -> `customers.customer_id`
   - `sessions.customer_id` -> `customers.customer_id`
   - `support_tickets.customer_id` -> `customers.customer_id`
7. These relationships ensure that the synthetic data maintains referential integrity — every synthetic transaction will belong to a valid synthetic customer

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

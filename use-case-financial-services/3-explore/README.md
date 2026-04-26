# Step 3: Explore

In this step you will use **SAS Visual Analytics** and its built-in **Copilot** to visually explore the Analytical Base Table (ABT) you created in Step 2. The goal is to understand the patterns behind loan default before building a predictive model.

---

## Prerequisites

The analytical base table should already be loaded into the **Public** CAS library. If you completed Step 2 the data was saved as `financial_services_abt.csv`. Your bootcamp environment has this pre-loaded as a CAS table named **`FINANCIAL_SERVICES_ABT`** in the **Public** caslib.

---

## Accessing the Data in SAS Visual Analytics

1. Open **SAS Visual Analytics** from the SAS Viya home page
2. Click **New Report**
3. In the data panel click on the Add Data button and from the available tables please select **FINANCIAL_SERVICES_ABT**
4. Add it as your data source — you should see all the features from Step 2 listed in the data items pane on the left

> **Tip:** If the table does not appear in the Public caslib, ask your SAS Mentor to help promote it. You can also load it directly by uploading the CSV through the **Manage Data** interface.

---

## Using the SAS Visual Analytics Copilot

SAS Visual Analytics includes a **Copilot** — an AI assistant that helps you explore data faster. You can find the Copilot icon in the top right hand corner. The Copilot can:

- **Suggest visualizations** based on the variables you select
- **Answer questions** about your data in natural language
- **Generate insights** by automatically scanning for interesting patterns
- **Build charts** from plain-language prompts

### How to Use the Copilot

1. Click the **Copilot** icon to open the assistant panel
2. Type a question or request in natural language
3. The Copilot will suggest or create a visualization directly in your report
4. You can refine the result by following up with additional prompts
5. You can right click into the chat panel and get suggestions on prompts to help you.

---

## Guided Exploration: Questions to Ask

Work through the following questions to build your understanding of the default patterns. For each question, try creating the visualization manually **and/or** by asking the Copilot.

Please do not feel obligated to answer all of these questions, rather pick and choose questions that interest you, help you advance your understanding of the data or sound challenging for your current level.

You are also more than welcome to just explore the data on your own making use of SAS Visual Analytics and the SAS Viya Copilot.

### Understanding the Target Variable

**Goal:** Get a baseline understanding of default in the dataset.

- *"Show me the distribution of defaulted loans"*
- *"What percentage of loans have defaulted?"*

Create a **bar chart** or **pie chart** of the `defaulted` variable. You should see roughly 25-26% defaulted and 74-75% current — confirming this is a moderately imbalanced classification problem.

### Hypothesis 1: Credit Score Drives Default

**Goal:** Validate whether lower FICO scores correlate with higher default rates.

- *"Compare the average credit score between defaulted and current loans"*
- *"Show me default rate by FICO score band"*
- *"What is the distribution of credit scores for defaulted vs. current loans?"*

Create **box plots** of `credit_score` grouped by `defaulted`. Create a **bar chart** showing default rate by FICO band (use the `fico_Excellent`, `fico_Good`, `fico_Fair`, `fico_Poor`, `fico_VeryPoor` columns).

> **What to look for:** Loans with Very Poor and Poor FICO scores should have noticeably higher default rates. There may be a threshold effect around the 650 mark.

### Hypothesis 2: Debt-to-Income Impacts Default

**Goal:** Determine whether financial strain predicts default.

- *"Show me the distribution of debt_to_income by default status"*
- *"Compare average debt-to-income ratio between defaulted and current loans"*
- *"Is there a relationship between debt_service_ratio and default?"*

Create a **histogram** of `debt_to_income` colored by `defaulted`. Also create a **scatter plot** of `debt_to_income` vs. `credit_score` with `defaulted` as the color.

> **What to look for:** Defaulted loans likely have higher DTI ratios. A DTI above 43% is a common regulatory threshold that indicates elevated risk.

### Hypothesis 3: Payment Behavior Predicts Future Default

**Goal:** Explore whether past payment patterns signal default risk.

- *"Show me the average late payment rate for defaulted vs. current loans"*
- *"What is the distribution of average days late by default status?"*
- *"Compare `late_payment_count` between defaulted and current loans"*

Create a **box plot** of `avg_days_late` by `defaulted` and a **box plot** of `late_payment_rate` by `defaulted`.

> **What to look for:** Defaulted loans should show a higher mean late payment rate and longer maximum days late than current loans. Note that in this synthetic dataset no loans cross the 60-day severe-delinquency threshold (`severe_delinquency_flag` is uniformly 0), so payment behavior shows up through the rate and days-late columns rather than the flag.

### Hypothesis 4: Employment Stability Matters

**Goal:** Check if employment characteristics affect default risk.

- *"What is the default rate by income band?"*
- *"Compare default rates for verified vs. unverified employment"*
- *"Show me default rate by years employed band"*

Create **stacked bar charts** showing default proportions for each income band and tenure band. Create a **bar chart** comparing default rates by `employment_verified` and `income_verified`.

> **What to look for:** Lower income bands and shorter employment tenure should correlate with higher default. Unverified income may carry additional risk.

### Hypothesis 5: Loan Characteristics Affect Risk

**Goal:** Understand how loan structure relates to default.

- *"Show me default rate by loan-to-value ratio"*
- *"Compare loan amounts between defaulted and current loans"*
- *"What is the relationship between interest rate and default?"*

Create a **scatter plot** of `loan_to_value` vs. `interest_rate` with `defaulted` as the color. Create a **histogram** of `loan_amount` by default status.

> **What to look for:** Higher LTV ratios (less equity) and higher interest rates should correlate with more defaults — interest rate itself may be a proxy for the lender's initial risk assessment. `loan_term_months` does not show a clean monotonic relationship with default in this dataset, so don't expect a strong signal there.

### Correlation and Multi-Variable Exploration

**Goal:** Find feature interactions and the strongest predictors.

- *"Which features are most correlated with default?"*
- *"Show me a correlation matrix of the top 10 numeric features"*
- *"Create a decision tree to show which factors split defaulted from current loans"*

Use the Copilot's **automated analysis** feature to let it scan for the strongest relationships.

> **What to look for:** The Copilot may surface interactions you wouldn't have checked manually, such as "loans with DTI above 45% AND credit score below 620 have a 40%+ default probability."

### Regulatory Considerations for Exploration

**Important:** When exploring lending data, be mindful of fair lending requirements:

- **Do not visualize default rates by protected classes** (race, ethnicity, gender, age, marital status, national origin) — even in exploratory analysis, this creates compliance risk
- **Focus on legitimate credit factors:** credit score, DTI, LTV, payment history, employment stability
- **Document your exploration:** Note which variables show strong separation and which do not — this documentation supports model validation per SR 11-7
- If you discover that a variable like `income_band` shows dramatically different default rates, flag it for the fairness assessment in Step 4, as income can serve as a proxy for protected characteristics

---

## Building Your Report

As you work through the questions above, organize your findings into a report:

1. **Overview Page:** Default distribution, key summary stats
2. **Credit Profile Page:** Credit score bands, utilization, delinquency history
3. **Financial Strain Page:** DTI, debt service ratio, income bands
4. **Payment Behavior Page:** Late payment rates, delinquency flags, shortfall ratios
5. **Loan Characteristics Page:** LTV, loan amount, interest rate, term

Use **filters** and **interactions** between visualizations — clicking a bar in one chart should filter the others. This lets you drill into segments like "loans with DTI above 43% and credit score below 650."

---

## Key Takeaways to Carry Forward

Before moving on to Step 4, summarize what you've learned:

1. **Which hypotheses were confirmed?** (likely all five, with varying strength)
2. **Which features show the strongest separation** between defaulted and current loans?
3. **Are there any surprising patterns** the Copilot surfaced?
4. **What is the class balance?** (important for model training strategy — expect moderate imbalance)
5. **Are there any proxy variable concerns** to flag for fairness testing?

These insights will directly inform the model building approach in the next step.

Finally feel free to save the report, the default location is My Folder, which is ideal here as to not clutter up the workspace for everybody else. You can also give it a name so that it is easier to remember what this report is about.

---

## Next Steps

Proceed to **[Step 4: Model](../4-model/)** to build and compare predictive models in SAS Model Studio.

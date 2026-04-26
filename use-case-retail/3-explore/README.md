# Step 3: Explore

In this step you will use **SAS Visual Analytics** and its built-in **Copilot** to visually explore the Analytical Base Table (ABT) you created in Step 2. The goal is to understand the patterns behind customer churn before building a predictive model.

---

## Prerequisites

The analytical base table should already be loaded into the **Public** CAS library. If you completed Step 2 the data was saved as `retail_abt.csv`. Your bootcamp environment has this pre-loaded as a CAS table named **`RETAIL_ABT`** in the **Public** caslib.

---

## Accessing the Data in SAS Visual Analytics

1. Open **SAS Visual Analytics** from the SAS Viya home page
2. Click **New Report**
3. In the data panel click on the Add Data button and from the available tables please select **RETAIL_ABT**
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

Work through the following questions to build your understanding of the churn patterns. For each question, try creating the visualization manually **and/or** by asking the Copilot.


Please do not feel obligated to answer all of these questions, rather pick and choose questions that interest you, help you advance your understanding of the data or sound challenging for your current level.

You are also more than welcome to just explore the data on your own making use of SAS Visual Analytics and the SAS Viya Copilot.

### Understanding the Target Variable

**Goal:** Get a baseline understanding of churn in the dataset.

- *"Show me the distribution of churned customers"*
- *"What percentage of customers have churned?"*

Create a **bar chart** or **pie chart** of the `churned` variable. You should see roughly 30-31% churned and 69-70% active — confirming this is a moderately imbalanced classification problem.

### Hypothesis 1: Engagement Drives Retention

**Goal:** Validate whether low engagement correlates with churn.

- *"Compare the average session duration between churned and active customers"*
- *"Show me total sessions by churn status"*
- *"What is the average pages viewed per session for churned vs. active customers?"*

Create **box plots** of `avg_session_duration`, `total_sessions`, and `avg_pages_per_session` grouped by `churned`. Look for clear separation between the two groups.

> **What to look for:** Churned customers should have noticeably lower session counts and shorter session durations.

### Hypothesis 2: Purchase Frequency Matters

**Goal:** Determine whether purchase behavior predicts churn.

- *"Show me the distribution of days since last purchase by churn status"*
- *"Compare total spend between churned and active customers"*
- *"How does `total_transactions` compare between churned and active customers?"*

Create a **histogram** of `days_since_last_purchase` colored by `churned`. Also create a **box plot** of `total_spend` by `churned`.

> **What to look for:** Churned customers should have a much higher `days_since_last_purchase` and much lower `total_spend` — these are typically the strongest predictors. `purchase_frequency` is normalized by tenure (transactions per month the customer was active), so it is not a clean churn signal in this dataset; rely on the recency and total-spend columns instead.

### Hypothesis 3: Support Issues Indicate Risk

**Goal:** Explore whether support interactions signal churn risk.

- *"Show me the average satisfaction score for churned vs. active customers"*
- *"How many high-priority tickets do churned customers have compared to active ones?"*
- *"Is there a relationship between resolution time and churn?"*

Create a **heat map** or **crosstab** of `total_tickets` and `high_priority_tickets` by `churned`. Create a **box plot** of `avg_satisfaction_score` by `churned`.

> **What to look for:** Lower satisfaction scores and more high-priority tickets among churned customers.

### Hypothesis 4: Subscription Tier Impacts Churn

**Goal:** Check if tier affects churn rate.

- *"What is the churn rate by subscription tier?"*
- *"Compare churn rates across Basic, Standard, and Premium tiers"*

Create a **stacked bar chart** showing churn proportions for each tier (use the `tier_Basic`, `tier_Standard`, `tier_Premium` columns).

> **What to look for:** Basic tier customers should have a meaningfully higher churn rate than Premium customers.

### Hypothesis 5: Email Opt-Out Signals Disengagement

**Goal:** Test whether opting out of emails predicts churn.

- *"Show me churn rate by email opt-in status"*

Create a **bar chart** of churn rate by `email_opt_in`.

> **What to look for:** Customers who opted out of email (email_opt_in = 0) are expected to have higher churn.

### Correlation and Multi-Variable Exploration

**Goal:** Find feature interactions and the strongest predictors.

- *"Which features are most correlated with churn?"*
- *"Show me a correlation matrix of the top 10 numeric features"*
- *"Create a decision tree to show which factors split churned from active customers"*

Use the Copilot's **automated analysis** feature to let it scan for the strongest relationships.

> **What to look for:** The Copilot may surface interactions you wouldn't have checked manually, such as "customers with low session duration AND high cart abandonment rate have a 90% churn probability."

---

## Building Your Report

As you work through the questions above, organize your findings into a report:

1. **Overview Page:** Churn distribution, key summary stats
2. **Engagement Page:** Session and browsing behavior by churn status
3. **Purchase Page:** Transaction patterns, recency, frequency
4. **Support Page:** Ticket metrics, satisfaction scores
5. **Demographics Page:** Tier, email opt-in, age distributions

Use **filters** and **interactions** between visualizations — clicking a bar in one chart should filter the others. This lets you drill into segments like "Basic tier customers who haven't purchased in 60+ days."

---

## Key Takeaways to Carry Forward

Before moving on to Step 4, summarize what you've learned:

1. **Which hypotheses were confirmed?** (likely H1, H2, and H3)
2. **Which features show the strongest separation** between churned and active customers?
3. **Are there any surprising patterns** the Copilot surfaced?
4. **What is the class balance?** (important for model training strategy)

These insights will directly inform the model building approach in the next step.

Finally feel free to save the report, the default location is My Folder, which is ideal here as to not clutter up the workspace for everybody else. You can also give it a name so that it is easier to remember what this report is about.

---

## Next Steps

Proceed to **[Step 4: Model](../4-model/)** to build and compare predictive models in SAS Model Studio.

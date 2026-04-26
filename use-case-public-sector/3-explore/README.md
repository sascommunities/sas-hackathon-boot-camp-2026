# Step 3: Explore

In this step you will use **SAS Visual Analytics** and its built-in **Copilot** to visually explore the Analytical Base Table (ABT) you created in Step 2. The goal is to understand the patterns behind service request urgency and identify equity gaps across districts before building a predictive model.

---

## Prerequisites

The analytical base table should already be loaded into the **Public** CAS library. If you completed Step 2 the data was saved as `public_sector_abt.csv`. Your bootcamp environment has this pre-loaded as a CAS table named **`PUBLIC_SECTOR_ABT`** in the **Public** caslib.

---

## Accessing the Data in SAS Visual Analytics

1. Open **SAS Visual Analytics** from the SAS Viya home page
2. Click **New Report**
3. In the data panel click on the Add Data button and from the available tables please select **PUBLIC_SECTOR_ABT**
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

### Copilot Tips and Caveats

A few behaviors to be aware of while working through this step:

- **Reference columns by their exact name.** The prompts in this guide use backtick-quoted column names (e.g., `` `is_urgent` ``, `` `district_avg_response_time` ``). Copilot works best when you do the same. The ABT carries both the raw columns (`request_type`, `department`, `location_district`) and the engineered features derived from them (`inherent_urgency`, `dept_avg_*`, `district_*`) — pick the column that matches the question you're asking.
- **Charts sometimes land on a different page.** If a generated visualization appears on another report page, drag it to the page you are working on.
- **Ignore suggestions to reclassify numeric measures as categories.** Copilot occasionally recommends changing numeric columns (e.g., `district_avg_request_count`) to categories. Decline those suggestions — they are measures and should stay that way.
- **If a chart doesn't answer the question, rephrase.** Ask Copilot for a specific chart type and specific column roles rather than an open-ended question (e.g., *"Create a bar chart with `inherent_urgency` on the x-axis and average `is_urgent` on the y-axis"*).

---

## Guided Exploration: Questions to Ask

Work through the following questions to build your understanding of service request urgency patterns. For each question, try creating the visualization manually **and/or** by asking the Copilot.

### Understanding the Target Variable

**Goal:** Get a baseline understanding of urgency in the dataset.

- *"Show me the distribution of urgent vs. non-urgent service requests"*
- *"What percentage of requests are classified as urgent?"*

Create a **bar chart** or **pie chart** of the `is_urgent` variable. Examine the class balance — this will inform your modeling strategy in Step 4.

### Hypothesis 1: Inherent Urgency Predicts Urgency

**Goal:** Validate that the `inherent_urgency` flag (derived from request type during feature engineering) is a strong predictor of whether a request is truly urgent.

- *"Show the average of `is_urgent` grouped by `inherent_urgency`"*
- *"Compare the distribution of `response_time_hours` for `inherent_urgency`=1 vs `inherent_urgency`=0"*
- *"What percentage of requests with `inherent_urgency`=1 are marked `is_urgent`=1?"*
- *"Show the average of `is_urgent` grouped by `request_type`"*

Create a **bar chart** of `is_urgent` (as a measure, aggregated as average) grouped by `inherent_urgency`. The `inherent_urgency` flag was derived during Step 2 from a curated list of safety-related request types (water main breaks, gas leaks, etc.); the raw `request_type` column is also retained in the ABT, so you can drill into specific request categories that drive the signal.

> **What to look for:** Requests flagged as inherently urgent (safety hazards, water main breaks) should have a much higher average `is_urgent` value than non-inherent ones. If the gap is small, the flag is not pulling its weight as a predictor.

### Hypothesis 2: Department Capacity Affects Response

**Goal:** Determine whether department workload and staffing affect response times.

- *"Show the correlation between `dept_avg_staff_count` and `dept_avg_response_time`"*
- *"Show the correlation between `dept_avg_budget_util` and `dept_avg_resolution_rate`"*
- *"Show the distribution of `dept_avg_overtime` across requests"*
- *"Show the average of `response_time_hours` grouped by `department`"*

Create a **scatter plot** of `dept_avg_staff_count` (x) vs. `dept_avg_response_time` (y). Create a second scatter plot of `dept_avg_overtime` (x) vs. `dept_avg_resolution_rate` (y). The ABT keeps both the raw `department` column and the engineered `dept_avg_*` aggregates — use the aggregates for capacity-vs-response correlation analysis and the raw column when you want a per-department breakdown.

> **What to look for:** Negative correlation between staff count and response time (more staff = faster response). Departments with high overtime and low resolution rates are the bottlenecks.

### Hypothesis 3: District Equity

**Goal:** Identify whether some districts consistently receive slower or worse service.

- *"Show the distribution of `district_avg_response_time` across requests"*
- *"Show the correlation between `district_avg_request_count` and `district_avg_response_time`"*
- *"Show the correlation between `district_avg_resolution_rate` and `response_time_hours`"*
- *"Show the sum of `district_total_critical` and `district_total_high` across all requests"*
- *"Show the average of `response_time_hours` grouped by `location_district`"*

Create a **histogram** of `district_avg_response_time` to see the spread of district-level service speed. Create a **scatter plot** of `district_avg_request_count` (x) vs. `district_avg_response_time` (y) to check whether high-volume districts are slower. The ABT keeps both the raw `location_district` label and the engineered `district_*` aggregates — use the aggregates to look at the overall spread of service speed, and the raw column when you want to call out specific districts that fall behind.

> **What to look for:** Wide spread in `district_avg_response_time` indicates the 40% variance problem is real. A strong positive correlation with request volume suggests capacity, not bias, is the driver; a weak correlation suggests uneven service allocation.

### Hypothesis 4: Seasonal Patterns

**Goal:** Explore whether request volumes and urgency vary by season.

- *"Show the count of requests grouped by `submit_month`"*
- *"Show the average of `is_urgent` grouped by `submit_month`"*
- *"Show the average of `response_time_hours` grouped by `submit_quarter`"*

Create a **line chart** with `submit_month` on the x-axis and count of requests on the y-axis. Create a second chart with `submit_month` on the x-axis and average `is_urgent` as a secondary measure.

> **What to look for:** Monthly spikes in volume or urgency point to seasonal pressure on departments. Months with high average `response_time_hours` are the stressed periods.

### Hypothesis 5: Citizen Satisfaction Patterns

**Goal:** Understand what drives `citizen_satisfaction` and how it relates to urgency.

- *"Show the distribution of `citizen_satisfaction`"*
- *"Show the average of `citizen_satisfaction` grouped by `is_urgent`"*
- *"Show the correlation between `response_time_hours` and `citizen_satisfaction`"*
- *"Show the correlation between `citizen_previous_requests` and `citizen_satisfaction`"*

Create a **scatter plot** with `response_time_hours` on the x-axis and `citizen_satisfaction` on the y-axis. Create a **bar chart** of average `citizen_satisfaction` grouped by `is_urgent`.

> **What to look for:** A strong negative correlation between `response_time_hours` and `citizen_satisfaction` (faster = happier). `is_urgent`=1 requests resolved quickly should maintain satisfaction; slow urgent requests are the worst customer experience.

### Equity-Focused Deep Dive

**Goal:** Specifically assess service equity across district aggregates and age groups.

- *"Show the scatter plot of `district_avg_response_time` vs `response_time_hours` filtered where `is_urgent`=1"*
- *"Show the correlation between `age_65+` and `citizen_satisfaction`"*
- *"Show the correlation between `age_18-24` and `citizen_satisfaction`"*
- *"Show the correlation between `district_avg_request_count` and `district_avg_resolution_rate`"*

Create a **scatter plot** of `district_avg_response_time` vs. the individual `response_time_hours`, filtered to `is_urgent`=1. Create a **bar chart** showing average `citizen_satisfaction` for each age-group dummy (`age_18-24` through `age_65+`, where the value is 1).

> **What to look for:** If districts with high `district_avg_response_time` also have high individual `response_time_hours` for urgent requests, the slowness is systemic. If certain age-group dummies have markedly lower average satisfaction, that is a demographic equity signal to carry into the fairness assessment in Step 4.

### Correlation and Multi-Variable Exploration

**Goal:** Find feature interactions and the strongest predictors of `is_urgent`.

- *"Show the correlation matrix for `is_urgent`, `inherent_urgency`, `response_time_hours`, `dept_avg_response_time`, `district_avg_response_time`, `citizen_satisfaction`"*
- *"Show the variable importance of all features for predicting `is_urgent`"*
- *"Create a decision tree with `is_urgent` as the target"*

Use the Copilot's **automated analysis** feature to scan for the strongest relationships.

> **What to look for:** `inherent_urgency` is likely the single strongest predictor. Watch for interactions — e.g., certain `submit_month` values combined with high `district_avg_request_count` may produce consistently slower response.

---

## Building Your Report

As you work through the questions above, organize your findings into a report:

1. **Overview Page:** `is_urgent` distribution, key summary stats
2. **Urgency Patterns Page:** `is_urgent` by `inherent_urgency`, temporal patterns from `day_of_week` / `submit_month`
3. **Department Performance Page:** `dept_avg_response_time`, `dept_avg_staff_count`, `dept_avg_overtime` relationships
4. **District Equity Page:** `district_avg_response_time`, `district_avg_resolution_rate`, `district_total_critical` / `district_total_high`
5. **Citizen Experience Page:** `citizen_satisfaction` drivers, age-group dummies, `citizen_engagement_score`

Use **filters** and **interactions** between visualizations — clicking a bar in one chart should filter the others. This lets you drill into segments like "urgent requests in the Downtown district during summer months."

---

## Key Takeaways to Carry Forward

Before moving on to Step 4, summarize what you have learned. Expected answers (if your exploration went well) are shown alongside each question — if your numbers differ substantially, revisit the relevant hypothesis before continuing.

1. **Which hypotheses were confirmed?** Expect **H1** (inherent urgency is the single strongest predictor), **H3** (wide spread in `district_avg_response_time`), and **H5** (strong negative correlation between `response_time_hours` and `citizen_satisfaction`). H2 and H4 tend to be weaker signals in this dataset.
2. **Which features show the strongest separation** between `is_urgent`=1 and `is_urgent`=0? Expect `inherent_urgency`, `response_time_hours`, `district_total_critical`, and `district_total_high` to separate the two classes most clearly.
3. **Which districts have the biggest equity gaps?** The highest values of `district_avg_response_time` combined with the lowest values of `district_avg_resolution_rate` mark the districts with the biggest service gap.
4. **What is the class balance?** Roughly **36% `is_urgent`=1** and **64% `is_urgent`=0** — moderately imbalanced, but not so skewed that you must rebalance before modeling.
5. **Are there any surprising patterns?** Common surprises: weekend submissions have longer response times; certain `submit_month` values show spiky urgency; high `citizen_engagement_score` does not always correlate with higher `citizen_satisfaction`.

If Copilot did not produce clear answers, the most common causes (in order) are: (a) your prompt did not reference exact column names, (b) the chart landed on another page — check all pages, or (c) the underlying ABT was not fully loaded in memory. Ask a SAS mentor if you are stuck.

These insights will directly inform the model building approach and fairness assessment in the next step.

Finally feel free to save the report, the default location is My Folder, which is ideal here as to not clutter up the workspace for everybody else. You can also give it a name so that it is easier to remember what this report is about.

---

## Next Steps

Proceed to **[Step 4: Model](../4-model/)** to build and compare predictive models in SAS Model Studio.

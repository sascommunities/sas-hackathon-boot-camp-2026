# Step 4: Model

In this step you will use **SAS Model Studio** to build, compare, and evaluate churn prediction models. SAS Model Studio provides a visual pipeline interface, a built-in **Copilot**, **AutoML** capabilities, and tools for custom model building — all with integrated fairness assessment. At the end you will register your champion model to **SAS Model Manager**.

---

## Prerequisites

The analytical base table (`RETAIL_ABT`) should be available in the **Public** CAS library. If you went through Step 2 & Step 3 before this one you will have a comprehensive understanding of the data already, if not take a second longer to read through the columns to get an understanding of the data.

---

## Opening SAS Model Studio

1. From the SAS Viya home page, open **SAS Model Studio** (under *Build Models* in the main menu)
2. Click **New Project**
3. Configure the project:
   - **Name:** *ShopEase Customer Churn Prediction*
   - **Project Type:** *Data Mining and Machine Learning*
   - **Data Source:** Select `RETAIL_ABT` from the Public caslib
   - Leave the **Template, Location & Description** as they are by default
   - **Target Variable:** `churned`
4. Click **Save**
5. Once the SAS Model Studio project has opened on the Data tab, select the variable `churned` and set its Role to ´Target´
6. On the same tab look for the variable `tier_Basic` and activate the checkbox `Assess this variable for bias`, this will enable us to talk more about the Trustworthy AI features of SAS Model Studio

The project is now ready for us to start modelling.

---

## Using the SAS Model Studio Copilot

SAS Model Studio includes a **Copilot** that acts as your AI-powered modeling assistant. Access it from the Copilot icon in the toolbar.

### What the Copilot Can Do

- **Recommend pipeline configurations** — ask it to suggest the best approach for a binary classification problem with imbalanced data
- **Explain model results** — ask it to interpret feature importance, model comparison metrics, or fairness reports
- **Generate pipeline nodes** — describe what you want and the Copilot can add nodes to your pipeline
- **Answer methodology questions** — ask about techniques like "What is gradient boosting?" or "How does SMOTE work?"

### Example Copilot Prompts

- *"Set up an AutoML pipeline for binary classification with class imbalance handling"*
- *"Add a variable selection node that removes features with importance below 1%"*
- *"Explain the difference between AUC-ROC and F1 score for this churn problem"*
- *"What does the misclassification rate tell me about my model?"*
- *"How should I handle the 69/31 class split in this dataset?"*

---

## Approach 1: AutoML (Recommended Starting Point)

AutoML automatically trains and compares multiple algorithms, handles preprocessing, and selects the best model. This is the fastest way to establish a strong baseline.

### Setting Up AutoML

1. In your project pipeline, click **New Pipeline** and select **Automatically generate the pipeline**
2. Configure AutoML settings:
   - **Maximum Time:** Set a time budget (e.g., 5 minutes for the bootcamp)
3. Click **Save**
4. Now the Pipeline is going to be built for us.

### What AutoML Does

AutoML will automatically:

- Test multiple algorithms (logistic regression, decision trees, random forests, gradient boosting, neural networks, support vector machines)
- Handle feature preprocessing (missing value imputation, encoding)
- Tune hyperparameters
- Compare models on the validation set
- Rank models by the chosen optimization metric

### Reviewing AutoML Results

After the run completes:

1. Open the **Model Comparison** node to see all models ranked by AUC
2. Click on individual models to see:
   - **Fit Statistics:** AUC, misclassification rate, Gini coefficient, KS statistic
   - **ROC Chart:** Visual comparison of model discrimination
   - **Variable Importance:** Which features matter most
   - **Score Rankings:** How well the model separates high-risk from low-risk customers
3. The top model is automatically selected as the **champion**

> **What to expect:** For this dataset, gradient boosting or random forest models typically achieve an AUC of 0.82-0.88. Logistic regression is usually close behind at 0.78-0.83.

---

## Approach 2: Custom Pipeline Building

If you want more control — or want to experiment with specific model types — build a custom pipeline step by step.

### Recommended Pipeline Nodes

Add these nodes in sequence by clicking **Add Node** in the pipeline canvas:

1. **Data** — Your input table (already connected)
3. **Imputation** — Handle any remaining missing values (mean for numeric, mode for categorical)
5. **Model Nodes** — Add one or more of the following:
   - **Logistic Regression** — Interpretable baseline model
   - **Forest** — Random forest ensemble
   - **Gradient Boosting** — Often the best performer
6. **Model Comparison** — Automatically compares all model nodes and selects the champion

### Configuring Individual Models

**Logistic Regression:**
- Selection-process stopping criterion: Significance level
- Entry significance level: 0.05
- This gives you interpretable coefficients for each feature

**Forest (Random Forest):**
- Number of trees: 200
- Maximum depth: 10
- Go to the Post-training Properties and activate *HyperSHAP* under the Local Interpretability section. This is used to explain the model predictions

**Gradient Boosting:**
- Number of trees: 150
- Learning rate: 0.1
- Maximum depth: 5
- Subsample rate: 0.8

### Running the Custom Pipeline

1. Verify all nodes are connected in the pipeline canvas
2. Click **Run Pipeline** (or right-click the Model Comparison node and select *Run*)
3. Wait for all models to finish training

---

## Comparing Models

Once both (or either) approach has completed, open the **Model Comparison** results:

| Metric | What It Tells You | Target |
|--------|-------------------|--------|
| **AUC-ROC** | Overall ability to distinguish churned from active | ≥ 0.80 |
| **Misclassification Rate** | Percentage of incorrect predictions | ≤ 0.20 |
| **KS Statistic** | Maximum separation between cumulative distributions | ≥ 0.40 |
| **Gini Coefficient** | 2 x AUC - 1; measures discrimination power | ≥ 0.60 |
| **F1 Score** | Harmonic mean of precision and recall | ≥ 0.70 |

Review the **ROC Overlay Chart** to visually compare how well each model separates the two classes. The model closest to the top-left corner performs best.

Review the **Variable Importance** chart for your champion model — the top predictors should align with what you found in Step 3 (likely `days_since_last_purchase`, `total_sessions`, `avg_session_duration`, `total_spend`).

---

## Fairness Assessment

Trustworthy AI requires that models do not discriminate unfairly against protected groups. In this use case we will assess fairness with respect to **subscription tier** — specifically whether the model treats Basic-tier customers equitably compared to Premium-tier customers.

### Why Subscription Tier?

While subscription tier is not a legally protected class (like race or gender), it can serve as a **proxy for economic status**. If the model systematically flags Basic-tier customers as high churn risk while giving Premium customers a pass, the resulting retention campaigns could:

- Over-target lower-spend customers with aggressive discounting, eroding margins
- Under-serve higher-spend customers who are silently disengaging
- Create a self-fulfilling prophecy where Basic customers receive "we think you're leaving" messaging that actually drives them away

Fairness analysis ensures the model performs **equitably across segments** and that business actions based on model scores don't inadvertently discriminate.

### Running the Fairness Assessment

1. As we set the `tier_Basic` variable to be assess for fairness we get this assessments with every model
4. Review the fairness metrics:

| Metric | What It Measures | Acceptable Range |
|--------|-----------------|------------------|
| **Demographic Parity** | Are churn predictions distributed equally across tiers? | Ratio > 0.80 |
| **Equal Opportunity** | Is the true positive rate similar across tiers? | Difference < 0.10 |
| **Predictive Parity** | Is the precision similar across tiers? | Difference < 0.10 |
| **Calibration** | Does a 70% predicted probability mean 70% actual churn for both groups? | Slope close to 1.0 |

### Interpreting Results

- If **Demographic Parity** is below 0.80, the model disproportionately flags one tier as high-risk
- If **Equal Opportunity** difference exceeds 0.10, the model catches churners in one tier better than the other
- Review the **Score Distribution** by group — both groups should have similar shaped curves

### The Value of Fairness Assessment

Fairness assessment is not just an ethical checkbox — it provides **direct business value**:

1. **Better decisions:** A model that performs equally well across segments leads to more effective retention campaigns for everyone
2. **Trust:** Stakeholders (marketing, customer success, finance) trust models more when they can see fairness evidence
3. **Risk reduction:** Proactively identifying bias prevents costly remediation later
4. **Regulatory readiness:** As AI regulations expand, documented fairness assessment becomes a compliance requirement

> **Tip:** Ask the Copilot *"Is my model fair across subscription tiers?"* to get a plain-language interpretation of the fairness metrics.

---

## Registering to SAS Model Manager

Once you have selected your champion model and reviewed its fairness, register it to **SAS Model Manager** for governance, version control, and deployment.

### Steps to Register

1. In the Pipeline Comparison tab, identify your overall **champion model** (the one with the best KS (Youden))
2. Right-click the champion model and select **Register Model** (or use the menu: *Actions* > *Register Model*)
3. Confirm the Location which is /Model Repositories/DM Repository and click OK
4. Wait for the registration to finish in this pop up, then you can close and right click the model again and select **Manage Models**
5. Now we will be navigated into SAS Model Manager where we can review the Model Card of this model
6. Explore the Model Card that is populated automatically as you develop and manage the model on SAS Viya. The Overview tab offers a high-level summary of the model, including an overview of the model's training accuracy, training fairness, generalizability, and influential variables.

### What Registration Provides

Once registered in SAS Model Manager, your model benefits from:

- **Version control:** Track changes across model iterations
- **Performance monitoring:** Set up automated performance tracking over time
- **Governance:** Maintain an audit trail of who built the model, what data was used, and what fairness checks were performed
- **Deployment readiness:** The model can be published to CAS, MAS (Micro Analytic Service), or a container for scoring
- **Model card:** Auto-generated documentation capturing inputs, outputs, performance, and lineage

> **Tip:** Ask the Copilot *"Register this model to Model Manager"* and it will walk you through the process.

---

## Summary

At this point you have:

1. Built models using AutoML and/or custom pipelines
2. Compared models on AUC, misclassification rate, and other metrics
3. Assessed fairness across subscription tiers
4. Registered your champion model to SAS Model Manager
5. Viewed the Model Card in SAS Model Manager

---

## Next Steps

Proceed to **[Step 5: Deploy & Act](../5-deploy-act/)** to create a decision flow in SAS Intelligent Decisioning that operationalizes your model.

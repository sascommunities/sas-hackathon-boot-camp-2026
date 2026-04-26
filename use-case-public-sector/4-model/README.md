# Step 4: Model

In this step you will use **SAS Model Studio** to build, compare, and evaluate service request urgency prediction models. SAS Model Studio provides a visual pipeline interface, a built-in **Copilot**, **AutoML** capabilities, and tools for custom model building — all with integrated fairness assessment. At the end you will register your champion model to **SAS Model Manager**.

---

## Prerequisites

The analytical base table (`PUBLIC_SECTOR_ABT`) should be available in the **Public** CAS library. If you went through Step 2 & Step 3 before this one you will have a comprehensive understanding of the data already, if not take a second longer to read through the columns to get an understanding of the data.

---

## Opening SAS Model Studio

1. From the SAS Viya home page, open **SAS Model Studio** (under *Build Models* in the main menu)
2. Click **New Project**
3. Configure the project:
   - **Name:** *Metro City Service Request Urgency Prediction*
   - **Project Type:** *Data Mining and Machine Learning*
   - **Data Source:** Select `PUBLIC_SECTOR_ABT` from the Public caslib
   - Leave the **Template, Location & Description** as they are by default
   - **Target Variable:** `is_urgent`
4. Click **Save**
5. Once the SAS Model Studio project has opened on the Data tab, select the variable `is_urgent` and set its Role to ´Target´
6. On the same tab look for the variable `age_65p` and activate the checkbox `Assess this variable for bias`, this will enable us to talk more about the Trustworthy AI features of SAS Model Studio

The project is now ready for us to start modelling.

---

## Using the SAS Model Studio Copilot

SAS Model Studio includes a **Copilot** that acts as your AI-powered modeling assistant. Access it from the Copilot icon in the toolbar.

### What the Copilot Can Do

- **Recommend pipeline configurations** — ask it to suggest the best approach for a binary classification problem where recall on the positive class is critical
- **Explain model results** — ask it to interpret feature importance, model comparison metrics, or fairness reports
- **Generate pipeline nodes** — describe what you want and the Copilot can add nodes to your pipeline
- **Answer methodology questions** — ask about techniques like "What is gradient boosting?" or "How does oversampling work?"

### Example Copilot Prompts

- *"Set up an AutoML pipeline for binary classification where we cannot afford to miss urgent requests"*
- *"Add a variable selection node that removes features with importance below 1%"*
- *"Explain the difference between precision and recall for this urgency prediction problem"*
- *"What does the F1 score tell me about my model's balance between precision and recall?"*
- *"How should I handle class imbalance if urgent requests are the minority class?"*

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

1. Open the **Model Comparison** node to see all models ranked by the optimization metric
2. Click on individual models to see:
   - **Fit Statistics:** AUC, misclassification rate, Gini coefficient, KS statistic
   - **ROC Chart:** Visual comparison of model discrimination
   - **Variable Importance:** Which features matter most
   - **Score Rankings:** How well the model separates urgent from non-urgent requests
3. The top model is automatically selected as the **champion**

> **What to expect:** For this dataset, gradient boosting or random forest models typically achieve an AUC of 0.85-0.92. The key metric to watch is recall on the urgent class — the target is greater than 90%, meaning the model should catch at least 9 out of 10 truly urgent requests.

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
| **Accuracy** | Overall percentage of correct predictions | > 85% |
| **Recall (Sensitivity)** | Proportion of actual urgent requests correctly identified | > 90% |
| **AUC-ROC** | Overall ability to distinguish urgent from non-urgent | >= 0.85 |
| **Misclassification Rate** | Percentage of incorrect predictions | <= 0.15 |
| **F1 Score** | Harmonic mean of precision and recall | >= 0.80 |
| **KS Statistic** | Maximum separation between cumulative distributions | >= 0.50 |

Review the **ROC Overlay Chart** to visually compare how well each model separates the two classes. The model closest to the top-left corner performs best.

Review the **Variable Importance** chart for your champion model — the top predictors should align with what you found in Step 3 (likely `inherent_urgency`, `response_time_hours`, `dept_avg_response_time`, `district_avg_response_time`, `district_total_critical`).

**Critical note on recall:** In this use case, a false negative (missing a truly urgent request) is far worse than a false positive (escalating a non-urgent request). If you must choose between a model with 95% accuracy and 85% recall versus one with 88% accuracy and 93% recall, prefer the second model. Missing urgent requests can mean delayed response to safety hazards, water main breaks, or other critical situations.

---

## Fairness Assessment

Trustworthy AI requires that models do not discriminate unfairly against certain groups. In this use case we will assess fairness with respect to **`age_65p`** — the age-group dummy indicating whether the citizen submitting the request is 65 or older.

### Why Age 65+?

Senior citizens are a sensitive demographic in public service delivery. They may rely more heavily on municipal services, face barriers to submitting digital requests, and be less able to tolerate delays on safety-related issues (heating outages, sidewalk hazards, water problems). If the urgency prediction model systematically under-predicts urgency for requests from older citizens, the consequences are direct:

1. **Model under-predicts urgency** for requests where `age_65p`=1
2. Those requests are **deprioritized** in the triage queue
3. Response times for older citizens **increase**
4. Citizen satisfaction among older citizens **decreases**
5. Future data reflects **worse outcomes** for this group, reinforcing the bias

This is not a hypothetical concern. Automated decision systems have been documented to produce disparate impacts on protected demographic groups, and age is one of the protected classes under most municipal anti-discrimination frameworks.

> **Note on district equity:** The raw `location_district` column is retained in the ABT alongside engineered district-level aggregates (`district_avg_response_time`, `district_avg_resolution_rate`, etc.) so the Step 5 decision flow can pass it through, and district equity is reflected in the model through those aggregates (you explored this in Step 3). Formal fairness assessment in Model Studio works best with a single binary/categorical column present on each row — which is why we use the `age_65p` dummy here rather than `location_district`.

Fairness assessment ensures the model performs **equitably across age groups** and that the resulting triage decisions do not systematically disadvantage older citizens.

### Running the Fairness Assessment

1. As we set the `age_65p` variable to be assessed for fairness we get this assessment with every model
2. Review the fairness metrics:

| Metric | What It Measures | Acceptable Range |
|--------|-----------------|------------------|
| **Demographic Parity** | Are urgency predictions distributed equally across age groups? | Ratio > 0.80 |
| **Equal Opportunity** | Is the true positive rate (recall for urgent) similar for 65+ vs under-65? | Difference < 0.10 |
| **Predictive Parity** | Is the precision similar for 65+ vs under-65? | Difference < 0.10 |
| **Calibration** | Does a 70% predicted probability mean 70% actual urgency for both groups? | Slope close to 1.0 |

### Interpreting Results

- If **Demographic Parity** is below 0.80, the model disproportionately flags one age group as urgent while under-flagging the other
- If **Equal Opportunity** difference exceeds 0.10, the model catches urgent requests for one age group better than the other — meaning one group gets worse triage
- Review the **Score Distribution** by group — both groups should have similarly shaped curves

### The Value of Fairness Assessment

Fairness assessment in public sector AI is not just an ethical consideration — it is a governance requirement with direct impact on citizens:

1. **Equitable service:** Every demographic group deserves the same quality of urgency prediction
2. **Public trust:** Citizens trust government algorithms more when there is documented evidence of fairness testing
3. **Legal compliance:** Algorithmic bias prevention is increasingly part of municipal AI governance frameworks and may be required under local ordinances
4. **Accountability:** Documented fairness assessment provides an audit trail for public records requests and oversight bodies
5. **Better outcomes:** A model that performs equitably across groups leads to more effective resource allocation citywide

> **Tip:** Ask the Copilot *"Is my model fair across age groups?"* to get a plain-language interpretation of the fairness metrics.

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
2. Compared models on accuracy, recall, AUC, and other metrics
3. Assessed fairness across age groups (`age_65p`) to ensure equitable service for older citizens
4. Registered your champion model to SAS Model Manager
5. Viewed the Model Card in SAS Model Manager

---

## Next Steps

Proceed to **[Step 5: Deploy & Act](../5-deploy-and-act/)** to create a decision flow in SAS Intelligent Decisioning that operationalizes your model.

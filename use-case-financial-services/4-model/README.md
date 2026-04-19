# Step 4: Model

In this step you will use **SAS Model Studio** to build, compare, and evaluate loan default prediction models. SAS Model Studio provides a visual pipeline interface, a built-in **Copilot**, **AutoML** capabilities, and tools for custom model building — all with integrated fairness assessment. At the end you will register your champion model to **SAS Model Manager** with adverse action code documentation.

---

## Prerequisites

The analytical base table (`FINANCIAL_SERVICES_ABT`) should be available in the **Public** CAS library. If you went through Step 2 & Step 3 before this one you will have a comprehensive understanding of the data already, if not take a second longer to read through the columns to get an understanding of the data.

---

## Opening SAS Model Studio

1. From the SAS Viya home page, open **SAS Model Studio** (under *Build Models* in the main menu)
2. Click **New Project**
3. Configure the project:
   - **Name:** *PremierBank Loan Default Prediction*
   - **Project Type:** *Data Mining and Machine Learning*
   - **Data Source:** Select `FINANCIAL_SERVICES_ABT` from the Public caslib
   - Leave the **Template, Location & Description** as they are by default
   - **Target Variable:** `defaulted`
4. Click **Save**
5. Once the SAS Model Studio project has opened on the Data tab, select the variable `defaulted` and set its Role to ´Target´
6. On the same tab look for the variable `inc_Low` and activate the checkbox `Assess this variable for bias`, this will enable us to talk more about the Trustworthy AI features of SAS Model Studio

The project is now ready for us to start modelling.

---

## Using the SAS Model Studio Copilot

SAS Model Studio includes a **Copilot** that acts as your AI-powered modeling assistant. Access it from the Copilot icon in the toolbar.

### What the Copilot Can Do

- **Recommend pipeline configurations** — ask it to suggest the best approach for a binary classification problem with imbalanced data
- **Explain model results** — ask it to interpret feature importance, model comparison metrics, or fairness reports
- **Generate pipeline nodes** — describe what you want and the Copilot can add nodes to your pipeline
- **Answer methodology questions** — ask about techniques like "What is gradient boosting?" or "Why is logistic regression preferred for regulatory models?"

### Example Copilot Prompts

- *"Set up an AutoML pipeline for binary classification with moderate class imbalance"*
- *"Add a variable selection node that removes features with importance below 1%"*
- *"Explain the difference between AUC-ROC and Gini coefficient for this default prediction problem"*
- *"Why would a regulator prefer logistic regression over a neural network?"*
- *"How should I handle the 74/26 class split in this dataset?"*
- *"What are adverse action reason codes and how do I generate them from this model?"*

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
   - **Score Rankings:** How well the model separates high-risk from low-risk loans
3. The top model is automatically selected as the **champion**

> **What to expect:** For this dataset, gradient boosting or random forest models typically achieve an AUC of 0.82-0.90. Logistic regression is usually close behind at 0.78-0.85.

---

## Approach 2: Custom Pipeline Building

If you want more control — or need a specific model type for regulatory reasons — build a custom pipeline step by step.

### Why Logistic Regression for Regulatory Models

In financial services, **logistic regression** holds a special status:

- **Interpretability:** Every coefficient maps directly to a feature's contribution to default probability, making it easy to explain to regulators and applicants
- **Adverse action codes:** The model's coefficients can be rank-ordered to produce the top reasons for a credit decision (required by FCRA/ECOA)
- **Regulatory familiarity:** Examiners are experienced with logistic regression and its validation methods
- **SR 11-7 compliance:** Model risk management is straightforward when the model is fully transparent

This does not mean you cannot use ensemble methods — but you should benchmark against logistic regression and be prepared to justify the incremental lift.

### Recommended Pipeline Nodes

Add these nodes in sequence by clicking **Add Node** in the pipeline canvas:

1. **Data** — Your input table (already connected)
3. **Imputation** — Handle any remaining missing values (mean for numeric, mode for categorical)
5. **Model Nodes** — Add one or more of the following:
   - **Logistic Regression** — Interpretable baseline (regulatory gold standard)
   - **Forest** — Random forest ensemble
   - **Gradient Boosting** — Often the best performer for AUC
6. **Model Comparison** — Automatically compares all model nodes and selects the champion

### Configuring Individual Models

**Logistic Regression:**
- Selection-process stopping criterion: Significance level
- Entry significance level: 0.05
- This gives you interpretable coefficients for each feature and directly supports adverse action reason code generation

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
| **AUC-ROC** | Overall ability to distinguish defaulted from current loans | >= 0.82 |
| **Misclassification Rate** | Percentage of incorrect predictions | <= 0.15 |
| **KS Statistic** | Maximum separation between cumulative distributions | >= 0.45 |
| **Gini Coefficient** | 2 x AUC - 1; measures discrimination power | >= 0.64 |
| **F1 Score** | Harmonic mean of precision and recall | >= 0.50 |

Review the **ROC Overlay Chart** to visually compare how well each model separates the two classes. The model closest to the top-left corner performs best.

Review the **Variable Importance** chart for your champion model — the top predictors should align with what you found in Step 3 (likely `credit_score`, `late_payment_rate`, `debt_to_income`, `severe_delinquency_flag`, `loan_to_value`).

---

## Fairness Assessment

Trustworthy AI requires that models do not discriminate unfairly against protected groups. In lending, this is not just an ethical best practice — it is a **legal requirement** under ECOA and the Fair Housing Act. In this use case we will assess fairness with respect to **income band** as a proxy variable.

### Why Income Band?

While income is not a legally protected class per se, it can serve as a **proxy for protected characteristics** such as race and national origin. Research has consistently shown that income is correlated with race and ethnicity in the United States. If the model systematically treats low-income borrowers differently in ways that are not justified by their actual credit risk, this could constitute **disparate impact** — a form of discrimination that is illegal even when unintentional.

Fair lending regulations require lenders to:

1. **Test for disparate impact:** Do approval rates or pricing differ across groups defined by protected characteristics or their proxies?
2. **Justify with business necessity:** If disparate impact exists, is it driven by legitimate credit risk factors?
3. **Search for less discriminatory alternatives:** Even if the impact is justified, is there a model or policy that achieves similar risk prediction with less disparate impact?

### Running the Fairness Assessment

1. As we set the `inc_Low` variable to be assess for fairness we get this assessments with every model
4. Review the fairness metrics:

| Metric | What It Measures | Acceptable Range |
|--------|-----------------|------------------|
| **Demographic Parity** | Are default predictions distributed equally across income bands? | Ratio > 0.80 |
| **Equal Opportunity** | Is the true positive rate (catching actual defaults) similar across income bands? | Difference < 0.10 |
| **Predictive Parity** | Is the precision similar across income bands? | Difference < 0.10 |
| **Calibration** | Does a 20% predicted default probability mean 20% actual default for both groups? | Slope close to 1.0 |

### Interpreting Results

- If **Demographic Parity** is below 0.80, the model disproportionately flags one income group as high-risk
- If **Equal Opportunity** difference exceeds 0.10, the model catches defaults in one income group better than the other
- Review the **Score Distribution** by group — both groups should have similarly shaped curves
- If you find evidence of disparate impact, consider:
  - Removing proxy variables
  - Reweighting the training data
  - Using a constrained optimization approach
  - Switching to a model that achieves similar AUC with less disparity

### The Value of Fairness Assessment in Lending

Fairness assessment provides both compliance and business value:

1. **Regulatory compliance:** Fair lending exams specifically look for disparate impact — documented fairness assessment is your defense
2. **Adverse action accuracy:** Fair models produce more accurate adverse action reason codes, reducing applicant confusion and complaint volume
3. **Legal risk reduction:** Proactively identifying and mitigating bias prevents costly enforcement actions and class-action lawsuits
4. **Community trust:** Demonstrable fairness strengthens the bank's reputation in the communities it serves
5. **Better decisions:** A model that performs equitably across segments produces more accurate risk assessments for everyone

> **Tip:** Ask the Copilot *"Is my model fair across income bands?"* to get a plain-language interpretation of the fairness metrics.

---

## Registering to SAS Model Manager

Once you have selected your champion model and reviewed its fairness, register it to **SAS Model Manager** for governance, version control, and deployment.

### Steps to Register

1. In the Pipeline Comparison tab, identify your overall **champion model** (the one with the best KS (Youden))
2. Right-click the champion model and select **Register Model** (or use the menu: *Actions* > *Register Model*)
3. Confirm the Location which is /Model Repositories/DM Repository and click OK
4. Wait for the registration to finish in this pop up, then you can close and right click the model again and select **Manage Models**
5. Now we will be navigated into SAS Model Manager where we can review the Model Card of this model
6. Explore the Model Card that is populated automatically as you develop and manage the model on SAS Viya. The Overview tab offers a high-level summary of the model, including an overview of the model’s training accuracy, training fairness, generalizability, and influential variables.

### What Registration Provides

Once registered in SAS Model Manager, your model benefits from:

- **Version control:** Track changes across model iterations
- **Performance monitoring:** Set up automated performance tracking over time
- **Governance:** Maintain an audit trail of who built the model, what data was used, and what fairness checks were performed
- **Deployment readiness:** The model can be published to CAS, MAS (Micro Analytic Service), or a container for scoring
- **Model card:** Auto-generated documentation capturing inputs, outputs, performance, and lineage — critical for SR 11-7 compliance

> **Tip:** Ask the Copilot *"Register this model to Model Manager"* and it will walk you through the process.

---

## Summary

At this point you have:

1. Built models using AutoML and/or custom pipelines
2. Compared models on AUC, Gini, KS, and other metrics
3. Assessed fairness across income bands for fair lending compliance
4. Registered your champion model to SAS Model Manager
5. Viewed the Model Card in SAS Model Manager

---

## Next Steps

Proceed to **[Step 5: Deploy & Act](../5-deploy-and-act/)** to create a loan decision flow in SAS Intelligent Decisioning that operationalizes your model.

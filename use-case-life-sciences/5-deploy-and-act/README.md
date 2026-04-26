# Step 5: Deploy & Act

In this final step you will use **SAS Intelligent Decisioning** to operationalize your readmission prediction model by embedding it in an automated discharge decision flow. You will also explore its **Copilot** and learn how decisions can function as **tools in agentic workflows** - or become agentic workflows themselves.

---

## Prerequisites

Your champion model should be registered in **SAS Model Manager** from Step 4. SAS Intelligent Decisioning will pull the model directly from the Model Manager registry. If you did not register your own do not worry a default one is provided.

---

## What is SAS Intelligent Decisioning?

SAS Intelligent Decisioning is the platform for creating, managing, and executing business decisions that combine analytical models, business rules, and contextual logic into a single decision flow. Instead of just scoring a patient with a model, a decision flow can:

- Score the patient's readmission probability
- Classify it into a risk tier
- Apply clinical rules (e.g., "always flag patients with 4+ comorbidities for care management review")
- Branch logic based on diagnosis category and discharge disposition
- Select the appropriate post-discharge care plan and intervention
- Return a complete discharge recommendation

This turns a model prediction into an **actionable clinical decision**.

If you have any questions around SAS Intelligent Decisioning activate the SAS Viya copilot within the application via the icon in the top right hand corner next to your profile or ask one of the onsite SAS Mentors.

---

## Creating a Discharge Readmission Risk Decision

### 1. Open SAS Intelligent Decisioning

1. From the SAS Viya main menu, navigate to **SAS Intelligent Decisioning** (under *Build Decisions*)
2. Click **New Decision**
3. Name it: *MedCare Discharge Readmission Risk Decision*
4. Leave the Description, Location and Workflow on default and click OK
5. Navigate to the *Variables* tab, click on the *Add variable* dropdown and either select *Custom variable* if you want to add them all yourself of *Decision* if you want to copy it from the template (this is faster). The manual steps are described in the below sub steps 1 & 2 while the copy is described in step 3:
    1. Define the **input variables** (these will be passed in when the decision is called):
        1. `patient_id` (character)
        2. `comorbidity_count` (decimal)
        3. `insurance_type` (character)
        4. `emergency_flag` (decimal)
        5. `polypharmacy_flag` (decimal)
        6. `clinical_risk_score` (decimal)
        7. `discharge_disposition` (character)
    2. Define the **output variables** (what the decision returns):
        1. `risk_tier` (character) - the patient's readmission risk level
        2. `care_plan` (character) - the recommended post-discharge care plan
        3. `intervention` (character) - specific intervention to execute
        4. `follow_up_days` (decimal) - when the first follow-up should occur
        5. `priority` (character) - urgency of care coordination
        6. `reason` (character) - why this care plan was selected
        7. Now click OK to add all of them
    3. Copy the **variables** from a template decision:
        1. Click on the folder icon in the *Decision* input field
        2. Navigate to *SAS Content > SAS Hackathon Bootcamp 2026 > Use Case Life Sciences* select *MedCare Discharge Readmission Risk Decision* and click OK
        3. Click on the *Add all* icon in the middle of the dialogue to bring all the variables into your decision and then click the Add button


From here you can also always activate the SAS Viya Copilot via the icon in the top right hand corner to ask questions about SAS Intelligent Decisioning to deepen your understanding of the application.

### 2. Add the Model Node

1. In the decision flow canvas, add a **Model** node
2. Select your registered champion model from SAS Model Manager (*MedCare Readmission Prediction - GAM*)
3. Map the input variables to the model's expected features
    1. For the inputs the `patient_id` should be mapped automatically and the remaining clinical features will align by name - review any that remain unmapped and connect them to the matching input variable
    2. For the outputs we are going to be clicking the *More* menu up top and select *Add missing variables* this will add all of the required output variables to our decision - if you copied the variables using the template they are already present - in the dialogue please make sure to deselect them from the Output as we will create our own custom outputs


### 3. Add Business Rules

After the model scores the patient, add **Rule Set** nodes to determine the care plan. For this make first sure that you have clicked the save icon of your decision and than we will be adding Rule Sets to our decision. There are two ways of doing this the easy way where you use the pre build rule sets by clicking on the three vertical dots on the model node and selecting *Add > Rule Set*, then in the dialogue navigate to *SAS Content > SAS Hackathon Bootcamp 2026 > Use Case Life Sciences* and add the rule set as specified below. If you want to create them yourself you can go to the right hand side click on the *Objects* and drag & drop a Rule Set onto the previous node. This will open up a dialogue where you should name your decision correspondingly, please leave the location as the default (*My Folder*) - then add the variables from the decision you created and start building the Rule Sets as described below - the required variables are noted either as the columns or in the **Rule Conditions**.

We recommend you try to build at least one of these rule sets yourself to get an understanding of how it is done. If you have any questions around SAS Intelligent Decisioning activate the SAS Viya copilot within the application via the icon in the top right hand corner next to your profile or ask one of the onsite SAS Mentors.

**Rule Set: Risk Tier Classification**

| Rule Conditions | risk_tier |
|-----------|-----------|
| P_readmitted_30days1 >= 0.80 | Critical |
| P_readmitted_30days1 >= 0.60 | High |
| P_readmitted_30days1 >= 0.40 | Moderate |
| P_readmitted_30days1 < 0.40 | Low |

**Rule Set: Care Plan Assignment**

| risk_tier | discharge_disposition | care_plan | follow_up_days |
|-----------|----------------------|-----------|----------------|
| Critical | Any | Intensive transitional care | 1 |
| High | Home | Home health referral + phone follow-up | 2 |
| High | SNF | SNF care coordination + pharmacy consult | 3 |
| High | Home with Services | Enhanced home health with daily check-ins | 2 |
| Moderate | Any | Standard follow-up protocol | 5 |
| Low | Any | Routine discharge (continue monitoring) | 14 |

**Rule Set: Intervention Assignment**

| Rule Conditions | intervention |
|-----------|-------------|
| polypharmacy_flag = 1 | Medication reconciliation by clinical pharmacist |
| clinical_risk_score >= 3 | Urgent PCP follow-up within 48 hours |
| emergency_flag = 1 AND comorbidity_count >= 3 | Care manager assignment + social work consult |
| comorbidity_count >= 4 | Chronic disease management enrollment |
| Otherwise | Automated discharge instructions + patient portal reminder |

**Rule Set: Priority Assignment**

| Rule Conditions | priority |
|-----------|----------|
| risk_tier = 'Critical' AND comorbidity_count >= 3 | Urgent |
| risk_tier = 'Critical' OR (risk_tier = 'High' AND polypharmacy_flag = 1) | High |
| risk_tier = 'High' OR risk_tier = 'Moderate' | Normal |
| risk_tier = 'Low' | Routine |

**Rule Set: Reason Assignment**

These rules capture the dominant driver behind the care plan and populate the `reason` variable used downstream by the LLM:

| Rule Conditions | reason |
|-----------|----------|
| emergency_flag = 1 AND comorbidity_count >= 3 | Multiple chronic conditions with emergency admission |
| polypharmacy_flag = 1 AND clinical_risk_score >= 3 | Complex medication regimen with elevated clinical risk |
| comorbidity_count >= 4 | Multiple concurrent chronic conditions |
| clinical_risk_score >= 3 | Elevated clinical risk score at discharge |
| polypharmacy_flag = 1 | Complex medication regimen |
| Otherwise | Standard discharge with routine monitoring |

### 4. Adding an LLM to the Mix

Now right click the last node in the decision and add first a Rule Set (see blow) assignment and then on that node a Call LLM node. The Call LLM node will ask us to add missing variables again (like we did for the model node) - only add the prompt variable  (not as an input) and for the XXX we will map our reason variable that we have been using all along - then click the save icon.

Next we are going back to the Rule Set node and will assign our prompt variable the following value - you have to go into the editing mode via the pencil icon:

```
prompt = 'You are a compassionate MedCare patient education specialist. Using the patient discharge data and care plan below, write a warm, respectful, and clearly structured long-form discharge summary (3 to 5 paragraphs) that a patient with no clinical background can read to understand their next steps after leaving the hospital. Do not expose clinical codes or jargon verbatim - translate them into plain, everyday language. Do not provide medical advice beyond what the care plan specifies, do not diagnose, and do not predict outcomes. Patient and care plan context: Assigned risk tier: ' + risk_tier + '. Recommended care plan: ' + care_plan + '. Specific intervention: ' + intervention + '. First follow-up in (days): ' + follow_up_days + '. Care coordination priority: ' + priority + '. Internal reason code: ' + reason + '. Structure your response as follows. First, open with a warm, personal acknowledgment that the patient is being discharged today and that MedCare is committed to supporting their recovery at home. Do not explicitly state the ' + risk_tier + ' risk tier - instead, frame the care plan as tailored to their individual situation. Second, explain the recommended care plan ' + care_plan + ' in plain language and describe what the patient can expect in the coming days. Third, explain the specific intervention ' + intervention + ' - including who will reach out, what the purpose is, and why it matters for their recovery. Translate the internal reason ' + reason + ' into an empathetic, plain-language explanation of why this extra level of support is being offered. Fourth, clearly communicate the follow-up timeline (within ' + follow_up_days + ' days), what kind of follow-up it is, and what the patient needs to do to prepare. Reflect the ' + priority + ' care coordination priority in the tone - more proactive and immediate for urgent priority, calmer and routine for standard priority. Fifth, close with clear guidance on warning signs that require immediate attention (calling 911 or going to the emergency room) and reassure the patient that their care team is available for questions. Tone: warm, respectful, reassuring, and never alarming or condescending. Length: 300 to 450 words. Write in the second person (you, your recovery). Always include a clear reminder to call 911 or go to the emergency room for any life-threatening symptoms.'
```

SAS provides the [https://github.com/sassoftware/sas-agentic-ai-accelerator](https://github.com/sassoftware/sas-agentic-ai-accelerator) which enables you to connect any LLM and do extensive prompt engineering & monitoring, but here we have a hard coded LLM (OpenAI GPT 5.4) available.

### 5. Test the Decision

1. Click **Test** in the toolbar
2. Enter sample values:
   - patient_id: P042
   - comorbidity_count: 4
   - insurance_type: Medicare
   - emergency_flag: 1
   - polypharmacy_flag: 1
   - clinical_risk_score: 3
   - discharge_disposition: Home
3. Review the output
4. Feel free to further test with different scenarios to validate the logic:
   - A low-risk patient being discharged home should receive routine discharge instructions
   - A patient with polypharmacy should trigger a medication reconciliation intervention
   - A Critical-risk patient with 3+ comorbidities should be flagged as Urgent priority with intensive transitional care

### 6. Publish the Decision

1. Click the **Validate** button and then **Publish** to make the decision available as a callable service
2. Choose a **destination:**
   - **CAS** - for batch execution against your full patient population at discharge
   - **MAS (Micro Analytic Service)** - for real-time, low-latency API calls during the discharge workflow - only one available here!
   - **Container** - for deployment in external systems (e.g., integration with Epic EHR)
3. Please make sure to give it a unique name
4. Once published, the decision is available as a REST API endpoint

---

## Using the SAS Intelligent Decisioning Copilot

The Copilot in SAS Intelligent Decisioning is a conversational assistant that can answer questions about the documentation for **SAS Intelligent Decisioning**, **SAS Container Runtime**, and **SAS Micro Analytic Service**. Use it to quickly find information about how these products work without leaving the application.

### What the Copilot Can Do

- **Answer documentation questions** about SAS Intelligent Decisioning features, concepts, and workflows
- **Explain SAS Micro Analytic Service (MAS)** deployment options, configuration, and API usage
- **Clarify SAS Container Runtime** setup, publishing, and management
- **Help you navigate** product capabilities by describing how specific features work
- **Provide guidance** on decision flow concepts, rule set configuration, and publishing options based on the official documentation

### Example Copilot Prompts

- *"How do I publish a decision to MAS?"*
- *"What is the difference between CAS and MAS as publishing destinations?"*
- *"How does SAS Container Runtime work for deploying decisions?"*
- *"What types of nodes can I add to a decision flow?"*
- *"How do I configure input and output variables for a decision?"*
- *"How do I integrate a decision with an external EHR system via REST API?"*

The Copilot is a useful reference tool for quickly getting answers about the platform's capabilities while you are building your decision flows.

---

## Decisions as Tools in Agentic Workflows

A published SAS Intelligent Decisioning decision is exposed as a **REST API endpoint**. This means it can be called as a **tool** by any AI agent - including large language model (LLM) agents that use tool-calling capabilities.

### How This Works

```
┌──────────────┐     ┌─────────────────────────┐     ┌──────────────────┐
│   AI Agent   │────>│  SAS Intelligent         │────>│  Care Plan       │
│  (e.g. LLM)  │     │  Decisioning API         │     │  Recommendation  │
│              │<────│  /decisions/readmitRisk   │<────│                  │
└──────────────┘     └─────────────────────────┘     └──────────────────┘
```

**Example scenario:** A clinical decision support agent (powered by an LLM) is assisting a care team during the discharge planning process. The agent can:

1. Pull the patient's clinical data from the EHR
2. **Call the SAS Intelligent Decisioning API** with the patient's features
3. Receive back: "High risk - home health referral + medication reconciliation, follow-up in 2 days"
4. Present this recommendation to the discharging physician with clinical rationale, enabling a more informed discharge plan

The decision becomes a **tool** in the agent's toolkit, just like a medication interaction checker or a lab result lookup. This bridges the gap between analytical models and clinical workflow.

### Why This Matters

- **Consistency:** Every discharge uses the same evidence-based decision logic - the rules and model scores are centralized, not dependent on individual clinician memory
- **Governance:** The decision is version-controlled and auditable in SAS Intelligent Decisioning, not buried in an LLM's system prompt
- **Separation of concerns:** Data scientists own the model, clinical leaders own the rules, and the AI agent just calls the endpoint
- **Real-time execution:** MAS endpoints return in milliseconds, fast enough for use during the discharge conversation

---

## Decisions as Agentic Workflows

Beyond being called as tools, SAS Intelligent Decisioning can itself orchestrate **agentic workflows** - multi-step processes that autonomously execute a chain of decisions and actions.

### How a Decision Becomes an Agent

An agentic decision flow goes beyond simple "input -> rules -> output." It can:

1. **Observe:** Receive a trigger event (e.g., a patient has been discharged and no follow-up appointment is scheduled within 48 hours)
2. **Reason:** Score the patient's readmission risk, check their care plan, review medication reconciliation status
3. **Decide:** Select the appropriate escalation action from the rule sets
4. **Act:** Trigger downstream actions - schedule a follow-up call, alert the care manager, send a patient portal message, create an EHR task
5. **Monitor:** Track whether the patient attends follow-up, refills medications, or shows signs of deterioration - and feed that outcome back into future decisions

### Example: Automated Post-Discharge Monitoring Agent

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Event       │     │  Decision Flow   │     │  Actions         │
│  Trigger     │────>│                  │────>│                  │
│              │     │  1. Score model   │     │  • Schedule call │
│  "Patient    │     │  2. Apply rules   │     │  • Alert care    │
│   discharged │     │  3. Assign care   │     │    manager       │
│   48 hrs ago,│     │     plan          │     │  • Send portal   │
│   no follow- │     │  4. Set priority  │     │    message       │
│   up booked" │     │  5. Select        │     │  • Create EHR    │
│              │     │     intervention  │     │    task           │
└─────────────┘     └──────────────────┘     └──────────────────┘
                              │
                              v
                    ┌──────────────────┐
                    │  Feedback Loop   │
                    │                  │
                    │  Did patient     │
                    │  attend follow-  │
                    │  up? Readmitted? │
                    │  Update model.   │
                    └──────────────────┘
```

This is **agentic** because the system autonomously:
- Detects the trigger condition (missed follow-up window)
- Makes decisions without human intervention
- Executes real-world clinical actions
- Learns from outcomes to improve future predictions

### Scaling Agentic Decisioning

In a production environment, this agentic workflow can process **hundreds of discharges per day** without manual intervention:

- **Batch mode:** Every morning, score all patients discharged in the past 24 hours, identify high-risk ones, trigger care coordination actions
- **Event-driven mode:** As soon as a patient is discharged, trigger the flow in real time from the EHR discharge event
- **Multi-decision chaining:** One decision flow calls another - e.g., the readmission risk decision calls a "medication reconciliation priority" decision which calls a "follow-up scheduling optimization" decision

SAS Intelligent Decisioning provides the orchestration layer that turns individual models and rules into **enterprise-scale autonomous clinical agents**.

---

## Summary

In this step you have:

1. **Created a decision flow** that combines your readmission model with clinical rules to produce actionable care plan recommendations at discharge
2. **Added an LLM call** that turns the decision into a warm, patient-friendly discharge summary
3. **Used the Copilot** to get answers about SAS Intelligent Decisioning, MAS, and Container Runtime documentation
4. **Published the decision** as a callable API endpoint
5. **Learned how decisions work as tools** for LLM-powered clinical decision support agents
6. **Explored agentic workflows** where decisions autonomously detect, reason, decide, and act in post-discharge monitoring

---

## Congratulations!

You have completed the full Data and AI Life Cycle for the MedCare patient readmission use case:

| Step | What You Did | SAS Technology |
|------|-------------|---------------|
| 1. Ask & Access | Understood the problem, generated synthetic data | SAS Data Maker |
| 2. Prepare | Loaded, profiled, and joined data into an ABT | SAS Viya Workbench |
| 3. Explore | Visually explored patterns with AI assistance | SAS Visual Analytics + Copilot |
| 4. Model | Built, compared, and fairness-tested models | SAS Model Studio + Copilot |
| 5. Deploy & Act | Operationalized with automated clinical decisions | SAS Intelligent Decisioning + Copilot |

If you have time remaining, explore another use case or dive deeper into any step. Talk to your bootcamp mentor for follow-up topics or to share feedback.

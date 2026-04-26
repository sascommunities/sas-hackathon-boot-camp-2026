# Step 5: Deploy & Act

In this final step you will use **SAS Intelligent Decisioning** to operationalize your urgency prediction model by embedding it in an automated service request triage decision flow. You will also explore its **Copilot** and learn how decisions can function as **tools in agentic workflows** - or become agentic workflows themselves.

---

## Prerequisites

Your champion model should be registered in **SAS Model Manager** from Step 4. SAS Intelligent Decisioning will pull the model directly from the Model Manager registry. If you did not register your own do not worry a default one is provided.

---

## What is SAS Intelligent Decisioning?

SAS Intelligent Decisioning is the platform for creating, managing, and executing business decisions that combine analytical models, business rules, and contextual logic into a single decision flow. Instead of just scoring a service request with a model, a decision flow can:

- Score the request's urgency probability
- Classify it into an urgency tier
- Route it to the appropriate department
- Allocate resources based on priority and availability
- Flag equity-sensitive cases for escalation
- Return a complete triage recommendation

This turns a model prediction into an **actionable triage decision**.

If you have any questions around SAS Intelligent Decisioning activate the SAS Viya copilot within the application via the icon in the top right hand corner next to your profile or ask one of the onsite SAS Mentors.

---

## Creating a Service Request Triage Decision

### 1. Open SAS Intelligent Decisioning

1. From the SAS Viya main menu, navigate to **SAS Intelligent Decisioning** (under *Build Decisions*)
2. Click **New Decision**
3. Name it: *Metro City Service Request Triage*
4. Leave the Description, Location and Workflow on default and click OK
5. Navigate to the *Variables* tab, click on the *Add variable* dropdown and either select *Custom variable* if you want to add them all yourself of *Decision* if you want to copy it from the template (this is faster). The manual steps are described in the below sub steps 1 & 2 while the copy is described in step 3:
    1. Define the **input variables** (these will be passed in when the decision is called):
        1. `request_id` (character)
        2. `request_type` (character)
        3. `department` (character)
        4. `location_district` (character)
        5. `response_time_hours` (decimal)
    2. Define the **output variables** (what the decision returns):
        1. `urgency_tier` (character) - the classified priority level
        2. `assigned_department` (character) - the department to handle the request
        3. `target_response_hours` (decimal) - the SLA target for this tier
        4. `resource_allocation` (character) - resource level to assign
        5. `escalation_flag` (character) - whether to escalate to management
        6. `reason` (character) - why this triage decision was made
        7. Now click OK to add all of them
    3. Copy the **variables** from a template decision:
        1. Click on the folder icon in the *Decision* input field
        2. Navigate to *SAS Content > SAS Hackathon Bootcamp 2026 > Use Case Public Sector* select *Metro City Service Request Triage* and click OK
        3. Click on the *Add all* icon in the middle of the dialogue to bring all the variables into your decision and then click the Add button


From here you can also always activate the SAS Viya Copilot via the icon in the top right hand corner to ask questions about SAS Intelligent Decisioning to deepen your understanding of the application.

### 2. Add the Model Node

1. In the decision flow canvas, add a **Model** node
2. Select your registered champion model from SAS Model Manager (*Metro City Urgency Prediction - Neural Network*)
3. Map the input variables to the model's expected features
    1. For the inputs the `request_id` should be mapped automatically and the remaining request features will align by name - review any that remain unmapped and connect them to the matching input variable
    2. For the outputs we are going to be clicking the *More* menu up top and select *Add missing variables* this will add all of the required output variables to our decision - if you copied the variables using the template they are already present - in the dialogue please make sure to deselect them from the Output as we will create our own custom outputs


### 3. Add Business Rules

After the model scores the request, add **Rule Set** nodes to determine the triage decision. For this make first sure that you have clicked the save icon of your decision and than we will be adding Rule Sets to our decision. There are two ways of doing this the easy way where you use the pre build rule sets by clicking on the three vertical dots on the model node and selecting *Add > Rule Set*, then in the dialogue navigate to *SAS Content > SAS Hackathon Bootcamp 2026 > Use Case Public Sector* and add the rule set as specified below. If you want to create them yourself you can go to the right hand side click on the *Objects* and drag & drop a Rule Set onto the previous node. This will open up a dialogue where you should name your decision correspondingly, please leave the location as the default (*My Folder*) - then add the variables from the decision you created and start building the Rule Sets as described below - the required variables are noted either as the columns or in the **Rule Conditions**.

We recommend you try to build at least one of these rule sets yourself to get an understanding of how it is done. If you have any questions around SAS Intelligent Decisioning activate the SAS Viya copilot within the application via the icon in the top right hand corner next to your profile or ask one of the onsite SAS Mentors.

For Rule Sets with multiple conditions please change the second condition from **IF** to **ELSE** - this will reduce the amount of typing you have to do and also improves runtime performance. If a condition contains a / then this indicates it applies to both values.

**Rule Set: Urgency Tier Classification**

| Rule Conditions | urgency_tier | target_response_hours |
|-----------|-------------|-----------------|
| P_is_urgent1 >= 0.90 | Critical | 4 |
| P_is_urgent1 >= 0.70 | High | 12 |
| P_is_urgent1 >= 0.40 | Medium | 36 |
| P_is_urgent1 < 0.40 | Low | 72 |

**Rule Set: Department Routing**

| urgency_tier | Rule Conditions | assigned_department | resource_allocation |
|--------------|-----------------------|--------------------|---------------------|
| Critical | request_type contains 'water' OR 'sewer' OR 'gas' | Public Works (Emergency) | Emergency crew + supervisor |
| Critical | request_type contains 'traffic' OR 'road' | Transportation (Emergency) | Emergency crew + supervisor |
| Critical | request_type contains 'building' OR 'structural' | Building & Safety (Emergency) | Inspector + emergency crew |
| High | Any | Original department (priority queue) | Senior staff + overtime authorized |
| Medium | Any | Original department (standard queue) | Standard staffing |
| Low | Any | Original department (scheduled queue) | Standard staffing |

**Rule Set: Escalation Rules**

| Rule Conditions | escalation_flag |
|-----------|-----------|
| urgency_tier = 'Critical' | Immediate: notify department head and city operations center |
| urgency_tier = 'High' AND response_time_hours > 24 | Escalate: flag for equity review |
| urgency_tier = 'Medium' AND response_time_hours > 48 | Escalate: reassign or add resources |
| urgency_tier = 'Low' AND response_time_hours > 72 | Auto-escalate to Medium tier |
| Otherwise | No escalation |

**Rule Set: Reason Assignment**

These rules capture the dominant driver behind the triage decision and populate the `reason` variable used downstream by the LLM:

| Rule Conditions | reason |
|-----------|----------|
| urgency_tier = 'Critical' AND (request_type contains 'water' OR 'gas') | Imminent safety hazard |
| urgency_tier = 'Critical' AND request_type contains 'traffic' | Public safety risk on roadways |
| urgency_tier = 'Critical' | High-probability urgent request requiring immediate dispatch |
| urgency_tier = 'High' AND response_time_hours > 24 | Delayed response in an equity-sensitive district |
| urgency_tier = 'High' | Elevated urgency requiring priority handling |
| urgency_tier = 'Medium' | Standard service request within normal SLA |
| Otherwise | Routine request scheduled for next available slot |

### 4. Adding an LLM to the Mix

Now right click the last node in the decision and add first a Rule Set (see blow) assignment and then on that node a Call LLM node. The Call LLM node will ask us to add missing variables again (like we did for the model node) - only add the prompt variable  (not as an input) and for the XXX we will map our reason variable that we have been using all along - then click the save icon.

Next we are going back to the Rule Set node and will assign our prompt variable the following value - you have to go into the editing mode via the pencil icon:

```
prompt = 'You are a professional Metro City 311 citizen communications specialist. Using the service request triage data below, write a warm, respectful, and clearly structured long-form citizen notification (2 to 4 paragraphs) that the person who filed the request can read to understand how their request has been triaged and what to expect next. Do not expose internal codes or jargon verbatim - translate them into plain, everyday language. Do not commit to outcomes beyond the target response time specified, and do not make political or judgmental statements about the city or about other requests. Request and triage context: Urgency tier: ' + urgency_tier + '. Assigned department: ' + assigned_department + '. Target response time (hours): ' + target_response_hours + '. Resource allocation: ' + resource_allocation + '. Escalation flag: ' + escalation_flag + '. Internal reason code: ' + reason + '. Structure your response as follows. First, open with a respectful thank-you for filing the request and confirm that Metro City has received it and takes it seriously. Second, explain in plain language what the ' + urgency_tier + ' urgency tier means for this request - translate it into everyday expectations rather than quoting the tier label verbatim. Third, describe which department (' + assigned_department + ') will handle the request and, in a single sentence, what level of resources (' + resource_allocation + ') has been assigned. Fourth, clearly communicate the target response window of ' + target_response_hours + ' hours, what the citizen can expect during that window, and how they will be notified of progress or completion. If the escalation flag ' + escalation_flag + ' indicates an immediate escalation, briefly note that senior city operations have also been notified. Translate the internal reason ' + reason + ' into a brief, plain-language note about why this particular triage decision was made. Fifth, close with clear next steps - how the citizen can check status, how to reach 311 with questions, and a clear instruction to call 911 for any life-threatening emergency. Tone: warm, accountable, professional, and never dismissive or bureaucratic. Length: 200 to 350 words. Write in the second person (you, your request).'
```

SAS provides the [https://github.com/sassoftware/sas-agentic-ai-accelerator](https://github.com/sassoftware/sas-agentic-ai-accelerator) which enables you to connect any LLM and do extensive prompt engineering & monitoring, but here we have a hard coded LLM (OpenAI GPT 5.4) available.

### 5. Test the Decision

1. Click **Test** in the toolbar
2. Enter sample values:
   - request_id: REQ000501
   - request_type: Water Main Break
   - department: Public Works
   - location_district: Downtown
   - response_time_hours: 48.0
3. Review the output
4. Feel free to further test with different scenarios to validate the logic:
   - A low-priority park maintenance request should fall into the Low tier with scheduled queue
   - A high-urgency request from a district with known equity gaps should trigger the equity-review escalation
   - A medium request that has been open for more than 48 hours should be flagged for reassignment

### 6. Publish the Decision

1. Click the **Validate** button and then **Publish** to make the decision available as a callable service
2. Choose a **destination:**
   - **CAS** - for batch execution against your full request backlog
   - **MAS (Micro Analytic Service)** - for real-time, low-latency API calls from the 311 system - only one available here!
   - **Container** - for deployment in external systems
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
- *"What are the publishing options for deploying a decision as a REST endpoint?"*

The Copilot is a useful reference tool for quickly getting answers about the platform's capabilities while you are building your decision flows.

---

## Decisions as Tools in Agentic Workflows

A published SAS Intelligent Decisioning decision is exposed as a **REST API endpoint**. This means it can be called as a **tool** by any AI agent - including large language model (LLM) agents that use tool-calling capabilities.

### How This Works

```
┌──────────────┐     ┌─────────────────────────┐     ┌──────────────────┐
│  311 Chatbot │────>│  SAS Intelligent         │────>│  Triage          │
│  Agent       │     │  Decisioning API         │     │  Recommendation  │
│  (LLM)       │<────│  /decisions/serviceTriage │<────│                  │
└──────────────┘     └─────────────────────────┘     └──────────────────┘
```

**Example scenario:** A citizen calls the 311 hotline or uses the Metro City chatbot to report a problem. The chatbot agent (powered by an LLM) can:

1. Collect the request details from the citizen through conversation
2. **Call the SAS Intelligent Decisioning API** with the request data
3. Receive back: "Critical - deploy emergency crew to Water Main Break, 4-hour SLA"
4. Inform the citizen of the expected response time and confirm the request has been prioritized
5. Simultaneously trigger the department dispatch workflow

The decision becomes a **tool** in the agent's toolkit, just like a search function or a database query. This bridges the gap between analytical models and citizen-facing conversational AI.

### Why This Matters

- **Consistency:** Every 311 interaction uses the same triage logic - the rules and model scores are centralized, not hardcoded into chatbot prompts
- **Governance:** The decision is version-controlled and auditable in SAS Intelligent Decisioning, not buried in an LLM's system prompt
- **Separation of concerns:** Data scientists own the model, city operations staff own the rules, and the chatbot just calls the endpoint
- **Real-time execution:** MAS endpoints return in milliseconds, fast enough for conversational use
- **Public accountability:** Because the decision logic is documented and versioned, it can be reviewed under public records requests

---

## Decisions as Agentic Workflows

Beyond being called as tools, SAS Intelligent Decisioning can itself orchestrate **agentic workflows** - multi-step processes that autonomously execute a chain of decisions and actions.

### How a Decision Becomes an Agent

An agentic decision flow goes beyond simple "input -> rules -> output." It can:

1. **Observe:** Receive a new service request or detect that an existing request has exceeded its SLA
2. **Reason:** Score the request's urgency, check the department's current workload, review the district's equity status
3. **Decide:** Select the optimal triage action, department routing, and resource allocation
4. **Act:** Trigger downstream actions - dispatch a crew, send a citizen notification, create a work order, update the dashboard
5. **Monitor:** Track whether the request is resolved within the SLA and feed that outcome back into future decisions

### Example: Automated Request Routing and Escalation Agent

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Event       │     │  Decision Flow   │     │  Actions         │
│  Trigger     │────>│                  │────>│                  │
│              │     │  1. Score model   │     │  - Dispatch crew  │
│  "New 311    │     │  2. Classify tier │     │  - Notify citizen │
│   request"   │     │  3. Route dept    │     │  - Create work    │
│   - or -     │     │  4. Allocate      │     │    order          │
│  "SLA breach │     │     resources     │     │  - Update         │
│   detected"  │     │  5. Check equity  │     │    dashboard      │
└─────────────┘     └──────────────────┘     └──────────────────┘
                              │
                              v
                    ┌──────────────────┐
                    │  Feedback Loop   │
                    │                  │
                    │  Was request     │
                    │  resolved within │
                    │  SLA? Update     │
                    │  model and rules.│
                    └──────────────────┘
```

This is **agentic** because the system autonomously:
- Detects the trigger condition (new request or SLA breach)
- Makes decisions without human intervention
- Executes real-world actions (dispatching, notifications)
- Learns from outcomes to improve future triage

### Scaling Agentic Decisioning

In a production environment, this agentic workflow can process **15,000 requests per month** without manual intervention:

- **Real-time mode:** As each new 311 request arrives, score and triage it instantly
- **Batch mode:** Every morning, re-score the open request backlog, identify SLA risks, and re-prioritize
- **Event-driven mode:** When a request's SLA clock is about to expire, trigger automatic escalation
- **Multi-decision chaining:** The triage decision calls a "resource optimization" decision which calls a "citizen notification" decision - creating a fully automated service delivery pipeline

SAS Intelligent Decisioning provides the orchestration layer that turns individual models and rules into **enterprise-scale autonomous agents** for municipal service delivery.

---

## Summary

In this step you have:

1. **Created a decision flow** that combines your urgency model with business rules to produce actionable triage recommendations
2. **Added an LLM call** that turns the decision into a warm, citizen-friendly notification message
3. **Used the Copilot** to get answers about SAS Intelligent Decisioning, MAS, and Container Runtime documentation
4. **Published the decision** as a callable API endpoint
5. **Learned how decisions work as tools** for the 311 chatbot agent
6. **Explored agentic workflows** where decisions autonomously detect, reason, decide, and act on service requests

---

## Congratulations!

You have completed the full Data and AI Life Cycle for the Metro City citizen service request prioritization use case:

| Step | What You Did | SAS Technology |
|------|-------------|---------------|
| 1. Ask & Access | Understood the problem, generated synthetic data | SAS Data Maker |
| 2. Prepare | Loaded, profiled, and joined data into an ABT | SAS Viya Workbench |
| 3. Explore | Visually explored patterns with AI assistance | SAS Visual Analytics + Copilot |
| 4. Model | Built, compared, and fairness-tested models across districts | SAS Model Studio + Copilot |
| 5. Deploy & Act | Operationalized with automated triage decisions | SAS Intelligent Decisioning + Copilot |

If you have time remaining, explore another use case or dive deeper into any step. Talk to your bootcamp mentor for follow-up topics or to share feedback.

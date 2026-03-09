# Cortex Code (CoCo) CLI Workshop
## Build AI Agents and Predictive Workflows with Snowflake

> **⚠️ BEFORE YOU START:** Replace all occurrences of `PARTICIPANT_XX` in this document with your assigned participant number (e.g., `PARTICIPANT_01`, `PARTICIPANT_02`, ... `PARTICIPANT_20`).

> **Duration:** 45-60 minutes (core workshop) + optional extension  
> **Tools:** Cortex Code CLI (VS Code), Snowflake Intelligence agent (SI agent), Snowflake ML  
> **Goal:** Build a reusable pattern to go from data → intelligence → predictive insight within 60 minutes

---

## Who This Workshop Is For

- Data platform architects exploring AI agent patterns
- Data science teams interested in integrating ML with AI agents
- Innovation teams evaluating Snowflake Cortex capabilities
- Technical teams running workshops or proof-of-concepts

---

## What You'll Build

Using **[Cortex Code CLI](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli)**, you will:

1. Generate realistic mock data (single-table for speed)
2. Create a **Snowflake Intelligence agent (SI agent)** over that data
3. Add a simple, explainable **predictive signal** (model-ready target)
4. Ask the agent questions to validate the experience
5. See how the **Data Science workflow** extends the pattern
6. Save your prompts for reuse

> **Note:** This workshop is scoped for 45-60 minutes. See the [Optional Extension](#optional-extension-industry-specific-customization) below for industry customization.

---

## Mental Model

- **Cortex Code CLI** → *Build the world* (data, schema, agents, business context)
- **SI Agent** → *Explain what's happening*
- **Data Science Workflow** → *Predict what will happen next*

This workshop focuses on the **handoff point** between Intelligence and ML.

---

## Prerequisites

Before starting, complete the following setup:

1. **Install Cortex Code CLI** - Follow the [installation guide](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli#install-cortex-code-cli)
2. **Create a Snowflake Personal Access Token (PAT)** or use an existing SQL connection credential — follow your organization's security policies
3. **Configure** `~/.snowflake/config.toml` with your connection
4. **Verify** you can launch Cortex Code CLI from VS Code terminal
5. **Verify Cortex Features** - Confirm your account has Cortex Agents and Cortex Search enabled. Check with your account team if unsure.

> **Important:** You must have a Snowflake user role with access to run SQL and create objects (e.g., SYSADMIN or equivalent). For full Cortex Code CLI setup and permission requirements (including [SNOWFLAKE.CORTEX_USER role](https://docs.snowflake.com/en/user-guide/snowflake-cortex/aisql#cortex-user-database-role)).

> **Security:** Never store secrets or tokens in your repository. This workshop uses demo/mock data. When transitioning to real customer data or production environments, follow your organization's data governance and security policies.

### Required Privileges

Your role needs the following privileges (or ask your Snowflake administrator to grant them):

- `CREATE DATABASE`, `CREATE SCHEMA`, `CREATE STAGE`
- `CREATE TABLE`, `CREATE VIEW`, `CREATE SEMANTIC VIEW ON SCHEMA`
- `CREATE CORTEX SEARCH SERVICE ON SCHEMA`
- `CREATE AGENT ON SCHEMA`
- `USAGE ON WAREHOUSE`

### Successful Setup

When setup is complete, launching Cortex Code CLI should show:
- The CLI banner
- Your connection name
- A warehouse
- A working directory

```bash
cortex -c <YOUR_CONNECTION>
```

**Resources:**
- [Cortex Code CLI Overview](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code)

---

## Workshop Flow

| Time | Step |
|---|---|
| 0–5 min | Overview & setup verification |
| 5–12 min | Context prompt setup |
| 12–22 min | Generate data + agent artifacts |
| 22–30 min | Add predictive signal |
| 30–40 min | Load data and create objects |
| 40–50 min | Validate with agent questions |
| 50–60 min | Data Science workflow walkthrough |

---

## Step 1 — Generate Demo Artifacts (7–10 minutes)

In this step, you use Cortex Code CLI to generate:
- Demo data (CSV)
- SQL scripts
- SQL script for semantic view
- Search service + agent definitions

**Important:** This step generates files locally only. No data is loaded into Snowflake yet.

### 1A — Start Cortex Code CLI

```bash
cortex -c <YOUR_CONNECTION>
```

### 1B — Run the Generation Prompt

Paste the following prompt into Cortex Code CLI:

```text
You are a Snowflake Cortex Code CLI assistant helping prepare a demo.

DO NOT DEVIATE FROM THESE REQUIREMENTS.

Use the following exact object names:
- Database: WORKSHOP_DB
- Schema: PARTICIPANT_XX_SCHEMA
- Stage: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE
- Table: TRANSACTIONS
- File format: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.CSVFORMAT
- Semantic view: DEMO_SEMANTIC_VIEW
- Search service: TEXT_SEARCH
- Warehouse: PARTICIPANT_XX_WH
- SI agent: DEMO_AGENT

## TASK ORDER (follow exactly):

### Step 1: Create and run a Python script
Write generate_data.py that generates transactions.csv with:
- 100 rows (plus header)
- Columns: TRANSACTION_ID, CUSTOMER_ID, CUSTOMER_NAME, TRANSACTION_DATE, TRANSACTION_TYPE, AMOUNT, MERCHANT, CHANNEL, LOCATION, IS_FLAGGED, NOTES_TEXT
- Use the csv module and random for realistic data
- Set `random.seed(42)` at the top for deterministic output
- NOTES_TEXT must contain realistic investigation/customer-service notes (randomly selected from ~20 pre-defined templates)
- Context: Financial services focusing on fraud detection and customer experience

Run the script immediately after writing it to generate the CSV.

### Step 2: Create these SQL files (do NOT execute):
- 01_create_table.sql - CREATE TABLE for TRANSACTIONS with appropriate column types
- 02_load_from_stage.sql - COPY INTO from @WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE/transactions.csv using FILE_FORMAT = WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.CSVFORMAT
- 03_create_search_service.sql - CREATE CORTEX SEARCH SERVICE on NOTES_TEXT column
- 04_create_agent.sql - CREATE AGENT using FROM SPECIFICATION with this exact YAML structure:
      models:
        orchestration: auto
      tools:
        - tool_spec:
            type: cortex_analyst_text_to_sql
            name: Analyst1
            description: Analyzes transaction data using natural language
        - tool_spec:
            type: cortex_search
            name: Search1
            description: Searches transaction notes for investigation details
      tool_resources:
        Analyst1:
          semantic_view: "WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DEMO_SEMANTIC_VIEW"
        Search1:
          name: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TEXT_SEARCH
          max_results: 5
          title_column: TRANSACTION_ID
- 05_grants.sql - Standard grants for the objects

### Step 3: Create 03b_create_semantic_view.sql
Create a SQL file that creates a SEMANTIC VIEW named `DEMO_SEMANTIC_VIEW` in WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.

The semantic view should:
- Reference the base table WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TRANSACTIONS
- Define practical dimensions (date, channel, location, type, merchant, flagged)
- Define practical measures (count of transactions, total amount, avg amount, flagged rate)
Review code agaist the example in the docs: https://docs.snowflake.com/en/user-guide/views-semantic/example 




Use CREATE OR REPLACE SEMANTIC VIEW syntax.

### Step 4: Print PUT commands
Print PUT commands to upload to @WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE:
- transactions.csv

## RULES:
- Do NOT execute any SQL
- Do NOT invent additional object names
- Do NOT reference local file paths in SQL
- Output must be deterministic and reusable

Now generate the artifacts.
```

### What You Should See

- Local files created in your working directory:
    • transactions.csv
    • 01_create_table.sql
    • 02_load_from_stage.sql
    • 03_create_search_service.sql
    • 03b_create_semantic_view.sql
    • 04_create_agent.sql
    • 05_grants.sql
- PUT commands printed at the end

### 1C — Validate Generated Files (Optional)

Before proceeding, you can validate your generated artifacts:

List the files I just created and confirm the semantic view SQL has valid syntax.

**Do not run any SQL yet.**

### Success Criteria

- `transactions.csv` exists with 100 data rows + header
- `03b_create_semantic_view.sql` exists
- SQL scripts `01` through `05` exist in working directory
- PUT commands were printed

---

## Step 2 — Add Predictive Signal (5 minutes)

This step creates the **handoff point** from the SI agent to the Data Science workflow by adding a model-ready target variable (the predictive signal).

### Run the Predictive Signal Prompt

```text
Extend the existing dataset with a single business-relevant target variable.

IMPORTANT:
- Fixed table: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TRANSACTIONS (do NOT create a new table)
- Add column: IS_FRAUD BOOLEAN
- Output Snowflake SQL only (no prose), notebook-ready
- Write the SQL to a local file named: 06_add_target.sql
- SQL must include ONLY:
  1) ALTER TABLE ... ADD COLUMN IF NOT EXISTS
  2) UPDATE ... SET IS_FRAUD = CASE ... END

Logic requirements:
- Use simple, explainable rules based on existing columns (and optionally NOTES_TEXT)
- No randomness. No model training. No external calls.
- Include a short comment block at the top explaining the rules in plain English.

Now generate 06_add_target.sql.
```

### What You Should See

- A new SQL file: `06_add_target.sql`
- SQL contains ALTER + UPDATE statements only
- Simple, explainable logic

**Do not run any SQL yet.**

### Success Criteria

- `06_add_target.sql` exists
- File contains ALTER TABLE and UPDATE statements
- Logic is explainable in plain English

---

## Step 3 — Load Data + Create Objects (8–10 minutes)

Now switch from generation to execution.

### 3A — Bootstrap Required Objects

> **Note:** Use a role with CREATE privileges (e.g., SYSADMIN or a specially provisioned admin role). If you lack permission, coordinate with your Snowflake administrator.

Run the following SQL to create the required infrastructure:

```text
Run the following SQL in my current connection.

-- Use a role with CREATE privileges on database/schema/stage
-- Examples: SYSADMIN, or a custom admin role provisioned by your org
-- Avoid ACCOUNTADMIN unless required by your organization's policies
USE ROLE SYSADMIN;

-- Database + schema
CREATE DATABASE IF NOT EXISTS WORKSHOP_DB;
CREATE SCHEMA IF NOT EXISTS WORKSHOP_DB.PARTICIPANT_XX_SCHEMA;

-- Stage
CREATE STAGE IF NOT EXISTS WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

-- CSV file format
CREATE FILE FORMAT IF NOT EXISTS WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.CSVFORMAT
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  EMPTY_FIELD_AS_NULL = TRUE
  FIELD_OPTIONALLY_ENCLOSED_BY = '"';

-- Warehouse (if needed)
CREATE WAREHOUSE IF NOT EXISTS PARTICIPANT_XX_WH
  WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;

-- Cortex grants (requires appropriate admin privileges)
-- If these fail, contact your Snowflake administrator
GRANT CREATE AGENT ON SCHEMA WORKSHOP_DB.PARTICIPANT_XX_SCHEMA TO ROLE SYSADMIN;
GRANT CREATE SEMANTIC VIEW ON SCHEMA WORKSHOP_DB.PARTICIPANT_XX_SCHEMA TO ROLE SYSADMIN;
GRANT CREATE CORTEX SEARCH SERVICE ON SCHEMA WORKSHOP_DB.PARTICIPANT_XX_SCHEMA TO ROLE SYSADMIN;

SHOW GRANTS ON SCHEMA WORKSHOP_DB.PARTICIPANT_XX_SCHEMA;
```

> **Troubleshooting:** If any CREATE AGENT, CREATE CORTEX SEARCH SERVICE, or grant commands fail, confirm your role has the required privileges or contact your Snowflake administrator. Avoid running these steps under ACCOUNTADMIN unless required by your organization.

### 3B — Upload Files to Stage

> **Note:** PUT commands require SnowSQL or Cortex Code CLI. They do not work in Snowsight SQL worksheets.  
> **Alternative:** Use the Snowsight UI: **Data > Databases > WORKSHOP_DB > PARTICIPANT_XX_SCHEMA > Stages > DATA_STAGE > Upload Files**

```text
Upload the local file from my current working directory to the stage @WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE using PUT.
Overwrite if it already exists:

- transactions.csv
```

### 3C — Execute SQL Scripts

```text
In my current connection, read and execute the contents of these local SQL files from my current working directory, in this exact order:

1) 01_create_table.sql
2) 02_load_from_stage.sql
3) 03_create_search_service.sql
4) 03b_create_semantic_view.sql
5) 04_create_agent.sql
6) 05_grants.sql
7) 06_add_target.sql
```

**After each SQL execution, confirm:**
- Tables are created successfully (no errors)
- Stage has the uploaded files (verify with `LIST @WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.DATA_STAGE`)
- Search service and agent report "created" with no errors
- COPY INTO reports rows loaded (should be ~100 rows)

> **Troubleshooting:** If CREATE AGENT or CREATE CORTEX SEARCH SERVICE fails:
> - Verify your account has Cortex features enabled
> - Confirm your role has `CREATE AGENT` and `CREATE CORTEX SEARCH SERVICE` grants on the schema
> - Contact your Snowflake administrator if issues persist

### Success Criteria

- Table exists: `WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TRANSACTIONS` (~100 rows)
- Search service exists: `WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TEXT_SEARCH`
- SI agent exists: `DEMO_AGENT` (visible in Snowsight under Cortex AI > Agents)
- Target column exists: `IS_FRAUD` with >1% positive examples (non-degenerate signal)

---

## Step 4 — Interact with the SI Agent (5–10 minutes)

### 4A — Open the SI Agent

1. In Snowsight, navigate to **Cortex AI > Agents**
2. Find and open `DEMO_AGENT`

### 4B — Ask Questions

Enter these questions in the agent chat panel:

```text
What are the top 5 items we should investigate or prioritize next — and why?
(Use the best identifier column and list the key drivers.)
```

```text
What patterns best explain outcomes related to detecting fraudulent transactions?
(Call out the strongest signals and any notable segments.)
```

```text
Give me a 30-second executive summary for a business stakeholder, include 1-2 recommended actions.
```

#### What "Good" Looks Like

A successful SI agent response should:
- Reference specific TRANSACTION_IDs (e.g., "TRANSACTION_ID 42, 87, 91...")
- Cite evidence from NOTES_TEXT (e.g., "Investigation notes mention 'unusual pattern'...")
- Provide quantified insights (e.g., "23% of flagged transactions occurred on weekends...")
- Give actionable recommendations (e.g., "Prioritize review of high-amount weekend transactions")

> **Expected direction:** The agent should return specific transactions with drivers tied back to fraud indicators from the data.

### 4C — Optimize Agent Responses (Optional)

In Cortex Code CLI, use the built-in agent optimization skill to refine tone and structure (no data or model changes required):

```text
Refine the agent's response behavior to make answers clearer, more concise, and executive-friendly. Use the agent optimize skill.

```

> **How to invoke:** In Cortex Code CLI, you can also type `/agent-optimization` to invoke the skill directly.

Then refresh the agent in Snowsight and re-ask a question to see improved responses.

### Success Criteria

- Agent returns specific identifiers (e.g., TRANSACTION_ID)
- Uses both structured analysis and NOTES_TEXT evidence
- Provides clear explanations and actionable next steps

---

## Step 5 — Data Science Workflow (10 minutes)

This step demonstrates the handoff from **Intelligence** (explain what's happening) to **Data Science** (predict what may happen next).

> **Compute Environment:** Cortex Code will generate Python code. You can:
> - Run it in a local Jupyter notebook (requires scikit-learn, pandas)
> - Copy it into a Snowflake Notebook (Snowsight > Projects > Notebooks)
> - Execute via Snowpark Container Services for full platform integration

### 5A — Validate the Target Column

```text
In my current connection, verify that the target column exists and has usable signal.

Context:
- Table: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TRANSACTIONS
- Target column: IS_FRAUD (BOOLEAN)

Generate AND EXECUTE SQL that:
1) Shows total row count
2) Shows counts and percentages by IS_FRAUD
3) Returns 5 example rows where IS_FRAUD = TRUE, including NOTES_TEXT

Then summarize the findings in 3 short bullets focused on business relevance.
```

### Success Criteria

- IS_FRAUD column exists
- Both TRUE and FALSE values present
- Positive rate >1% (non-degenerate signal)

### 5B — Create Feature View

```text
Generate AND EXECUTE Snowflake SQL to create a model-ready feature view for ML training.

Context:
- Base table: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.TRANSACTIONS
- Target column: IS_FRAUD (BOOLEAN)

Requirements:
- Create a feature view named: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.FEATURES_V
- Include IS_FRAUD as the label column
- Keep all features simple and explainable

Include features such as:
- Numeric features (AMOUNT)
- Time-derived features (day_of_week, hour_of_day)
- Text flags from NOTES_TEXT using ILIKE patterns

After creation, run SELECT * FROM the view LIMIT 5.
```

### Success Criteria

- View `WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.FEATURES_V` exists
- Contains IS_FRAUD label column
- Contains engineered features

### 5C — Train and Evaluate Model

> **Execution Options:**
> - **Local:** Run in your Python environment with scikit-learn and pandas installed
> - **Snowflake Notebook:** Copy generated code into Snowsight > Projects > Notebooks
> - **Snowpark Container Services:** For full platform integration

```text
Create and execute a notebook-based ML workflow.

Context:
- Feature view: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.FEATURES_V
- Label column: IS_FRAUD (BOOLEAN)

Requirements:
- Use 80/20 train/test split (deterministic)
- Train simple classifier (logistic regression or gradient boosting)
- Use scikit-learn and pandas

Generate notebook content for:
1) Model training
2) Evaluation metrics (confusion matrix, precision/recall)
3) Top 10 risk predictions with explanations
4) Feature importance analysis

Execute the notebook and summarize results.
```

### 5D — Data Science Questions

```text
Using the trained model outputs, answer:

1) What are the top 10 transactions most likely to be fraudulent?
   - Include transaction identifiers
   - Include predicted probability
   - Include feature-based explanations

2) Which features contribute most strongly to fraud risk?
   - Summarize in plain business language

3) What are the top 2 actions the team should take next?
```

### Success Criteria

- Model trained and evaluated
- Can rank records by predicted risk
- Can explain why records matter
- Can recommend next actions

---

## Step 6 — Save Your Work (3 minutes)

Before finishing:
- Save your Cortex Code prompts
- Save your generated SQL files
- Note the industry + target variable you used

This becomes a **reusable template** for future projects.

---

## Summary

You've completed the workshop if you have:
- A working SI agent
- A business-relevant target column
- A saved Cortex Code prompt you can reuse
- Understanding of the Data Science handoff pattern

**Pattern learned:** Data → Intelligence → Data Science → Action

---

## Cleanup (Optional)

To remove workshop objects from your account:

```sql
-- Run these commands to clean up workshop objects
DROP DATABASE IF EXISTS WORKSHOP_DB;
DROP WAREHOUSE IF EXISTS PARTICIPANT_XX_WH;
```

---

---

# Optional Extension: Industry-Specific Customization

This extension allows you to tailor the workshop to a specific industry or use case. Complete the core workshop first.

> **Quickstart (advanced users):**  
> ```bash
> git clone <repo-url> && cd cortex-code-cli-workshop && cortex -c <YOUR_CONNECTION>
> ```
> Then paste the customization prompt below.

---

## When to Use This Extension

- Customize for a specific industry vertical
- Go deeper into features, models, and predictions
- Prepare for a workshop or proof-of-concept

---

## Extension Bootstrap

Before running the extension, create the CUSTOM schema:

```text
Run the following SQL in my current connection.

USE ROLE SYSADMIN;

-- Create schema for extension
CREATE SCHEMA IF NOT EXISTS WORKSHOP_DB.CUSTOM;

-- Stage for custom data
CREATE STAGE IF NOT EXISTS WORKSHOP_DB.CUSTOM.DATA_STAGE
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

-- Grants
GRANT CREATE AGENT ON SCHEMA WORKSHOP_DB.CUSTOM TO ROLE SYSADMIN;
GRANT CREATE SEMANTIC VIEW ON SCHEMA WORKSHOP_DB.CUSTOM TO ROLE SYSADMIN;
GRANT CREATE CORTEX SEARCH SERVICE ON SCHEMA WORKSHOP_DB.CUSTOM TO ROLE SYSADMIN;
```

---

## Customization Workflow

### Phase 1: Define Your Context

Replace the placeholders below with your specific context:

```text
=====================
CUSTOMIZATION INPUTS
=====================
- Organization/Industry: {{INDUSTRY}}
- Primary business priority: {{PRIORITY_1}}
- Secondary business priority: {{PRIORITY_2}}
- Key stakeholder role: {{STAKEHOLDER_ROLE}}
=====================
END INPUTS
=====================
```

### Phase 2: Generate Custom Artifacts

Use this prompt with your customization inputs:

```text
=====================
CUSTOMIZATION INPUTS
=====================
- Organization/Industry: {{INDUSTRY}}
- Primary business priority: {{PRIORITY_1}}
- Secondary business priority: {{PRIORITY_2}}
- Key stakeholder role: {{STAKEHOLDER_ROLE}}
=====================
END INPUTS
=====================

------------------------------------------------------------

You are a Snowflake Cortex Code CLI assistant helping prepare a customized demo.

DO NOT DEVIATE FROM THESE REQUIREMENTS.

Use the following exact object names:
- Database: WORKSHOP_DB
- Schema: CUSTOM
- Stage: WORKSHOP_DB.CUSTOM.DATA_STAGE
- Table: CUSTOM_TRANSACTIONS
- File format: WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.CSVFORMAT (reuse from core workshop)
- Semantic view: CUSTOM_SEMANTIC_VIEW
- Search service: CUSTOM_TEXT_SEARCH
- Warehouse: PARTICIPANT_XX_WH
- SI agent: CUSTOM_AGENT

## TASK ORDER (follow exactly):

### Step 1: Create and run a Python script
Write generate_custom_data.py that generates custom_transactions.csv with:

- Exactly 100 rows (plus header)
- Use the csv module and random for realistic data
- Set `random.seed(42)` at the top for deterministic output
- Single-table dataset aligned to the CUSTOMIZATION INPUTS above
- Columns (exact order):
  TRANSACTION_ID,
  CUSTOMER_ID,
  CUSTOMER_NAME,
  TRANSACTION_DATE,
  TRANSACTION_TYPE,
  AMOUNT,
  MERCHANT,
  CHANNEL,
  LOCATION,
  IS_FLAGGED,
  NOTES_TEXT

NOTES_TEXT requirements:
- Must contain realistic freeform notes aligned to the industry + priorities
- Randomly selected from ~20 pre-defined templates
- Templates should reference the stakeholder role when relevant
- Include a mix of investigation and operational notes

Run the script immediately after writing it to generate the CSV.

### Step 2: Create SQL files (do NOT execute)
- 01_create_table.sql - CREATE TABLE for CUSTOM_TRANSACTIONS
- 02_load_from_stage.sql - COPY INTO using FILE_FORMAT = WORKSHOP_DB.PARTICIPANT_XX_SCHEMA.CSVFORMAT
- 03_create_search_service.sql - CREATE CORTEX SEARCH SERVICE CUSTOM_TEXT_SEARCH ON WORKSHOP_DB.CUSTOM.CUSTOM_TRANSACTIONS(NOTES_TEXT)
    WAREHOUSE = PARTICIPANT_XX_WH
    TARGET_LAG = '1 hour';
- 03b_create_semantic_view.sql - CREATE OR REPLACE SEMANTIC VIEW CUSTOM_SEMANTIC_VIEW
    referencing WORKSHOP_DB.CUSTOM.CUSTOM_TRANSACTIONS with dimensions and measures. Review code agaist the example in the docs: https://docs.snowflake.com/en/user-guide/views-semantic/example 

- 04_create_agent.sql - CREATE AGENT with tool_resources referencing:
    semantic_view: "WORKSHOP_DB.CUSTOM.CUSTOM_SEMANTIC_VIEW"
- 05_grants.sql - Standard grants for table, semantic view, search service, and agent

### Step 3: Create 03b_create_semantic_view.sql
Create a SQL file that creates a SEMANTIC VIEW named `CUSTOM_SEMANTIC_VIEW` in WORKSHOP_DB.CUSTOM.

The semantic view should:
- Reference the base table WORKSHOP_DB.CUSTOM.CUSTOM_TRANSACTIONS
- Define practical dimensions (date, channel, location, type, merchant, flagged)
- Define practical measures (count of transactions, total amount, avg amount, flagged rate)
- Include verified queries phrased in industry language tied to PRIORITY_1 and PRIORITY_2

Use CREATE OR REPLACE SEMANTIC VIEW syntax.

### Step 4: Print PUT commands
Print PUT commands to upload to @WORKSHOP_DB.CUSTOM.DATA_STAGE:
- custom_transactions.csv

## RULES:
- Do NOT execute any SQL
- Do NOT invent additional object names
- Output must be deterministic and reusable

Now generate the artifacts.
```

### Phase 3: Add Custom Target Signal

```text
Extend the existing dataset with a single, explainable target signal.

DO NOT DEVIATE FROM THESE REQUIREMENTS:
- Fixed table name: WORKSHOP_DB.CUSTOM.CUSTOM_TRANSACTIONS
- Add exactly one new column named: TARGET_SIGNAL
- Column type: NUMBER(5,2) with values between 0.00 and 1.00 (inclusive)
- Output Snowflake SQL only, notebook-ready
- Write the SQL to: 06_add_target_signal.sql

SQL OUTPUT MUST INCLUDE ONLY:
1) ALTER TABLE ... ADD COLUMN IF NOT EXISTS TARGET_SIGNAL NUMBER(5,2);
2) UPDATE ... SET TARGET_SIGNAL = CASE ... END;

How to build TARGET_SIGNAL:
- Represents likelihood/intensity of the primary business priority
- Use simple, explainable CASE/WHEN rules
- No randomness, no model training

Include a comment block explaining the rules in plain English.

Now generate 06_add_target_signal.sql.
```

### Phase 4: Execute and Validate

Follow the same execution steps as the core workshop:
1. Upload `custom_transactions.csv` to `@WORKSHOP_DB.CUSTOM.DATA_STAGE`
2. Execute SQL scripts in this order:
   1) 01_create_table.sql
   2) 02_load_from_stage.sql
   3) 03_create_search_service.sql
   4) 03b_create_semantic_view.sql
   5) 04_create_agent.sql
   6) 05_grants.sql
   7) 06_add_target_signal.sql
3. Test the agent with industry-specific questions

---

## Industry Examples

| Industry | Example Target | Business Meaning |
|----------|---------------|------------------|
| Banking/FSI | IS_FRAUD | Likelihood of fraudulent activity |
| Retail/CPG | LIKELY_TO_ENGAGE | Probability of customer response |
| Manufacturing | FAILURE_RISK | Risk of equipment failure |
| Healthcare | READMISSION_RISK | Likelihood of patient readmission |
| Life Sciences | TRIAL_ELIGIBLE | Patient trial eligibility |
| SaaS/Tech | EXPANSION_SIGNAL | Account expansion likelihood |

---

## Feature Store Integration (Advanced)

For more advanced use cases, create a Feature Store-backed feature view:

```text
Generate AND EXECUTE Snowflake SQL and Python to create a model-ready feature view for ML training.

Context:
- Base table: WORKSHOP_DB.CUSTOM.CUSTOM_TRANSACTIONS
- Target column: TARGET_SIGNAL (NUMBER between 0.00 and 1.00)

Requirements:
- Create a feature store: WORKSHOP_DB.CUSTOM
- Create an entity based on the identifier column
- Create a feature view: WORKSHOP_DB.CUSTOM.CUSTOM_FEATURES_V
- Include TARGET_SIGNAL as the label column
- Keep features simple and explainable

Include features such as:
- Numeric features relevant to the industry
- Time-derived features from timestamps
- Categorical groupings meaningful to the business
- 2–3 text-derived features from NOTES_TEXT using ILIKE patterns

Constraints:
- Create a view-based feature view, NOT a dynamic table
- Register in feature store with the entity
- No complex NLP or embeddings
- After creation, run SELECT * LIMIT 5
```

---

## What We Did Not Do (By Design)

To keep this workshop focused:
- Single-table dataset (no multi-table joins)
- Simple rule-based features (no embeddings)
- Lightweight model (no hyperparameter tuning)
- No production deployment or MLOps patterns

---

## Extending Further

Consider augmenting this pattern by:
- Adding real data instead of mock data
- Introducing multiple tables
- Training models using Snowflake ML
- Connecting to Feature Store
- Packaging into a repeatable workflow

**Key takeaway:** The same Cortex Code pattern scales from demo → workshop → production.

---

## Success Criteria

You've completed the workshop if you have:
- A working SI agent
- A business-relevant target column
- A saved Cortex Code prompt you can reuse

**You're done!**---

> **For detailed advanced scenarios**, consider creating a separate `ADVANCED.md` with extended Feature Store integration, multi-table joins, and production deployment patterns.
# coco-cli-workshop

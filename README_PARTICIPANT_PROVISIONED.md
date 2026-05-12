# Cortex Code (CoCo) CLI Workshop
## Build AI Agents and Predictive Workflows with Snowflake

> **⚠️ BEFORE YOU START — Pre-Provisioned Edition:** Your workshop admin has created a **role**, **database**, **schema**, and **warehouse** for you and granted your role the privileges needed below. You do NOT create databases, schemas, or warehouses yourself.
>
> Ask your admin for these four values, then run the SET / USE block below at the start of every new SQL session in this workshop:
>
> - `<YOUR_ROLE>` — your assigned workshop role
> - `<YOUR_DB>` — your assigned database
> - `<YOUR_SCHEMA>` — your assigned schema
> - `<YOUR_WH>` — your assigned warehouse
>
> ```sql
> SET YOUR_ROLE   = '<YOUR_ROLE>';
> SET YOUR_DB     = '<YOUR_DB>';
> SET YOUR_SCHEMA = '<YOUR_SCHEMA>';
> SET YOUR_WH     = '<YOUR_WH>';
>
> USE ROLE      IDENTIFIER($YOUR_ROLE);
> USE WAREHOUSE IDENTIFIER($YOUR_WH);
> USE SCHEMA    IDENTIFIER($YOUR_DB || '.' || $YOUR_SCHEMA);
> ```
>
> Throughout this guide, every SQL example uses the placeholders `<YOUR_DB>` / `<YOUR_SCHEMA>` / `<YOUR_WH>` / `<YOUR_ROLE>`. Substitute them with your assigned values when copying SQL into prompts.

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
3. Ask the agent questions to validate the experience
4. Save your prompts for reuse

> **Note:** This workshop is scoped for 45-60 minutes. See the [Optional Extension](#optional-extension-industry-specific-customization) below for industry customization.

---

## Mental Model

- **Cortex Code CLI** → *Build the world* (data, schema, agents, business context)
- **SI Agent** → *Explain what's happening*

This workshop focuses on rapidly going from data to a working SI agent.

---

## Prerequisites

Before starting, complete the following setup:

1. **Install Cortex Code CLI** - Follow the [installation guide](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli#install-cortex-code-cli)
2. **Create a Snowflake Personal Access Token (PAT)** or use an existing SQL connection credential — follow your organization's security policies
3. **Configure** `~/.snowflake/config.toml` with your connection
4. **Verify** you can launch Cortex Code CLI from VS Code terminal
5. **Verify Cortex Features** - Confirm your account has Cortex Agents and Cortex Search enabled. Check with your account team if unsure.
6. **Confirm your assigned values** — make sure your admin has given you `<YOUR_ROLE>`, `<YOUR_DB>`, `<YOUR_SCHEMA>`, and `<YOUR_WH>`.

> **Important:** This workshop uses a **pre-provisioned** model. Your admin has already created the database, schema, warehouse, and role and granted the privileges below. You will not (and cannot) run CREATE DATABASE / CREATE SCHEMA / CREATE WAREHOUSE.

> **Security:** Never store secrets or tokens in your repository. This workshop uses demo/mock data. When transitioning to real customer data or production environments, follow your organization's data governance and security policies.

### Required Privileges (already granted to your assigned role)

You do NOT need any account-level CREATE privileges. Your assigned role (`<YOUR_ROLE>`) should already have, on your assigned schema (`<YOUR_DB>.<YOUR_SCHEMA>`):

- `CREATE TABLE`, `CREATE VIEW`, `CREATE STAGE`, `CREATE FILE FORMAT`
- `CREATE SEMANTIC VIEW`
- `CREATE CORTEX SEARCH SERVICE`
- `CREATE AGENT`

Plus:

- `USAGE` on `<YOUR_DB>` and `<YOUR_SCHEMA>`
- `USAGE` on `<YOUR_WH>`
- The `SNOWFLAKE.CORTEX_USER` database role granted to `<YOUR_ROLE>` (see [Cortex User database role](https://docs.snowflake.com/en/user-guide/snowflake-cortex/aisql#cortex-user-database-role))

If any step in this workshop fails with an "insufficient privileges" error, contact your workshop admin and reference the exact missing grant — do NOT switch to ACCOUNTADMIN or SYSADMIN.

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
| 22–35 min | Load data and create objects |
| 35–45 min | Validate with agent questions |

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

Paste the following prompt into Cortex Code CLI. **Before pasting, replace `<YOUR_DB>`, `<YOUR_SCHEMA>`, and `<YOUR_WH>` with the exact values your admin gave you** — Snowflake DDL like `CREATE STAGE` and `CREATE AGENT` cannot be parameterized with session variables, so the literal names must appear in the generated SQL.

```text
You are a Snowflake Cortex Code CLI assistant helping prepare a demo.

DO NOT DEVIATE FROM THESE REQUIREMENTS.

Use the following exact object names (the four placeholders below are pre-provisioned by my admin and must NOT be created by you):
- Database: <YOUR_DB>            (already exists, do NOT CREATE)
- Schema: <YOUR_SCHEMA>          (already exists, do NOT CREATE)
- Warehouse: <YOUR_WH>           (already exists, do NOT CREATE)
- Stage: <YOUR_DB>.<YOUR_SCHEMA>.DATA_STAGE
- Table: TRANSACTIONS  (fully qualified: <YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS)
- File format: <YOUR_DB>.<YOUR_SCHEMA>.CSVFORMAT
- Semantic view: <YOUR_DB>.<YOUR_SCHEMA>.DEMO_SEMANTIC_VIEW
- Search service: <YOUR_DB>.<YOUR_SCHEMA>.TEXT_SEARCH
- SI agent: <YOUR_DB>.<YOUR_SCHEMA>.DEMO_AGENT

HARD RULES:
- Do NOT emit CREATE DATABASE, CREATE SCHEMA, or CREATE WAREHOUSE statements anywhere.
- All generated SQL must reference objects with the fully-qualified names above.

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
- 01_create_table.sql - CREATE TABLE for <YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS with appropriate column types
- 02_load_from_stage.sql - COPY INTO <YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS FROM @<YOUR_DB>.<YOUR_SCHEMA>.DATA_STAGE/transactions.csv using FILE_FORMAT = <YOUR_DB>.<YOUR_SCHEMA>.CSVFORMAT
- 03_create_search_service.sql - CREATE CORTEX SEARCH SERVICE <YOUR_DB>.<YOUR_SCHEMA>.TEXT_SEARCH on the NOTES_TEXT column of <YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS
- 04_create_agent.sql - CREATE AGENT <YOUR_DB>.<YOUR_SCHEMA>.DEMO_AGENT using FROM SPECIFICATION with this exact YAML structure:
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
          semantic_view: "<YOUR_DB>.<YOUR_SCHEMA>.DEMO_SEMANTIC_VIEW"
        Search1:
          name: <YOUR_DB>.<YOUR_SCHEMA>.TEXT_SEARCH
          max_results: 5
          title_column: TRANSACTION_ID

(Do NOT generate 05_grants.sql — grants are managed by my admin.)

### Step 3: Create 03b_create_semantic_view.sql
Create a SQL file that creates a SEMANTIC VIEW named `<YOUR_DB>.<YOUR_SCHEMA>.DEMO_SEMANTIC_VIEW`.

The semantic view should:
- Reference the base table <YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS
- Define practical dimensions (date, channel, location, type, merchant, flagged)
- Define practical measures (count of transactions, total amount, avg amount, flagged rate)
Review code against the example in the docs: https://docs.snowflake.com/en/user-guide/views-semantic/example

Use CREATE OR REPLACE SEMANTIC VIEW syntax.

### Step 4: Print PUT commands
Print PUT commands to upload to @<YOUR_DB>.<YOUR_SCHEMA>.DATA_STAGE:
- transactions.csv

## RULES:
- Do NOT execute any SQL
- Do NOT invent additional object names
- Do NOT reference local file paths in SQL
- Do NOT emit CREATE DATABASE / CREATE SCHEMA / CREATE WAREHOUSE / GRANT
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
- PUT commands printed at the end

> **Note:** `05_grants.sql` is intentionally NOT generated in the pre-provisioned model — your admin manages grants for you.

### 1C — Validate Generated Files (Optional)

Before proceeding, you can validate your generated artifacts:

List the files I just created and confirm the semantic view SQL has valid syntax.

**Do not run any SQL yet.**

### Success Criteria

- `transactions.csv` exists with 100 data rows + header
- `03b_create_semantic_view.sql` exists
- SQL scripts `01` through `04` exist in working directory
- PUT commands were printed

---

## Step 2 — Load Data + Create Objects (8–10 minutes)

Now switch from generation to execution.

### 2A — Set Session Context

> **Note:** Your admin has already created your role, database, schema, warehouse, and granted you the required in-schema privileges. Run only the following to point your session at them and create the stage + file format inside your assigned schema.

```text
Run the following SQL in my current connection.

SET YOUR_ROLE   = '<YOUR_ROLE>';
SET YOUR_DB     = '<YOUR_DB>';
SET YOUR_SCHEMA = '<YOUR_SCHEMA>';
SET YOUR_WH     = '<YOUR_WH>';

USE ROLE      IDENTIFIER($YOUR_ROLE);
USE WAREHOUSE IDENTIFIER($YOUR_WH);
USE SCHEMA    IDENTIFIER($YOUR_DB || '.' || $YOUR_SCHEMA);

-- Create the stage and file format inside YOUR pre-provisioned schema
CREATE STAGE IF NOT EXISTS DATA_STAGE
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

CREATE FILE FORMAT IF NOT EXISTS CSVFORMAT
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  EMPTY_FIELD_AS_NULL = TRUE
  FIELD_OPTIONALLY_ENCLOSED_BY = '"';

-- Verify your session context and grants
SELECT CURRENT_ROLE(), CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();
SHOW GRANTS TO ROLE IDENTIFIER($YOUR_ROLE);
```

> **Troubleshooting:** If `CREATE STAGE` or `CREATE FILE FORMAT` fails with an insufficient-privilege error, contact your workshop admin — your assigned role is missing the corresponding in-schema CREATE grant. Do NOT switch to SYSADMIN or ACCOUNTADMIN.

### 2B — Upload Files to Stage

> **Note:** PUT commands require SnowSQL or Cortex Code CLI. They do not work in Snowsight SQL worksheets.  
> **Alternative:** Use the Snowsight UI: **Data > Databases > `<YOUR_DB>` > `<YOUR_SCHEMA>` > Stages > DATA_STAGE > Upload Files**

```text
Upload the local file from my current working directory to the stage @<YOUR_DB>.<YOUR_SCHEMA>.DATA_STAGE using PUT.
Overwrite if it already exists:

- transactions.csv
```

### 2C — Execute SQL Scripts

```text
In my current connection, read and execute the contents of these local SQL files from my current working directory, in this exact order:

1) 01_create_table.sql
2) 02_load_from_stage.sql
3) 03_create_search_service.sql
4) 03b_create_semantic_view.sql
5) 04_create_agent.sql
```

**After each SQL execution, confirm:**
- Tables are created successfully (no errors)
- Stage has the uploaded files (verify with `LIST @<YOUR_DB>.<YOUR_SCHEMA>.DATA_STAGE`)
- Search service and agent report "created" with no errors
- COPY INTO reports rows loaded (should be ~100 rows)

> **Troubleshooting:** If CREATE AGENT or CREATE CORTEX SEARCH SERVICE fails:
> - Verify your account has Cortex features enabled
> - Confirm your assigned role has `CREATE AGENT` and `CREATE CORTEX SEARCH SERVICE` grants on `<YOUR_DB>.<YOUR_SCHEMA>`
> - Confirm your role has `SNOWFLAKE.CORTEX_USER`
> - Contact your workshop admin if issues persist — do NOT switch to ACCOUNTADMIN

### Success Criteria

- Table exists: `<YOUR_DB>.<YOUR_SCHEMA>.TRANSACTIONS` (~100 rows)
- Search service exists: `<YOUR_DB>.<YOUR_SCHEMA>.TEXT_SEARCH`
- SI agent exists: `<YOUR_DB>.<YOUR_SCHEMA>.DEMO_AGENT` (visible in Snowsight under Cortex AI > Agents)

---

## Step 3 — Interact with the SI Agent (5–10 minutes)

### 3A — Open the SI Agent

1. In Snowsight, navigate to **Cortex AI > Agents**
2. Find and open `DEMO_AGENT` under your assigned schema (`<YOUR_DB>.<YOUR_SCHEMA>`)

### 3B — Ask Questions

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

### 3C — Optimize Agent Responses (Optional)

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

## Step 4 — Save Your Work (3 minutes)

Before finishing:
- Save your Cortex Code prompts
- Save your generated SQL files
- Note the industry/use case you used

This becomes a **reusable template** for future projects.

---

## Summary

You've completed the workshop if you have:
- A working SI agent
- A saved Cortex Code prompt you can reuse

**Pattern learned:** Data → Intelligence → Action

---

## Cleanup (Optional)

Drop only the objects YOU created inside your assigned schema. Your admin will drop the role, database, schema, and warehouse for you.

```sql
USE ROLE   IDENTIFIER($YOUR_ROLE);
USE SCHEMA IDENTIFIER($YOUR_DB || '.' || $YOUR_SCHEMA);

DROP AGENT                 IF EXISTS DEMO_AGENT;
DROP CORTEX SEARCH SERVICE IF EXISTS TEXT_SEARCH;
DROP SEMANTIC VIEW         IF EXISTS DEMO_SEMANTIC_VIEW;
DROP TABLE                 IF EXISTS TRANSACTIONS;
DROP STAGE                 IF EXISTS DATA_STAGE;
DROP FILE FORMAT           IF EXISTS CSVFORMAT;
```

> **Do not run** `DROP DATABASE` or `DROP WAREHOUSE` — those are not yours to drop in this workshop model.

---

## Appendix — Admin Pre-Provisioning Recipe

This appendix is for the **workshop admin** running the event. The model is:

- **One shared role** for all participants (e.g. `WORKSHOP_ROLE`)
- **One shared database** for all participants (e.g. `WORKSHOP_DB`)
- **One shared warehouse** for all participants (e.g. `WORKSHOP_WH`)
- **One schema per participant** inside the shared database (e.g. `PARTICIPANT_01_SCHEMA`, `PARTICIPANT_02_SCHEMA`, …)

The shared role is granted CREATE privileges on each participant's schema. Snowflake's session/role model means participants can only meaningfully act inside the schema they `USE`, so this is sufficient isolation for a workshop. (For stricter isolation, use a per-participant role; see "Stricter isolation" note below.)

After the recipe runs, hand each participant their four values: the shared role name, the shared database name, **their** schema name, and the shared warehouse name.

> Run as a role with `CREATE ROLE`, `CREATE DATABASE`, `CREATE WAREHOUSE`, and the ability to grant `SNOWFLAKE.CORTEX_USER` (typically a custom admin role; avoid ACCOUNTADMIN unless your org policy requires it).

### Step 1 — Create the shared role, database, and warehouse (run once)

```sql
SET WS_ROLE = 'WORKSHOP_ROLE';
SET WS_DB   = 'WORKSHOP_DB';
SET WS_WH   = 'WORKSHOP_WH';

-- Shared role
CREATE ROLE IF NOT EXISTS IDENTIFIER($WS_ROLE);
GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE IDENTIFIER($WS_ROLE);

-- Shared database
CREATE DATABASE IF NOT EXISTS IDENTIFIER($WS_DB);
GRANT USAGE ON DATABASE IDENTIFIER($WS_DB) TO ROLE IDENTIFIER($WS_ROLE);

-- Shared warehouse
CREATE WAREHOUSE IF NOT EXISTS IDENTIFIER($WS_WH)
  WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;
GRANT USAGE, OPERATE ON WAREHOUSE IDENTIFIER($WS_WH) TO ROLE IDENTIFIER($WS_ROLE);
```

### Step 2 — For each participant: create their schema, grant CREATE on it, attach the role to their user

Re-run this block once per participant, substituting `P_SCHEMA` and `P_USER`:

```sql
SET WS_ROLE  = 'WORKSHOP_ROLE';
SET WS_DB    = 'WORKSHOP_DB';
SET P_SCHEMA = 'PARTICIPANT_01_SCHEMA';     -- change per participant
SET P_USER   = '<participant_snowflake_user>'; -- change per participant
SET FQ_SCHEMA = $WS_DB || '.' || $P_SCHEMA;

-- Per-participant schema
CREATE SCHEMA IF NOT EXISTS IDENTIFIER($FQ_SCHEMA);
GRANT USAGE ON SCHEMA IDENTIFIER($FQ_SCHEMA) TO ROLE IDENTIFIER($WS_ROLE);

-- In-schema CREATE grants needed by the workshop
GRANT CREATE TABLE,
      CREATE VIEW,
      CREATE STAGE,
      CREATE FILE FORMAT,
      CREATE SEMANTIC VIEW,
      CREATE CORTEX SEARCH SERVICE,
      CREATE AGENT
  ON SCHEMA IDENTIFIER($FQ_SCHEMA)
  TO ROLE IDENTIFIER($WS_ROLE);

-- Attach the shared role to this participant's user
GRANT ROLE IDENTIFIER($WS_ROLE) TO USER IDENTIFIER($P_USER);

-- Verify
SHOW GRANTS TO ROLE IDENTIFIER($WS_ROLE);
```

### Hand-off

After the recipe runs, give each participant these four values:

| Placeholder       | Example value               |
|-------------------|-----------------------------|
| `<YOUR_ROLE>`     | `WORKSHOP_ROLE`             |
| `<YOUR_DB>`       | `WORKSHOP_DB`               |
| `<YOUR_SCHEMA>`   | `PARTICIPANT_01_SCHEMA`     |
| `<YOUR_WH>`       | `WORKSHOP_WH`               |

Only `<YOUR_SCHEMA>` differs per participant. The other three values are the same for everyone.

> **Stricter isolation (optional):** If you need each participant unable to even see other participants' schemas, replace the single `WORKSHOP_ROLE` with one role per participant (`PARTICIPANT_01_ROLE`, …) and grant CREATE on only that participant's schema. The shared warehouse and database can remain shared.

### Admin Cleanup

```sql
SET WS_DB    = 'WORKSHOP_DB';
SET P_SCHEMA = 'PARTICIPANT_01_SCHEMA';     -- change per participant

DROP SCHEMA IF EXISTS IDENTIFIER($WS_DB || '.' || $P_SCHEMA) CASCADE;
```

After all participant schemas are dropped, you can also drop the shared objects:

```sql
DROP WAREHOUSE IF EXISTS WORKSHOP_WH;
DROP DATABASE  IF EXISTS WORKSHOP_DB;
DROP ROLE      IF EXISTS WORKSHOP_ROLE;
```

---

# CoCo CLI Workshop - Quick Start

Build AI Agents and Predictive Workflows with Snowflake in 45-60 minutes.

## What You'll Build
1. Generate mock transaction data (100 rows)
2. Create a Snowflake Intelligence (SI) agent
3. Add a predictive signal (IS_FRAUD target)
4. Validate with agent questions
5. Run a Data Science workflow (train/evaluate model)

## Prerequisites
- Cortex Code CLI installed
- Snowflake connection configured in `~/.snowflake/config.toml`
- Role with CREATE privileges (DATABASE, SCHEMA, STAGE, TABLE, AGENT, etc.)

## Quick Flow
| Step | Action |
|------|--------|
| 1 | Generate artifacts: `cortex -c <YOUR_CONNECTION>` + paste generation prompt |
| 2 | Add predictive signal: run prompt for `06_add_target.sql` |
| 3 | Bootstrap objects: create DB, schema, stage, warehouse |
| 4 | Upload CSV + execute SQL scripts (01-06) |
| 5 | Test SI agent in Snowsight > Cortex AI > Agents |
| 6 | Run Data Science workflow (feature view + model training) |

## Key Objects
- **Database:** `WORKSHOP_DB` | **Schema:** `PARTICIPANT_XX_SCHEMA`
- **Table:** `TRANSACTIONS` | **Agent:** `DEMO_AGENT`
- **Semantic View:** `DEMO_SEMANTIC_VIEW` | **Search Service:** `TEXT_SEARCH`

## Pattern Learned
**Data → Intelligence → Data Science → Action**

See `README_PARTICIPANT.md` for full instructions.

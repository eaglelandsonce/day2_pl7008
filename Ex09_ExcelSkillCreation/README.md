# Excel-to-Skill Example Lab

## Learning objective
Generate a reusable skill from an Excel workbook and business requirements, and execute it to write processed results into extra workbook tabs.

## Files
- `Projects_Input.xlsx`: 12 realistic raw project rows
- `Requirements.md`: business rules and acceptance tests
- `Agent_Prompt.md`: prompt for an agent to **generate** the skill from the first two files
- `SKILL.md`: example completed skill with YAML frontmatter
- `scripts/process_projects.py`: working deterministic Python implementation
- `Projects_Processed.xlsx`: tested workbook including new tabs

## Steps
1. Give learners only `Projects_Input.xlsx`, `Requirements.md`, and `Agent_Prompt.md`.
2. Ask them to prompt their AI agent to create SKILL.md and a supporting Python script.
3. Compare their generated files against example `SKILL.md` and `scripts/process_projects.py`.
4. Run with Python in an environment with artifact-tool available:
   `python scripts/process_projects.py Projects_Input.xlsx Projects_Processed.xlsx --as-of 2026-11-16`
5. Inspect the source Projects tab, processed results tab, and summary tab.
6. Discuss handling invalid statuses, missing budgets, dates, and owners, plus whether the agent actually executed the code.

## Copilot Studio note
Agent instructions and skill instructions are separate. The agent prompt describes **what to build**; SKILL.md describes **how to execute** the repeatable processing. Python execution must be supported by the target agent/runtime; simply uploading a .py file to Copilot Studio does not guarantee the agent can run Python directly. If Python execution is unsupported, connect an approved execution service/action or use a supported workflow.

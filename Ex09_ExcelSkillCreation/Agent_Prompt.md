# Agent prompt — generate the skill from the requirements

You are a Skill Builder Agent. I am attaching `Projects_Input.xlsx` and `Requirements.md`.

1. Inspect the Excel workbook and read all business requirements.
2. Create a reusable skill named `project-excel-processor`, including a YAML-frontmatter `SKILL.md` that explains triggers, inputs, procedures, outputs, safety rules, and validation checks.
3. Implement the processing steps in a Python script stored at `scripts/process_projects.py`. Use Excel workbook operations to preserve the original `Projects` worksheet unchanged and add new `Processed_Results` and `Processing_Summary` worksheets in a new output workbook.
4. Do not merely describe the processing; run it on the example workbook. Capture/flag invalid data without deleting rows.
5. Run the acceptance tests described in `Requirements.md`, correct any errors, and provide the complete skill package and the processed workbook.
6. Keep the skill reusable for other Excel files with the same schema. Separate deterministic Python execution from the agent's natural-language orchestration.

When running an already-created skill, use:
"Use the project-excel-processor skill to process the attached Projects_Input.xlsx according to Requirements.md. Return the workbook with the new results tabs and summarize exceptions."

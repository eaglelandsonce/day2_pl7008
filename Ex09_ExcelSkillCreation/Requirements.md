# Business Requirements — Project Spreadsheet Processor

## Inputs
- An .xlsx workbook containing the worksheet `Projects`.
- Required source columns: Project ID, Project Name, Department, Status, Budget, Actual Cost, Due Date, Owner.
- This requirements file. Use `2026-11-16` as the example evaluation date so results are reproducible. For production runs allow an explicit as-of date.

## Transformation rules
1. Copy every original project row into `Processed_Results` with standardized values. Never alter the `Projects` worksheet.
2. Normalize departments: `IT`, `technology`, `Information Technology`, `tech` -> `IT`; `HR`, `human resources`, `Human Resources` -> `Human Resources`; `Sales` -> `Sales`. Unknown department: retain value and flag.
3. Normalize status case, whitespace, hyphens: `in progress`, `in-progress` -> `In Progress`; `completed`, `complete`, `done` -> `Completed`; `not started` -> `Not Started`; `at risk` -> `At Risk`; `blocked` -> `Blocked`. Unrecognized statuses -> `Needs Review` with an issue flag.
4. Parse Budget and Actual Cost as USD numbers, accepting numbers and strings with dollar signs, commas and spaces. Blank or invalid values are flagged, not treated as valid zero. Allow valid numeric zero.
5. Parse dates in `YYYY-MM-DD`, `YYYY/MM/DD`, and `MM/DD/YYYY` formats. Invalid dates are flagged.
6. `Variance = Budget - Actual Cost` when both numbers exist; a negative variance means over budget. `Budget Flag = Over Budget` if actual cost exceeds budget, `Within Budget` otherwise, and `Missing Data` when calculation is impossible.
7. Mark `Overdue = Yes` if due date precedes the evaluation date AND normalized status is not `Completed`; otherwise `No`. If date invalid/missing, set `Unknown`.
8. Flag missing Project ID, Project Name or Owner. Preserve rows even when invalid; join data quality issues with `; `; if no issues write `OK`.
9. Add `Processed_Results` tab to a **new copy** of the input workbook, with original source tab intact; if run again replace the prior results tab rather than creating duplicates.
10. Include `Processing_Summary` tab with total projects, projects with issues, over-budget projects, overdue projects, and processing date.
11. Write output as `Projects_Processed.xlsx`, with clean headers, currency number formats, and visually distinct issue/over-budget indicators.

## Acceptance tests
- P-002 status is Completed and variance is -2400.
- P-004 is overdue as of 2026-11-16.
- P-008 has status Needs Review with invalid-status issue.
- P-010 has Missing Data as budget flag.
- P-011 has an invalid-date issue.
- P-012 flags a missing owner.
- Original 12 project rows remain intact.

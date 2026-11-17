---
name: customer-data-standardizer
description: Clean customer workbooks that follow the Customer_Data_Dirty.xlsx schema by standardizing names, emails, North American phone numbers, signup dates, countries, order totals, and statuses, while preserving customer IDs and escalating ambiguous values instead of guessing.
---

# Customer Data Standardizer

## When to use this skill
Use this skill when an Excel or CSV file contains these columns, or clear equivalents:

- Customer ID
- Name
- Email
- Phone
- Signup Date
- Country
- Order Total
- Status

Use it when values have inconsistent capitalization, abbreviations, punctuation, date formats, currency text, incomplete emails, shortened statuses, or missing values. The intended output matches the conventions demonstrated by `Customer_Data_Clean.xlsx`.

Do not use this skill when columns have different business meanings, phone numbers are not intended to follow the demonstrated `+1-555-####` pattern, currencies are mixed without a currency field, or country/status mappings are not approved.

## Output contract

1. Preserve the source row order and keep one output row per input row.
2. Preserve the eight column headers and their order.
3. Write cleaned data to a new sheet or new file. Never overwrite the raw source unless explicitly instructed.
4. Name the cleaned sheet `Customers_Cleaned` and, when creating an Excel table, name it `CustomersCleaned`.
5. Freeze row 1.
6. Store Signup Date as a real Excel date displayed as `yyyy-mm-dd`.
7. Store Order Total as a numeric value displayed with two decimal places (`#,##0.00`).
8. Maintain an exception log for ambiguous or unresolvable values.

# Column-by-column cleaning rules

## 1. Customer ID

### Rule
- Trim leading and trailing whitespace.
- Preserve the identifier exactly after trimming, including its letter prefix and leading zeros.
- Do not renumber, infer, or regenerate IDs.
- Require one nonblank, unique ID per row.

### Examples
- ` C001 ` -> `C001`
- `C020` -> `C020` (unchanged)

### Ambiguous cases
- Blank ID: flag the row; do not invent an ID.
- Duplicate ID: retain both raw rows in an exception output and require review.
- Case-only variation such as `c001`: normalize only if the approved ID pattern is explicitly `C` plus three digits; otherwise flag.

## 2. Name

### Rule
- Trim outer whitespace and collapse repeated internal spaces.
- Convert all-uppercase or all-lowercase names to title case.
- Preserve already well-formed title case.
- Do not remove punctuation or diacritics from legitimate names.

### Demonstrated changes
- `alice johnson` -> `Alice Johnson`
- `BRIAN SMITH` -> `Brian Smith`
- `carla gomez` -> `Carla Gomez`
- `david lee` -> `David Lee`

### Ambiguous cases
- Names containing particles, apostrophes, hyphens, suffixes, or intentional capitalization, such as `de la Cruz`, `O'Neil`, `Smith-Jones`, `McDonald`, or `III`, should not be forced through naive title case. Preserve when plausible or flag for review.
- Blank name: flag; do not infer from the email unless the workflow explicitly permits it.

## 3. Email

### Rule
- Trim whitespace.
- Convert the entire address to lowercase.
- Require exactly one `@`, a nonblank local part, and a domain containing at least one dot.
- When a malformed or missing email can be resolved unambiguously from the demonstrated pattern `firstname.lastname@example.com` and the row's Name, repair it.
- Otherwise leave the original value in the raw data, place the cleaned value blank, and log an exception.

### Demonstrated changes
- `Alice.Johnson@Example.com` -> `alice.johnson@example.com`
- `emma.brown@example` -> `emma.brown@example.com`
- `frank.miller.example.com` -> `frank.miller@example.com`
- blank email for Jack Davis -> `jack.davis@example.com`

### Ambiguous cases
Only infer an email when all of the following are true:
1. The workbook clearly uses one consistent domain, here `example.com`.
2. Neighboring valid rows consistently use `firstname.lastname@domain`.
3. The name maps to one obvious local part.
4. The inferred email does not duplicate another email.

If any condition fails, do not guess. Flag values with multiple `@` signs, spaces, conflicting domains, ambiguous multi-part names, duplicate addresses, or unsupported characters.

## 4. Phone

### Rule
- Trim whitespace.
- Remove presentation punctuation such as spaces, periods, and parentheses for parsing.
- For values consistent with the demonstrated synthetic North American pattern, normalize to `+1-555-####`.
- If the country code is omitted but the value contains the `555` prefix plus four digits, add `+1-`.
- Preserve the final four digits exactly.

### Demonstrated changes
- `555-0102` -> `+1-555-0102`
- `1 555 0103` -> `+1-555-0103`
- `(555)0104` -> `+1-555-0104`
- `+15550105` -> `+1-555-0105`
- `555.0108` -> `+1-555-0108`

### Ambiguous cases
- Do not apply this pattern to arbitrary real-world international numbers.
- Blank, extension-bearing, wrong-length, non-555, or conflicting country-code values must be flagged.
- If the business later accepts international phones, use an approved parser such as Google's libphonenumber and output E.164, rather than this dataset-specific display pattern.

## 5. Signup Date

### Rule
- Parse recognized date strings using the demonstrated row context.
- Convert to a real date value.
- Display uniformly as `yyyy-mm-dd`.
- Reject impossible dates.

### Demonstrated changes
- `01/15/2024` -> `2024-01-15`
- `Feb 3, 2024` -> `2024-02-03`
- `19-Feb-2024` -> `2024-02-19`
- `2024/03/25` -> `2024-03-25`
- `20.04.2024` -> `2024-04-20`
- `5/9/24` -> `2024-05-09`

### Ambiguous cases
- Numeric dates such as `03/04/2024` are ambiguous between month/day and day/month. Infer only when the workbook's locale or another unambiguous date establishes the convention; otherwise flag.
- Two-digit years must use an approved century window. In this dataset, `24` is interpreted as `2024`; do not generalize without a rule.
- Blank or invalid dates remain blank in cleaned output and are logged.

## 6. Country

### Rule
- Trim whitespace.
- Normalize capitalization.
- Map approved aliases to canonical country names.

### Approved mappings demonstrated by the reference file
- `USA`, `U.S.`, `us` -> `United States`
- `UK`, `U.K.` -> `United Kingdom`
- `canada` -> `Canada`
- `Mexico`, `Canada`, `United States`, `United Kingdom` -> unchanged canonical values

### Ambiguous cases
- Do not expand ambiguous abbreviations such as `Congo`, `Korea`, `VI`, or `GE` without an approved mapping or ISO code context.
- Unknown countries and misspellings not covered by the mapping must be flagged rather than silently replaced.

## 7. Order Total

### Rule
- Trim whitespace.
- Remove approved currency decorations: leading `$`, thousands commas, surrounding spaces, and trailing `USD`.
- Parse the result as a decimal number.
- Store it as a number, not text.
- Display with two decimal places using `#,##0.00`.
- Preserve the numeric amount exactly; do not round beyond the display format unless more than two source decimals require an approved rounding rule.

### Demonstrated changes
- `$1,250.00` -> numeric `1250.00`
- `980.5` -> numeric `980.50`
- `$2,100.75` -> numeric `2100.75`
- `1,895.90 USD` -> numeric `1895.90`
- `$2300` -> numeric `2300.00`
- `450.00` -> numeric `450.00` (type/format standardized)

### Ambiguous cases
- Parentheses, minus signs, or `CR` may indicate negative values; do not reinterpret without a business rule.
- Non-USD symbols/codes, mixed decimal separators, blank values, or nonnumeric text must be flagged.
- If totals must be nonnegative, reject negative values during validation.

## 8. Status

### Rule
- Trim whitespace and compare case-insensitively.
- Map approved full values and abbreviations to exactly `Active` or `Inactive`.

### Approved mappings demonstrated by the reference file
- `active`, `Active`, `A`, `act` -> `Active`
- `INACTIVE`, `Inactive`, `Inact` -> `Inactive`

### Ambiguous cases
- Blank, conflicting, or unknown labels such as `Pending`, `Disabled`, `I`, or numeric codes must be flagged unless an approved mapping exists.
- Do not infer status from Order Total, Signup Date, or activity in other fields.

# Exact differences observed in the reference pair

The comparison is keyed by Customer ID. Values described as type/format changes may look similar in text but differ because the clean file uses a consistent native type or display format.

| Customer ID | Column | Dirty value | Clean value | Explaining rule |
|---|---|---|---|---|
| C001 | Name | `alice johnson` | `Alice Johnson` | Title-case name |
| C001 | Email | `Alice.Johnson@Example.com` | `alice.johnson@example.com` | Lowercase email |
| C001 | Signup Date | `01/15/2024` | `2024-01-15` | Parse and standardize date |
| C001 | Country | `USA` | `United States` | Country alias mapping |
| C001 | Order Total | `$1,250.00` | `1250.00` | Remove currency text and store numeric |
| C001 | Status | `active` | `Active` | Canonical status capitalization |
| C002 | Name | `BRIAN SMITH` | `Brian Smith` | Title-case name |
| C002 | Phone | `555-0102` | `+1-555-0102` | Add country code and canonical punctuation |
| C002 | Order Total | `980.5` | `980.50` | Numeric type and two-decimal display |
| C003 | Name | `carla gomez` | `Carla Gomez` | Title-case name |
| C003 | Phone | `1 555 0103` | `+1-555-0103` | Canonical phone punctuation |
| C003 | Signup Date | `Feb 3, 2024` | `2024-02-03` | Parse and standardize date |
| C003 | Order Total | `450.00` | `450.00` | Store as numeric with two-decimal display |
| C003 | Status | `INACTIVE` | `Inactive` | Canonical status capitalization |
| C004 | Name | `david lee` | `David Lee` | Title-case name |
| C004 | Phone | `(555)0104` | `+1-555-0104` | Add country code and canonical punctuation |
| C004 | Country | `canada` | `Canada` | Canonical country capitalization |
| C004 | Order Total | `$2,100.75` | `2100.75` | Remove currency text and store numeric |
| C005 | Email | `emma.brown@example` | `emma.brown@example.com` | Complete unambiguous domain |
| C005 | Phone | `+15550105` | `+1-555-0105` | Canonical phone punctuation |
| C005 | Signup Date | `19-Feb-2024` | `2024-02-19` | Parse and standardize date |
| C005 | Country | `UK` | `United Kingdom` | Country alias mapping |
| C005 | Order Total | `760.00` | `760.00` | Store as numeric with two-decimal display |
| C006 | Email | `frank.miller.example.com` | `frank.miller@example.com` | Repair missing `@` using consistent pattern |
| C006 | Country | `us` | `United States` | Country alias mapping |
| C006 | Order Total | `310.25` | `310.25` | Store as numeric with two-decimal display |
| C006 | Status | `Inact` | `Inactive` | Approved status abbreviation |
| C007 | Order Total | `1580` | `1580.00` | Store as numeric with two-decimal display |
| C008 | Phone | `555.0108` | `+1-555-0108` | Add country code and canonical punctuation |
| C008 | Order Total | `890.00` | `890.00` | Store as numeric with two-decimal display |
| C008 | Status | `A` | `Active` | Approved status abbreviation |
| C009 | Signup Date | `2024/03/25` | `2024-03-25` | Parse and standardize date |
| C009 | Order Total | `1340.40` | `1340.40` | Store as numeric with two-decimal display |
| C010 | Email | blank | `jack.davis@example.com` | Infer only from unambiguous name/domain pattern |
| C010 | Country | `U.K.` | `United Kingdom` | Country alias mapping |
| C010 | Order Total | `275.00` | `275.00` | Store as numeric with two-decimal display |
| C011 | Order Total | `1,895.90 USD` | `1895.90` | Remove currency code/separator and store numeric |
| C012 | Signup Date | `20.04.2024` | `2024-04-20` | Parse day-month-year and standardize |
| C012 | Order Total | `640.00` | `640.00` | Store as numeric with two-decimal display |
| C013 | Order Total | `1120.00` | `1120.00` | Store as numeric with two-decimal display |
| C014 | Signup Date | `5/9/24` | `2024-05-09` | Parse established month/day convention and expand year |
| C014 | Order Total | `505.60` | `505.60` | Store as numeric with two-decimal display |
| C015 | Order Total | `$2300` | `2300.00` | Remove currency symbol and apply numeric display |
| C016 | Order Total | `415.00` | `415.00` | Store as numeric with two-decimal display |
| C016 | Status | `act` | `Active` | Approved status abbreviation |
| C017 | Order Total | `970.25` | `970.25` | Store as numeric with two-decimal display |
| C018 | Order Total | `1685.00` | `1685.00` | Store as numeric with two-decimal display |
| C019 | Order Total | `720.80` | `720.80` | Store as numeric with two-decimal display |
| C020 | Order Total | `1430.00` | `1430.00` | Store as numeric with two-decimal display |

No value changes were observed in Customer ID. Rows not listed for a particular column already matched the canonical value, although the clean workbook applies consistent column-level date and number formats.

# Recommended cleaning procedure

1. **Load safely**
   - Read the raw sheet without modifying it.
   - Confirm the expected eight headers.
   - Preserve all original values in a raw snapshot.

2. **Profile the file**
   - Count rows, blanks, duplicates, and distinct country/status labels.
   - Detect the dominant email domain and email naming pattern.
   - Detect whether phone values match the dataset-specific North American pattern.
   - Determine the workbook's date convention from unambiguous dates before parsing ambiguous numeric dates.

3. **Clean in dependency order**
   - Clean Customer ID and Name first.
   - Use the cleaned Name only as supporting evidence for an email repair.
   - Clean Email, Phone, Signup Date, Country, Order Total, and Status independently.
   - Never derive Status or financial values from unrelated columns.

4. **Log every transformation**
   - Record row/Customer ID, column, raw value, cleaned value, rule ID, and confidence (`deterministic`, `inferred`, or `needs review`).
   - Mark the repaired emails for C005, C006, and C010-style cases as `inferred`, even when the reference pattern is strong.

5. **Write the cleaned output**
   - Use the required sheet/table names and formats in the Output contract.
   - Keep exception rows visible in a separate `Cleaning_Exceptions` sheet or companion CSV.

6. **Validate before delivery**
   - Run the checklist below.
   - Do not claim success when unresolved exceptions remain; report their count and affected IDs.

# Validation checklist

## Structure
- [ ] Exactly the expected eight columns are present in the expected order.
- [ ] Output row count equals input row count.
- [ ] Source row order is preserved.
- [ ] Raw source remains unchanged.
- [ ] Cleaned sheet is named `Customers_Cleaned`.

## Customer ID
- [ ] No blank IDs.
- [ ] IDs are unique.
- [ ] Leading zeros are preserved.

## Name
- [ ] No unintended leading, trailing, or duplicate spaces.
- [ ] All-uppercase/all-lowercase simple names were normalized.
- [ ] Complex capitalization was preserved or reviewed.

## Email
- [ ] Emails are lowercase.
- [ ] Each nonblank email has exactly one `@` and a dotted domain.
- [ ] Emails are unique unless duplicates are explicitly allowed.
- [ ] Every inferred or repaired email is logged.

## Phone
- [ ] Demonstrated-pattern phones match `^\+1-555-\d{4}$`.
- [ ] No phone was reformatted under this rule if its structure was incompatible.

## Signup Date
- [ ] Every accepted date is a real date, not display text.
- [ ] Display format is `yyyy-mm-dd`.
- [ ] No future or out-of-range dates exist unless allowed.
- [ ] Ambiguous numeric dates were resolved by an explicit locale rule or flagged.

## Country
- [ ] Values use approved canonical country names.
- [ ] No unapproved abbreviations remain.
- [ ] Unknown/ambiguous values are logged.

## Order Total
- [ ] Every accepted total is numeric.
- [ ] Display format is `#,##0.00`.
- [ ] No currency symbols/codes remain in values.
- [ ] Negative, blank, or nonnumeric totals comply with an approved business rule or are flagged.

## Status
- [ ] Every accepted status is exactly `Active` or `Inactive`.
- [ ] No unknown abbreviations remain.
- [ ] Status was not inferred from other fields.

## Reconciliation
- [ ] Every changed cell has a transformation-log entry.
- [ ] Every unresolved cell has an exception-log entry.
- [ ] Deterministic changes and inferred repairs are distinguishable.
- [ ] For the supplied reference pair, the cleaned output matches `Customer_Data_Clean.xlsx` cell-for-cell in values and matches its date/number conventions.

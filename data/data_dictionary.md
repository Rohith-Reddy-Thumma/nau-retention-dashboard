# Data Dictionary of NAU Retention Dashboard

> ⚠️ **Privacy Notice:** This dictionary describes the schema of the data used in this project. No actual student data is included in this repository. All student-level records are protected under FERPA and remain within NAU's internal systems.

---

## Source Tables

The dashboard uses three data sources provided by NAU's institutional research office as Excel/CSV exports:

| Table | Description | Rows (approx.) |
|-------|-------------|----------------|
| `F24-S25 Retention` | Student enrollment + rec engagement, Fall 2024 → Spring 2025 | ~20,941 |
| `F24-F25 Retention` | Student enrollment + rec engagement, Fall 2024 → Fall 2025 | ~20,941 |
| `Campus` | Building-to-campus mapping dimension | ~200 |

---

## F24-S25 Retention / F24-F25 Retention (Fact Tables)

Both tables share the same schema. The only difference is the retention outcome field (`Retained or Graduated`) which measures re-enrollment at Spring 2025 vs. Fall 2025 respectively.

| Field | Type | Values | Description |
|-------|------|--------|-------------|
| `*Emplid` | String | Anonymized ID | Student identifier. Anonymized before export - no PII. Primary key for DISTINCTCOUNT measures. |
| `Retained or Graduated` | Integer | 0, 1 | 1 = student re-enrolled or graduated in the target term. 0 = did not return. |
| `Visit_1_Plus` | Integer | 0, 1 | Flag: student visited rec center ≥1 time in Fall 2024. |
| `Visit_5_Plus` | Integer | 0, 1 | Flag: student visited rec center ≥5 times in Fall 2024. |
| `Visit_10_Plus` | Integer | 0, 1 | Flag: student visited rec center ≥10 times in Fall 2024. |
| `Visit_15_Plus` | Integer | 0, 1 | Flag: student visited rec center ≥15 times in Fall 2024. |
| `Visit_30_Plus` | Integer | 0, 1 | Flag: student visited rec center ≥30 times in Fall 2024. |
| `IPEDS Ethnicity` | String | White, Hispanic/Latino, Asian, Two or More, Black/African A., American Indian, International, Native Hawaiian, Unknown | IPEDS-standard ethnicity classification. |
| `Sex` | String | Male, Female, Unknown | Student sex as reported at enrollment. |
| `First Gen Classification` | String | N (No), Y (Yes), U (Unknown) | First-generation college student status. N = has a parent with a 4-year degree. Y = does not. U = not reported. |
| `Academic Level` | String | UGRD, GRAD | Undergraduate or graduate enrollment. |
| `Academic Classification` | String | Freshman, Sophomore, Junior, Senior, Graduate | Classification based on credit hours completed. |
| `Campus` | String | North, South, Unknown | Student's housing campus. Derived from building code via Campus dimension table. |
| `College` | String | Various NAU colleges | Academic college of the student's declared major. |

---

## Campus (Dimension Table)

| Field | Type | Description |
|-------|------|-------------|
| `Building Code` | String | Internal NAU building identifier |
| `Building Name` | String | Human-readable building name |
| `Campus` | String | North or South, the campus assignment for that building |

**Purpose:** Allows the fact tables (which store raw building codes) to be grouped by campus in all visuals and measures without embedding the mapping logic in every DAX measure.

---

## Known Data Quality Issues

| Issue | Description | Impact | Status |
|-------|-------------|--------|--------|
| **Unknown First-Gen (519 students)** | 519 students have `First Gen Classification = U` (Unknown). These students retained at only 31.8%, roughly half the rate of every other segment. | Creates an outlier that skews FGen analysis; this population is invisible to outreach campaigns. | Flagged to leadership. Recommended: audit the enrollment intake form + manual outreach to resolve unknown status. |
| **Unknown Campus** | A small number of students have no building assignment and cannot be mapped to North or South campus. | Minor excluded from geographic analysis totals. | Acceptable data loss. |
| **Visit Count Source** | Visit counts come from swipe records at rec facility entrances. A student may enter multiple times in one day; the system counts unique days, not entries. | Visit thresholds (≥1, ≥5, etc.) represent unique visit days, not total entrances. | Documented in dashboard tooltip on the overview page. |

---

## What Is NOT In This Repo

The following data is intentionally excluded for privacy and FERPA compliance:

- Raw student-level CSV/Excel files
- The compiled Power BI `.pbix` file (contains student records)
- Any file that could be used to re-identify individual students
- NAU internal system credentials or connection strings

If you are a researcher or hiring manager who wants a full walkthrough, contact me directly: **rohiththumma2001@gmail.com**

# DAX Measures of NAU Retention Dashboard

All 16 calculated measures used in the dashboard. Measures are shared across both fact tables (F24-S25 Retention and F24-F25 Retention) via the Campus bridge table where applicable.

---

## Core Count Measures

### Retention Total
```dax
Retention Total = DISTINCTCOUNT('F24-F25 Retention'[*Emplid])
```
**Purpose:** Distinct student count. Anchors all retention rate denominators. Uses DISTINCTCOUNT to prevent double-counting students who appear in multiple rows.

---

### Count of Retained or Graduated for 1
```dax
Count of Retained or Graduated for 1 =
CALCULATE(
    COUNTA('F24-F25 Retention'[Retained or Graduated]),
    'F24-F25 Retention'[Retained or Graduated] IN { 1 }
)
```
**Purpose:** Counts students flagged as retained or graduated (value = 1). Used as the numerator for all retention rate calculations.

---

### Count of Not Retained
```dax
Count of Not Retained =
CALCULATE(
    COUNTA('F24-F25 Retention'[Retained or Graduated]),
    'F24-F25 Retention'[Retained or Graduated] IN { 0 }
)
```
**Purpose:** Counts students who did not return. Used in the pie chart "Not Retained" slice.

---

## Retention Rate Measures

### Overall Retention Rate F24-F25
```dax
Overall Retention Rate F24-F25 =
DIVIDE(
    [Count of Retained or Graduated for 1],
    [Retention Total],
    0
) * 100
```
**Purpose:** Top-level retention percentage for the full cohort. Displayed on the General Retention landing page.

---

### Overall Retention Rate F24-S25
```dax
Overall Retention Rate F24-S25 =
DIVIDE(
    [Count of Retained S25],
    [Retention Total S25],
    0
) * 100
```
**Purpose:** Same calculation for the shorter Fall→Spring retention window. Used on F24-S25 specific pages.

---

## Campus-Specific Measures

### Count of *Emplid for North
```dax
Count of *Emplid for North =
CALCULATE(
    DISTINCTCOUNT('F24-F25 Retention'[*Emplid]),
    'F24-F25 Retention'[Campus] = "North"
)
```
**Purpose:** Student count scoped to North Campus housing. Used as denominator for North Campus retention rate.

---

### Count of NAU ID for North
```dax
Count of NAU ID for North =
CALCULATE(
    DISTINCTCOUNT('F24-F25 Retention'[*Emplid]),
    'F24-F25 Retention'[Campus] = "North",
    'F24-F25 Retention'[Retained or Graduated] = 1
)
```
**Purpose:** Retained students on North Campus. Numerator for North Campus retention percentage.

---

### Percentage North F24
```dax
Percentage North F24 =
DIVIDE(
    [Count of *Emplid for North],
    [Count of NAU ID for North],
    0
) * 100
```
**Purpose:** North Campus retention rate. Compared against South Campus equivalent on the geographic analysis page.

---

### Percentage South F24
```dax
Percentage South F24 =
DIVIDE(
    [Count of *Emplid for South],
    [Count of NAU ID for South],
    0
) * 100
```
**Purpose:** South Campus retention rate. The gap between North (75.6%) and South (72.7%) fed the facility expansion proposal.

---

## Visit Bucket Measures

### REC Usage %  ≥1 Visit
```dax
REC Usage Pct 1 Visit =
DIVIDE(
    CALCULATE(
        DISTINCTCOUNT('F24-F25 Retention'[*Emplid]),
        'F24-F25 Retention'[Visit_1_Plus] = 1
    ),
    [Retention Total],
    0
) * 100
```
**Purpose:** Percentage of students who visited the rec center at least once. Repeated pattern for ≥5, ≥10, ≥15, ≥30 thresholds.

> **Note:** The same pattern (`CALCULATE` + `DISTINCTCOUNT` + bucket flag filter) repeats for all 5 visit thresholds. See measures `REC Usage Pct 5 Visit`, `REC Usage Pct 10 Visit`, `REC Usage Pct 15 Visit`, `REC Usage Pct 30 Visit`.

---

## Campus Resident Measure

### Campus Resident Percentage
```dax
Campus Resident Percentage =
DIVIDE(
    [Campus Resident Count],
    [Retention Total],
    0
)
```
**Purpose:** What proportion of the cohort are campus residents (vs. commuters/off-campus). Used in the North/South geographic analysis to normalize engagement percentages.

---

## Design Notes

**Why DIVIDE instead of `/`:**
All division uses `DIVIDE(numerator, denominator, 0)` rather than the `/` operator. This returns 0 (not an error) when the denominator is zero, critical for filtered views where a demographic slice might have no students in a given bucket.

**Why DISTINCTCOUNT not COUNT:**
Source data contains one row per student per enrollment event. DISTINCTCOUNT on student ID ensures each student is counted once regardless of how many enrollment records they have.

**Why no bidirectional relationships:**
The model uses single-direction relationships from fact tables to the Campus dimension. Bidirectional cross-filters were tested and caused ambiguous filter contexts in the visit-bucket measures removed to maintain calculation accuracy.

**Context transitions:**
All bucket measures use `CALCULATE` to impose an explicit filter context over the base DISTINCTCOUNT. This is intentional without it, row-context iterations from visuals would give incorrect results when cross-filtering by demographic.

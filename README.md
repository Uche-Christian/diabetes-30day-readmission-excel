# diabetes-30day-readmission-excel
[README.md](https://github.com/user-attachments/files/34169497/README.md)
# Diabetic 30-Day Readmissions: Finding Who Needs Extra Care Coordination (Excel)

> I analysed 99,343 diabetic hospital stays and built a simple four-point risk flag. The higher-risk tier is about 1 in 6 discharges and has an observed 30-day readmission rate of **17.7%**, compared with **10.1%** for everyone else. Targeting it would take about **83% fewer coordinator hours** than coordinating every patient.


**Quick links:** [Dashboard (PDF)](https://github.com/user-attachments/files/34169548/Diabeticspic.pdf)
) | [Excel workbook](https://1drv.ms/x/c/834069C157EE23D6/IQD5TVAjs8LTSbmj7KqKNyXQAUsFMvzMEB6DaKRyfnJsYzs?e=KcopfC) | [One-page memo](https://github.com/user-attachments/files/34169614/Readmission_Memo.pdf)
) | [Full build manual](https://github.com/user-attachments/files/34169630/Build_Manual.pdf)
)

*Built for the Healfica Healthcare Data Challenge 01 ("The 30-Day Readmission Penalty").*

---

## At a glance

| Question | Answer |
|---|---|
| Overall 30-day readmission rate | **11.4%** (11,314 of 99,343 encounters) |
| Patterns most associated with higher rates | Prior inpatient stays, discharge to a facility, longer stays, more medications |
| Patterns that looked weak | Age, admission type, admission source |
| Higher-risk tier (score of 3 or more out of 4) | 17,077 encounters (17.2%), observed rate 17.7% vs 10.1% for the rest |
| What the tier captures | 3,023 of 11,314 readmissions (26.7%). The other 8,291 (73.3%) fall outside it |
| Workload per 1,000 discharges | About 172 patients and 215 coordinator hours, vs 1,250 hours to coordinate everyone (assumes 75 minutes per patient) |

---

## 1. Business problem

Meridian Care Network's discharge planners treat diabetic patients in much the same way at discharge. There is no simple method for finding groups of encounters with higher 30-day readmission rates, so care-coordination effort is spread across everyone and no higher-risk tier gets extra attention.

Not every readmission is preventable, and this project does not try to remove them all. It asks a narrower question: **where does extra coordination have the best chance to matter?**

**Stakeholder:** VP of Population Health, Meridian Care Network (a scenario from the challenge brief). The VP wanted to know which patients are at elevated risk before they leave the building, and a case for where to spend a limited care-coordination budget, in a form the discharge planning team can use on a busy morning.

## 2. Dataset

[UCI Diabetes 130-US Hospitals for Years 1999-2008](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008)

| File | Contents |
|---|---|
| `diabetic_encounter_raw.csv` | 101,766 inpatient encounters, 50 variables, 130 US hospitals |
| `IDS_mapping.csv` | Lookup lists that translate numeric codes (admission type, discharge disposition, admission source) into words |

One row is one hospital stay, not one patient. After cleaning, the data holds 69,990 distinct patients, and about 46% of encounters belong to patients with more than one stay.

The outcome column `readmitted` has three values: `<30` (readmitted within 30 days), `>30` (readmitted later) and `NO`. Only `<30` counts as a 30-day readmission. `>30` is kept separate and never merged into `<30`.

## 3. Data preparation

All cleaning was done in **Power Query**, so every step is recorded and repeatable. The raw files were never edited.

| Issue | Decision | Rows affected |
|---|---|---|
| Missing values written as `?` | Replaced with blanks. Heavily missing columns dropped | Per column |
| Outcome has three text values | Built `Readmit30` (1 for `<30`, else 0). Original column kept | 11,357 flagged before exclusions |
| Expired and hospice discharges | **Excluded.** These patients cannot be readmitted, so keeping them lowers the rate | 2,423 removed (101,766 to 99,343) |
| Lookup codes meaning unknown, not available or not mapped | Grouped into one "Unknown" label and kept | Counted per column |
| ID fields stored inconsistently | Standardised to Whole Number before joining. Row counts checked before and after each join | 101,766 before and after |
| Two "Unknown" labels in admission source (hidden spaces) | Trimmed and cleaned, giving one category | 6,854 rows combined |
| Duplicate encounters | Checked. None found | 0 |
| Newborn and Trauma Center admission types | Kept in the data, not interpreted (10 and 18 encounters) | 28 |

The exclusion raised the rate slightly, from 11.2% (11,357 of 101,766) to 11.4% (11,314 of 99,343). Only 43 of the 2,423 excluded encounters were 30-day readmissions.

Columns not used (with reasons, such as `weight`, which is 96.9% missing) are listed in the data dictionary inside the [build manual](https://github.com/user-attachments/files/34169630/Build_Manual.pdf).

## 4. Questions I asked

1. What is the overall 30-day readmission rate?
2. How does the rate vary by age band, admission type and length of stay?
3. Which characteristics look most associated with 30-day readmission?
4. Do the patterns translate into a simple, explainable risk flag?
5. What share of encounters falls in the higher-risk tier?
6. What extra coordination workload does that tier create?

Every rate in the dashboard is shown with its group size, and groups under 500 encounters were not interpreted.

## 5. Approach

1. **Understand** the data and write a problem statement.
2. **Clean** in Power Query: types, missing values, outcome flag, lookup joins, exclusions, helper bands.
3. **Model** the data in the Excel Data Model with DAX measures for encounters, readmissions, rate, overall rate and lift.
4. **Explore** one variable at a time with PivotTables, comparing rate, group size and spread.
5. **Build a risk flag** from the strongest patterns and test several cutoffs.
6. **Estimate workload** with clearly labelled, editable assumptions.
7. **Present** the results in a one-page dashboard and a memo.

## 6. Key findings

Group differences below are **associations**, not causes.

| Characteristic | Observed 30-day rate |
|---|---|
| Prior inpatient stays: none / 1 / 2 / 3 or more | 8.6% / 13.3% / 17.9% / **26.4%** |
| Discharged to home vs to a facility | 9.3% vs **16.4%** |
| Length of stay: 1-2 days vs 8+ days | 9.3% vs **13.8%** |
| Medications: 1-10 vs 21+ | 9.2% vs **13.2%** |

- **Prior inpatient stays showed the widest gap.** Encounters with 3 or more prior inpatient stays have an observed rate of 26.4% (n = 6,814), compared with 8.6% for those with none (n = 66,245).
- **Age and admission type showed little difference.** Admission type ranged only from 10.5% to 11.8% among large groups.
- **Group size matters.** The 0-10 age band (n = 160) and the Newborn and Trauma Center admission types showed unusual rates that rest on very few encounters, so they were not interpreted.

## 7. The risk flag

The flag gives one point for each criterion met, so the score runs from 0 to 4. Each cutoff sits where that group's rate rises above the overall 11.4%.

| Criterion | Cutoff |
|---|---|
| Prior inpatient stays | 1 or more |
| Discharge destination | Facility (skilled nursing, rehab, another hospital or similar) |
| Length of stay | 5 or more days |
| Medication count | 16 or more |

The rate rises at every step of the score:

| Score | 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| Observed rate | 6.4% | 10.6% | 13.1% | 17.2% | 20.1% |

**Higher-risk tier = score of 3 or more.** Wider tiers flag too many encounters (a score of 2 or more flags 43.5%), and a score of 4 alone reaches only 3.2%.

| | Encounters | Readmissions | Rate |
|---|---|---|---|
| Higher-risk tier | 17,077 (17.2%) | 3,023 (26.7% of all) | **17.7%** |
| Everyone else | 82,266 (82.8%) | 8,291 (73.3% of all) | 10.1% |

**The flag misses most readmissions.** That is normal for a simple rule. It does not find everyone who will return. It finds a smaller group whose rate is much higher than average. It is an analytical flag based on observed patterns, not a diagnosis or a guaranteed prediction, and it supports clinical judgment rather than replacing it.

## 8. Workload estimate

The data has no coordination times, so the estimate uses stated assumptions that are editable in the workbook:

- 4 contacts per patient: 1 pre-discharge session (30 min), 2 follow-up calls (15 min each), 1 appointment check (15 min) = **75 minutes per patient**
- 1,500 productive hours per coordinator per year

| Option (per 1,000 discharges) | Patients | Coordinator hours | Readmissions among them |
|---|---|---|---|
| Coordinate everyone | 1,000 | 1,250 | 113.9 |
| Target score of 2 or more | 435 | 544 | 65.0 |
| **Target score of 3 or more** | **172** | **215** | **30.4** |

The saving does not depend on the minutes assumed: the tier is 17% of patients, so it takes about 17% of the effort whatever the time per patient. "Readmissions among them" counts the readmissions that happen in each group. It does **not** claim coordination would prevent them.

## 9. Recommendation

Start extra coordination with the higher-risk tier (score of 3 or more). If capacity allows, widen to a score of 2 or more, which reaches 57.1% of readmissions but needs about 2.5 times the hours. Review the cutoffs against Meridian's own recent data before relying on them.

## 10. Limitations

- **Association, not causation.** The data records what happened, not why.
- **Historical data.** It covers 1999 to 2008 across 130 hospitals, not Meridian's current patients.
- **Cutoffs were set on the same data they are measured on**, so rates inside the tier are probably slightly optimistic on new data.
- **Repeat patients.** Stays from the same patient are not independent. A check keeping one encounter per patient gave a tier rate of 16.7% vs 8.1% for the rest, so the pattern held.
- **The flag misses 73% of readmissions.**
- **Overlapping criteria.** Length of stay and medication count partly measure the same "complex stay" signal.
- **Workload times are assumptions**, not measurements.

## 11. Tools and skills demonstrated

Excel, Power Query (M), Power Pivot and the Data Model, DAX measures, PivotTables and PivotCharts, slicers, data cleaning and documentation, KPI and dashboard design, risk scoring, scenario analysis and plain-language communication for a non-technical stakeholder.

## 12. Repository contents

```
.
├── README.md
├── images/       Dashboard screenshots
├── dashboard/    Dashboard PDF
├── workbook/     Excel workbook (or a link to it)
├── data/         Cleaned data (or a link to the UCI source)
└── docs/         One-page memo, build manual, data dictionary and cleaning log
```

## 13. Data source and credit

Data from the UCI Machine Learning Repository: [Diabetes 130-US Hospitals for Years 1999-2008](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008). See the dataset page for licence terms.

Strack, B., DeShazo, J. P., Gennings, C., Olmo, J. L., Ventura, S., Cios, K. J., & Clore, J. N. (2014). Impact of HbA1c measurement on hospital readmission rates: analysis of 70,000 clinical database patient records. *BioMed Research International*.

Project brief and scenario by the Healfica community (Healthcare Data Challenge 01).

## 14. About me

**Nnanna Uche Christian** |  [Email](uche.nnanna00@gmail.com)

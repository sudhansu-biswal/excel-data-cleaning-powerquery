# From Three Messy Monthly Sheets to One Refreshable Dataset
### Fabrication DPR cleanup — Excel 365 and Power Query

> **The source file could not produce its own totals.** 121 of 255 rework-cost values and 47 of 255 joint counts were stored as text, so any pivot built over the raw data would have silently skipped them and under-reported both rework spend and rejection rate. After rebuilding the file as a Power Query pipeline, the same data returns **3,202 joints welded, a 16.0% rejection rate and ₹11,56,403 of rework cost** — and the next month is added by clicking Refresh.

*Synthetic dataset, modelled on real fabrication DPR structure. Contractor, welder and area names are invented.*

---

## The problem

Monthly welding DPR data was being produced in a usable form for people, but not for analysis.

Each month arrived as its own worksheet with presentation formatting, merged title blocks, inconsistent spellings, mixed data types, note rows, blank separators and footer text sitting inside the data range. Before any contractor, welder, rejection or rework-cost analysis could start, someone had to clean and standardise it — and then do the same thing again the following month.

That created two problems, and the second is the expensive one:

- **Preparation was displacing analysis.** The same cleanup logic was rebuilt from scratch every month.
- **Manual repair changes the answer.** Dates could reverse by locale. One contractor could split into three groups through spelling variation. Numeric-looking text could drop out of a total without warning — which is exactly what was happening.

The objective was not to clean a spreadsheet once. It was to convert a recurring repair task into a repeatable preparation system with a controlled, identical output every month.

---

## The data

Three monthly sheets — January to March 2026 — containing **320 occupied rows**, of which only **255 were actual DPR records**. The remaining 65 rows were title blocks, headers, blank separators, subtotals and footer text.

Fields covered DPR date, contractor, welder ID, area, shift, joint type, joint reference, total joints, rejected joints and rework cost.

The sheets shared a business meaning but not a structure. The rebuilt dataset was defined at an explicit grain:

> **one row = one welder × one day × one area × one joint type**

Final output: **255 rows, 12 typed columns, one consolidated fact table.**

---

## What was wrong with it

These were not typos. They were systematic faults that would return in every future month's file.

**Structural**
- Three separate sheets instead of one queryable table
- Four-row merged title blocks ahead of the real headers
- 34 blank separator rows inside the data ranges
- 16 junk text, subtotal and footer rows mixed in with records
- Month context carried only by the sheet name, which was itself inconsistent

**Standardisation**
- 5 different date formats in one text column, including ambiguous slash dates
- 10 contractor spellings for 3 actual contractors
- 11 welder-ID spellings for 6 actual welders
- 9 area spellings, 9 shift spellings, 8 joint-type spellings
- 54 values carrying trailing whitespace — each one a potential false group

**Data type**
- 47 of 255 `Total Joints` values stored as text
- `Rejected` using blanks, `-`, `nil` and `NIL` interchangeably for zero
- 121 of 255 rework-cost values stored as text, with `Rs.` prefixes and comma separators

Some of these could not be fixed by find-and-replace, because they required a judgement. `Konark Engineering`, `konark engg` and `Konark Engineering Pvt Ltd` are one contractor. `Gen` means `General`. Those decisions were put into visible mapping tables rather than buried in ad-hoc edits, so anyone can audit or change them later.

---

## What I built

A Power Query pipeline, not a repaired workbook. Six decisions carry it:

1. **A query per monthly sheet**, stripping the four presentation rows before promoting real headers.
2. **`Source_Month` stamped before the append**, so month context survives independently of sheet naming.
3. **Non-data rows removed by record logic** — filtering where `Welder_ID` is null — rather than by maintaining a list of note text to delete. Next month's new footer wording won't break it.
4. **Dates parsed across all five patterns and validated against their source month**, which is what prevents a silent US/India reversal.
5. **Numeric fields typed only after stripping prefixes, separators and zero tokens**, with a `Rejected_Source` field preserving whether a zero was entered explicitly or inferred from a blank. The assumption stays auditable.
6. **All months appended into one `Fact_Welding` table** with a refreshable summary layer on top.

`Joint No.` was renamed `Ref_Joint_No` after confirming it was a DPR reference field rather than a unique joint key — a distinction that would have corrupted every count built on it.

---

## What the clean data showed

Once the file could be trusted, the analysis it had been blocking came out immediately:

- **3,202 joints welded, 513 rejected — a 16.0% overall rejection rate**
- **₹11,56,403 total rework cost** across 6 welders and 3 standardised contractors
- **Sai Fab Works carried the highest total rework cost**, driven by volume rather than quality
- **Utkal Fabricators had the highest rejection rate**, which is the different and more actionable problem
- **Unit 3 Structure was the worst-performing area**

That distinction — highest cost versus highest rate — is the one that decides where corrective action goes, and it was unreachable while three contractors were spread across ten spellings.

---

## The result

**Before:** 3 sheets, 320 mixed rows, five date formats, 168 numbers stored as text, and a full manual repair required before any question could be answered.

**After:** 255 clean rows, 12 typed columns, one fact table, documented mapping logic, and a workflow where next month means dropping in the file and clicking Refresh.

The one-time cleanup is not the deliverable. The reusable cleanup logic is.

---

**Tools:** Excel 365 · Power Query (M) · Pivot Tables

**Files:** `01_before_messy.xlsx` · `02_after_clean.xlsx` · `defect_log.md` · `images/before_after.png`

---

*I spent 12 years in QA/QC on refinery, steel plant and fabrication projects — IOCL Paradip, Jindal Steel Angul, Tata Steel HSM. I have built and repaired more site reporting spreadsheets than I can count. This is what I wish they had been.*

# Defect Log — Fabrication Monthly Welding Logs

**Source file:** `fabrication_monthly_logs_MESSY.xlsx`
**Sheets:** `JAN 2026`, `FEB'26`, `MARCH 2026`
**Assessed:** before any cleaning. All counts verified against the raw file.

---

## 0. Starting position (the "before" numbers)

| Sheet | Rows occupied | Actual data rows | Junk rows | Blank rows |
|---|---|---|---|---|
| JAN 2026 | 101 | 80 | 5 | 11 |
| FEB'26 | 108 | 86 | 5 | 12 |
| MARCH 2026 | 111 | 89 | 6 | 11 |
| **Total** | **320** | **255** | **16** | **34** |

Three sheets, ten columns, no single queryable table. 255 usable rows out of 320 occupied rows — **20% of the file is noise.**

---

## A. Structural defects — these break the import before any column work starts

**A1. Merged title block occupies rows 1–4 on every sheet.**
`A1:J1` and `A2:J2` are merged; `A3:D3` and `F3:J3` are merged. Power Query reads merged cells as one value plus nulls, so a straight import produces four garbage rows and column headers named `Column1`–`Column10`.
*Fix:* Remove Top Rows = 4, then Use First Row as Headers.

**A2. Header row sits on row 5, not row 1.**
Consequence of A1. Any tool that assumes row 1 = header will mislabel every column.

**A3. Data is split across three sheets that should be one table.**
Identical column structure, split only by month. Every analysis in the client's Section 3 (highest rework cost, rejection totals to date) requires all three together.
*Fix:* Build one query per sheet, then Append Queries as New into a `Fact_Welding` query.

**A4. Sheet names are not machine-readable and are the only record of the month.**
`JAN 2026`, `FEB'26`, `MARCH 2026` — three different conventions, one containing an apostrophe. If a date value is ever blank, the month is unrecoverable after the append.
*Fix:* Add a `Source Month` custom column in each query **before** appending.

**A5. 34 blank separator rows scattered inside the data range.**
Not at the end — between records. They will load as null rows.
*Fix:* Remove Blank Rows after promoting headers.

**A6. 16 junk text rows sitting inside the data range.**
Free-text notes typed into column A with all other columns empty:
- `*** site shutdown - no welding ***` (2×)
- `--- new WPS issued from this date ---` (4×)
- `Note: RT film pending for above joints` (1×)
- `sub total ->` (3×)
- `TOTAL FOR THE MONTH` (3×, footer)
- `Checked by: ____________` (3×, footer)

*Fix:* Filter out rows where `Welder ID` is null. That single step removes all 16 plus the blanks — do not filter on the text itself, which is unmaintainable when new notes appear next month.

**A7. Footer signature block after the data ends.**
`TOTAL FOR THE MONTH`, `Checked by`, `Approved by` on each sheet. Handled by A6, but must be verified rather than assumed.

---

## B. Column-level defects

**B1. Date — five formats in one column, all stored as text.**
`dd-mm-yyyy` (52) · `dd/mm/yyyy` (53) · `yyyy-mm-dd` (61) · `d Mon yyyy` (43) · `d.m.yyyy` (46).
Highest risk in the file: `03/02/2026` is 3 Feb under Indian convention and 2 March under US locale. A locale-based type conversion will silently produce wrong months, not an error.
*Fix:* Do **not** use Change Type → Date directly. Split the handling by pattern, or use Change Type With Locale → English (India). Then validate that every parsed date falls inside its own sheet's month — that check is your proof the conversion was correct.

**B2. Welder ID — 11 spellings for 6 actual welders.**
`WLD-004` / `wld-004` / `WLD 004` · `WLD-011` / `wld-011` · `WLD-017` / `WLD-017 ` (trailing space) · `WLD-023` / `wld-023` · `WLD-031` · `WLD-042`.
*Fix:* Trim → Clean → Uppercase → Replace `" "` with `"-"`.

**B3. Contractor Name — 10 spellings for 3 firms.**
- Sai Fab Works: `Sai Fab Works`, `sai fab works`, `SAI FAB WORKS`, `Sai Fab Works ` (trailing space)
- Konark: `Konark Engineering`, `konark engg`, `Konark Engineering Pvt Ltd`
- Utkal: `Utkal Fabricators`, `UTKAL FABRICATORS`, `Utkal Fab`

Case and whitespace are mechanical. `konark engg` → `Konark Engineering Pvt Ltd` and `Utkal Fab` → `Utkal Fabricators` are **judgement calls requiring client confirmation** (see Section D).
*Fix:* Trim + Proper Case for the mechanical variants; a mapping table for the abbreviations, never hardcoded Replace Values.

**B4. Area / Location — 9 spellings for 3 areas.**
`Pipe Rack A` / `pipe rack a` / `PIPE RACK-A` · `Tank Farm` / `tank farm` / `TANK FARM` · `Unit-3 Structure` / `unit 3 structure` / `Unit 3 Structure`.
Note the hyphen inconsistency — case correction alone leaves `PIPE RACK-A` separate from `Pipe Rack A`.

**B5. Shift — 9 spellings for 3 shifts, including an abbreviation.**
`Day`/`day`/`DAY` · `Night`/`night`/`NIGHT` · `Gen`/`General`/`general`. `Gen` = `General` needs confirming.

**B6. Joint Type — 8 spellings for 3 types.**
`Butt`/`butt`/`BUTT` · `Fillet`/`fillet` · `Socket`/`socket`/`SOCKET`. Case only, no abbreviations. Cleanest column in the file.

**B7. Total Joints — 47 of 255 values stored as text.**
Numeric-looking strings. They will not sum, and will sort alphabetically (`10` before `9`).
*Fix:* Change Type → Whole Number, then check the error count is zero before proceeding.

**B8. Rejected — four different tokens used for zero.** *(resolved — see G1)*
Blank (9), `-` (3), `nil` (8), `NIL` (8). 28 rows total.
*Fix:* Replace all four with `0`, then Change Type → Whole Number. **Decide and document whether blank means "zero rejects" or "not recorded"** — they are not the same thing and the rejection rate changes depending on the answer.

**B9. Rework Cost — 121 of 255 values stored as text.**
50 carry an `Rs. ` prefix (`Rs. 12,400`); the rest carry thousand separators or are blank/`-`/`0`.
*Fix:* Replace `"Rs. "` with `""` → replace `","` with `""` → replace `"-"` with `"0"` → Change Type → Decimal Number.

**B10. Trailing whitespace on 54 values across text columns.**
Invisible on screen, fatal to grouping — `Sai Fab Works` and `Sai Fab Works ` group as two contractors.
*Fix:* Trim every text column. Apply it to all of them, not only the ones where you spotted a problem.

**B11. Header text itself is unclean.**
`Date ` has a trailing space. `Joint No.`, `Area / Location`, `Rework Cost (Rs)` contain punctuation and spaces that make DAX and formula references awkward.
*Fix:* Rename all ten headers to clean names after promotion.

---

## C. Validation checks to run after cleaning

| Check | Expected | Found in raw file |
|---|---|---|
| Row count after clean | 255 | — |
| Rejected ≤ Total Joints, every row | 0 violations | 0 violations ✔ |
| Duplicate `Ref_Joint_No` across all sheets | not a key — duplicates permitted | 3 repeats — expected, see G2 |
| Every parsed date inside its own sheet's month | 100% | to verify post-clean |
| Distinct contractors after standardisation | 3 | 10 before |
| Distinct welders after standardisation | 6 | 11 before |
| Distinct areas / shifts / joint types | 3 / 3 / 3 | 9 / 9 / 8 before |

Superseded by G2 — the joint reference is not a unique key, so repeats are expected and are not a defect.

---

## D. Open questions for the client

1. **Contractor consolidation.** Confirm `konark engg` and `Konark Engineering` are the same entity as `Konark Engineering Pvt Ltd`, and `Utkal Fab` the same as `Utkal Fabricators`.
2. **Shift naming.** Is `Gen` the same as `General`? Is `General` a third shift or a day-shift synonym?
3. **Blank Rejected cells.** Zero rejects, or not recorded? Affects every rejection-rate figure.
4. **Welder ID uniqueness.** Is welder numbering project-wide, or does each contractor number its own welders from 1? If the latter, `WLD-004` is not a unique key and every welder-level ranking is invalid without a Contractor + Welder composite key.
5. **Duplicate Joint Numbers.** Repeat inspections of the same joint, or data-entry duplicates?
6. **Area naming.** Is `Unit-3 Structure` the same physical area as `Unit 3 Structure`?

Items 3 and 4 block the analysis. The rest can be assumed with the assumption documented.

---

## E. Assumption register

Anything the client does not answer gets recorded here with the assumption taken, and this register goes into the final handover pack:

| # | Question | Assumption taken | Impact if wrong |
|---|---|---|---|
| 1 | Contractor variants | Same entity, consolidated | Contractor comparison invalid |
| 2 | Gen = General | Same shift | Shift analysis splits one shift into two |
| 3 | Blank Rejected | Zero rejects | Rejection rate understated |
| 4 | Welder ID scope | Project-wide unique | Welder ranking merges different people |
| 5 | Duplicate Joint No. | Retained, flagged | Joint count overstated by 3 |
| 6 | Unit-3 / Unit 3 | Same area | Area analysis splits one area |

---

## G. Resolved decisions

**G1. Rejected — blanks, `nil`, `NIL` and `–` all read as zero rejects.**
Basis: this is a DPR-style production log. Inspectors write a figure when joints are rejected and leave the cell alone when none are. All 28 non-numeric entries convert to `0`.
Retained control: a `Rejected_Source` column marks each row as `Recorded` (19 rows carrying an explicit `nil`/`NIL`/`–`) or `Blank` (9 rows genuinely empty), so the rejection rate can be recalculated with the 9 ambiguous rows excluded as a sensitivity check.

**G2. `Joint No.` is a reference field, not a primary key. Table grain redefined.**
Finding: rows carry a single joint identifier alongside a `Total Joints` count greater than one — internally contradictory if the field were joint-level. The column records the first joint in a day's range, as is common in DPR practice.
Decision: rename to `Ref_Joint_No`, exclude from all counting and joining, and define the grain of the fact table as:

> **one row = one welder × one day × one area × one joint type** (a DPR line item, not a joint record).

Consequence: the three repeated references are separate DPR lines, not duplicate records. No rows are removed on this basis.

**G3. Only a DPR date is available. Stated limitation.**
There is no separate joint-preparation date, fit-up date or inspection date. All activity on a row is assumed to have occurred on the DPR date.
Analyses this rules out, to be declared in the deliverable rather than left for the client to discover:
- prep-to-inspection lag
- repair turnaround time
- any time-in-process or ageing measure

Date-based reporting is therefore limited to volume, rejection rate and cost by day, week and month.

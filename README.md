# Illinois State Budget Dataset

This project creates separate, research-oriented expenditure and revenue datasets from Illinois Office of Comptroller records. It combines fiscal years, tracks reused fund numbers and agency reorganizations, attaches descriptive labels, and adds classifications developed for the Fiscal Futures project.

The datasets retain many items excluded from Fiscal Futures fiscal-gap calculations, including transfers and selected debt-related items. They support inspection and custom aggregation; summing every row does **not** reproduce the Fiscal Futures fiscal gap.

## Data available

The most recent local CSV pair reviewed is dated September 13, 2026:

| Dataset | Fiscal years present | Rows | Amount field |
|---|---|---:|---|
| `budget_data_exp_2026-09-13.csv` | 1998–2026 | 190,517 | `expenditure` |
| `budget_data_rev_2026-09-13.csv` | 1998–2026 | 75,996 | `receipts` |

These counts describe saved files, not a fresh pipeline run. A year appearing in a file does not establish that its data are final or complete. The date in the filename is the export date, not the fiscal year or source-data cutoff.

**Coverage qualification:** the expenditure exporter keeps only `expenditure > 0`, removing zero, negative, and missing values. The revenue exporter has no equivalent positive-value filter; the reviewed revenue file contains 788 zero or negative receipts. Earlier cleaning also removes expenditure rows without a fiscal year or fund, and an inner join can remove revenue records without matching fund metadata.

## Repository guide

| File or folder | Purpose |
|---|---|
| `inputs/ioc_data_received/` | Comptroller source files, including historical Stata files and newer Excel deliveries. |
| `inputs/funds_ab_in.xlsx` | Maintained fund crosswalk, labels, categories, and Fiscal Futures inclusion information. |
| `inputs/ioc_source.xlsx` | Revenue-source labels and classification lookup. |
| `code-cleaning.qmd` | Imports and combines years, recodes funds/agencies, joins metadata, and writes intermediate CSVs. |
| `data/FY2026 Files/exp_temp.csv` and `rev_temp.csv` | Intermediate inputs currently used by the final dataset script. |
| `all_data_cleaning_and_recoding.qmd` | Adds expenditure/revenue classifications and writes the dated research CSVs. |
| `GOMB_categories.qmd` | Separate comparisons with GOMB totals; currently configured for FY2025. |
| `CategoryRecoding.qmd` | Older FY2024 category and fiscal-gap workflow; not an additional required step in the current research export. |
| `Instructions_UpdatingFiles_EachYear.docx` | Annual update notes, including manual review of new and reused funds. |

See [Data construction and research decisions](Data-Construction-and-Research-Decisions.md) for implementation details, source references, and unresolved questions.

## How the data are constructed

```text
Comptroller annual expenditure and revenue files + previous combined files
  -> standardize column names and append the new year
  -> review new/reused funds, agencies, organizations, and revenue sources
  -> recode historical fund numbers and agency identifiers
  -> join maintained fund and revenue-source metadata
  -> exp_temp.csv and rev_temp.csv
  -> add functional groups, detailed labels, and fiscal-gap exclusion flags
  -> dated expenditure and revenue CSVs
```

Fund identifiers are research identifiers after harmonization. Reused numbers may acquire a `9` prefix or a five-digit replacement to distinguish different historical purposes. Keep identifiers as text when importing; leading zeros carry meaning.

## Principal variables

| Fields | Meaning and use |
|---|---|
| `fy` | Fiscal year. |
| `fund`, `agency` | Harmonized identifiers; consult the source code and lookup files when comparing with original Comptroller codes. |
| `fund_name_ab`, `fund_cat_name` | Maintained fund labels and broad fund categories. |
| `in_ff`, `drop_fund` | Fund inclusion information used in the Fiscal Futures workflow. These alone do not define a complete fiscal-gap sample. |
| `drop_reason` | An exclusion annotation; a marked row can still be present in these research files. Later rules can overwrite earlier reasons. |
| `expenditure`, `receipts` | Expenditure and revenue amounts, respectively. The reviewed export code applies no inflation adjustment or division into millions. |
| `appr_org`, `obj_seq_type`, `object`, `sequence`, `type` | Expenditure appropriation identifiers and components; transfer records receive special treatment. |
| `group`, `group_name`, `item_detail`, `pension` | Expenditure classifications and detail. |
| `source`, `source_name`, `source_name_AWM`, `rev_type`, `rev_type_name` | Revenue-source identifiers, labels, and classifications. |

Both reviewed CSVs include an unnamed first column written by R's default `write.csv()` row-name behavior. It is not a substantive identifier. Empty cells represent exported missing values.

## Reproduction and annual updates

Use R with Quarto and open `Illinois_State_Budget_Dataset.Rproj` from the repository root. The main export loads `tidyverse`, `lubridate`, `scales`, `kableExtra`, `ggplot2`, `readxl`, `data.table`, and `janitor`; its HTML tables also use `DT`, and chunk configuration uses `knitr`.

1. Preserve the annual source files in their expenditure/revenue folders and create `data/FY<year> Files/`.
2. Update the import filenames, source column mappings, `current_year`, and `past_year` in `code-cleaning.qmd`. The saved version uses FY2025 and hard-coded FY2025 filenames.
3. Run the current-year cleaning chunks manually in order. The script sets `eval = FALSE`; rendering it does not rebuild its intermediate data. Skip the historic Stata reconstruction chunk during routine annual updates.
4. Review new/reused identifiers and update both lookup workbooks and the year-specific recodes. Inspect unmatched records before joining.
5. Write and validate the new year's intermediate `exp_temp.csv` and `rev_temp.csv`.
6. Set `current_year` consistently in `all_data_cleaning_and_recoding.qmd`, then run it to produce the dated research CSVs. It currently uses FY2026.
7. Check year coverage, row counts, lookup matches, labels, and amount totals before distributing a release. Record source vintage and whether each annual delivery is preliminary or final.

The workflow was reviewed but **not executed end to end** for this documentation. Existing FY2026 intermediates are present, but the checked cleaning source is still configured for FY2025. There is no verified one-command rebuild of the latest exports.

## Website and methodology notes

The repository contains Quarto source pages and rendered HTML. No `_quarto.yml` website configuration was found in the reviewed checkout, so a complete website build or deployment command is not established here. Several prose links refer to the separate Fiscal-Futures-Topics project.

The detailed notes distinguish documented rationale, implemented operations, saved-file observations, and unresolved issues. In particular, users should review appropriation-label filling, classification flags, and differences between current and historical refund methods before constructing analytical totals.

## Provenance and citation

Source data: Illinois Office of Comptroller records described in project notes as obtained through FOIA. Data preparation and classifications: this repository's scripts and maintained crosswalks.

When citing or sharing results, record the repository, exact CSV filenames, fiscal years used, access date, and your filtering/aggregation choices. Include the applicable code revision and source-data vintage where available.

Documentation reflects local files reviewed September 16, 2026. Local HEAD was `7401e517ad524bb64af0a67c702c0344cabb7528`; modified scripts and untracked exports mean that commit alone does not identify the reviewed files.

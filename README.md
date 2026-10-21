# Pseudoword spelling analysis

Data and R analysis code associated with the manuscript **Literacy Skill Shapes Pseudoword Spelling Accuracy and Agreement Across Sublexical Grain Sizes**.

The analyses examine pseudoword spelling accuracy under corpus- and sample-based scoring, agreement in spelling responses, and associations between literacy skill and phonographeme (PG) and onset-rime (OR) consistency and frequency.

## Files

| File | Contents |
| --- | --- |
| [`ntr_pw_spell_9-24-26.Rmd`](ntr_pw_spell_9-24-26.Rmd) | R Markdown analysis code, including data preparation, mixed-effects models, and figures. |
| [`ntr_pw_spell_data_final.xlsx`](ntr_pw_spell_data_final.xlsx) | Source workbook with pseudoword spelling responses, participant characteristics, assessment scores, and additional worksheets. |

## Data

The analysis imports **Spellings** and **Demographics** from the workbook.

- **Spellings:** 80 stimulus rows and 79 participant columns. The first two columns are `target_spelling` and `pronunciation`; the remaining column headers are coded participant IDs. Each participant cell contains a spelling response or is blank. There are 5,836 nonblank responses out of 6,320 possible participant-by-item observations; 484 cells are blank. Blank cells are not spelling responses and should not be converted to zero-valued responses.
- **Demographics:** participant-level characteristics and literacy/assessment scores, joined to responses using `ID`. All 79 response-column IDs have matching demographic records. This worksheet also contains one additional record without a matching response column; the response-to-demographics left join does not add that participant to the analysis.

The cleaned workbook contains only `Spellings` and `Demographics`. It replaces the earlier seven-sheet workbook in the current repository. Its spelling and demographic values match the corresponding analysis sheets in that earlier workbook; the response worksheet has been renamed to `Spellings`. The supplied cleaned workbook is uploaded without altering its contents.

Among the 79 response participants, one education value and one spelling raw score are missing. The missing values are preserved, not imputed. Models may omit records with missing predictors, so their participant and trial counts can differ. The import reports these counts in `missing_covariates`. Other demographic/assessment fields also contain missing values.

### Variables used in the analysis

| Variable | Description |
| --- | --- |
| `target_spelling` | Reference spelling label for a stimulus; it is not necessarily the only plausible response. |
| `pronunciation` | Target pronunciation supplied to the transcription converter. |
| Participant-ID columns | Responses in wide format, reshaped into `ID` and `spelling_response` by the analysis. |
| `ID` | Coded participant identifier used to join response and demographic records. |
| `Age` | Age in years. |
| `Sex` | Recorded sex category. |
| `Education` | Years of education. |
| `WRMT_WID` | Woodcock Reading Mastery Tests Word Identification standard score. |
| `WRMT_WA` | Woodcock Reading Mastery Tests Word Attack standard score. |
| `WRMT_BSC` | Woodcock Reading Mastery Tests Basic Skills Composite score. |
| `WJTA_SPELL` | Woodcock-Johnson spelling raw score. |

The workbook also includes race, ethnicity, education category, number of languages spoken, and a WASI score. Derived Toolkit estimates and scoring variables are calculated by the R code rather than stored as a complete trial-level analysis dataset in the workbook.

## Software and external files

Use R with RStudio or another R Markdown-capable environment. The exact R and package versions used for the original analyses are not recorded in this repository.

Install the packages referenced by the analysis, plus the R Markdown rendering packages:

```r
packages <- c(
  "openxlsx", "readxl", "readr", "dplyr", "tidyr", "tibble",
  "stringr", "stringi", "ggplot2", "lme4", "lmerTest", "corrplot",
  "ggcorrplot", "patchwork", "purrr", "emmeans", "performance",
  "parameters", "ppcor", "reshape2", "broom", "scales", "knitr", "rmarkdown"
)
install.packages(setdiff(packages, rownames(installed.packages())))
```

The analysis also requires **`Toolkit_v2.0.RData`** and **`Transcription_Converter.R`**, which are not included here. The code expects Toolkit tables and functions such as `all_tables_PG`, `all_tables_OR`, `map_value()`, `summarize_words()`, and `pw_spell()`, and the converter function `ipa_to_inhouse()`.

Toolkit resources supplied with the study:

- [Sublexical Toolkit materials on OSF](https://osf.io/e95qw/overview)
- [Maintained Sublexical Toolkit repository](https://github.com/sublexicaltoolkit/toolkit-2.1)

The analysis explicitly loads version 2.0 files. A newer Toolkit version has not been verified as a drop-in replacement for those files.

## Running the analysis

1. Clone or download this repository and set its folder as the working directory in RStudio.
2. Open `ntr_pw_spell_9-24-26.Rmd`. The default data file is `ntr_pw_spell_data_final.xlsx` in that same folder.
3. Obtain the external Toolkit and converter files. Put them in a local `toolkit/` subfolder, or set their paths before running the first chunk:

```r
# Optional overrides; use the actual locations on your computer.
Sys.setenv(
  NTR_DATA_PATH = "/path/to/ntr_pw_spell_data_final.xlsx",
  NTR_TOOLKIT_PATH = "/path/to/Toolkit_v2.0.RData",
  NTR_CONVERTER_PATH = "/path/to/Transcription_Converter.R"
)
```

   Defaults are `ntr_pw_spell_data_final.xlsx`, `toolkit/Toolkit_v2.0.RData`, and `toolkit/Transcription_Converter.R`. Both Toolkit loads use the same configured path.
4. Install the packages above. The first chunk now attaches `purrr` explicitly for `map_chr()`. Its join validation uses a recent version of dplyr supporting `relationship = "many-to-one"`.
5. Run chunks in order from a fresh R session. The import checks the workbook sheets and demographic columns, rejects duplicate or unmatched demographic IDs, and reports missing covariates. Responses are lowercased, whitespace is removed, pronunciations are converted, and wide responses are reshaped and joined to demographics. The extra demographic record does not enter the response sample.
6. Inspect the limitations below before knitting the entire document. Figures saved with relative paths are written to the active working directory. Once the unresolved objects are addressed, render using RStudio's Knit command or `rmarkdown::render("ntr_pw_spell_9-24-26.Rmd")`.

## Current reproducibility limitations

Uploading the source workbook does not by itself make this analysis fully self-contained. This update switches the imports to the cleaned two-sheet workbook, configures file paths, attaches purrr, and adds participant-join and missing-covariate checks. Statistical models and scoring logic are unchanged. Workbook structure, response counts, participant matching, and equivalence with the prior analysis sheets were checked. A complete clean-session R run has not been performed; R and the external Toolkit files were unavailable in the editing environment.

Static inspection identified existing issues that need resolving before an uninterrupted render:

- The legal-spelling plotting chunk constructs `plot_df_wide_nopos` but uses `plot_df_wide` and differently named columns/plot objects later in the same chunk.
- The cumulative sample models refer to `transcriptions_long3_z`, which is not created in the current document.
- The final Word Attack figure is assigned to `plot_partial_WA`, while its `ggsave()` call refers to `plot_partial_min`.
- In the supplementary accuracy plot, non-positional legal bars are created by duplicating positional accuracy values; they are not independently calculated non-positional estimates in that plotting chunk.

These observations are not a full statistical or code audit. External files and their versions must also be available to reproduce the reported results.

## Access and citation

The data workbook is included in this repository. Access follows the repository's visibility and collaborator permissions; a private repository link is not sufficient on its own to give reviewers access.

When referring to these materials, cite the accompanying manuscript by its title above and identify the repository commit or release used. Publication details can be added once available.

# Data dictionary for patent commercialization probabilities

Release `v1` contains 8,426,952 rows, one per scored U.S. utility patent. Export verification confirms that Parquet and CSV preserve the source probabilities exactly. The source file was unchanged.

## Fields

Fields appear in the following order in both formats.

| Field | Parquet type | CSV representation | Definition |
| --- | --- | --- | --- |
| `patent_id` | String | String of decimal digits | U.S. patent grant identifier, copied from the master identifier and represented as a string. Unique and nonmissing in the audited source. Use as the merge key. |
| `application_year` | Nullable int32 | Integer, blank if missing | Source `fyear`, copied as recorded. All audited observations have a value. This is application year, not grant year. |
| `in_paper_sample` | Boolean | `1` or `0` | True/1 exactly when recorded `application_year` is between 1981 and 2024 inclusive. False/0 otherwise. All application years are nonmissing in this release. The audited source has 7,980,202 true values and 446,750 false values. |
| `prob_commercialization` | float32 | Decimal probability | Predicted probability that the invention is commercialized through in-house use, licensing, or both. Source outcome name: `use_strict`. |
| `prob_inhouse` | float32 | Decimal probability | Predicted probability that the applicant/owner has used the patented invention in a product, process, or service that has been commercialized. Source outcome name: `inhouse`. |
| `prob_licensing` | float32 | Decimal probability | Predicted probability that a patent holder has licensed the patent to an independent party. Source outcome name: `license`. |

All probabilities lie in [0, 1], not [0, 100]. Each has zero missing and zero nonfinite values in the audited source. Their definitions follow the harmonized inventor-survey outcomes in Section 2 of *What Share of Patents Is Commercialized?* They are model predictions, not observed survey answers and not a fixed-horizon forecast.

## Source mapping and precision

The source master is `patent_title_abstract_pv20250930_cpc_renew_kpss_govinst_tri_vpm_prob_section.parquet`. The probability fields are renamed as follows, with no change to their values:

| Release field | Master source field |
| --- | --- |
| `prob_commercialization` | `p_use_strict__abstract__RF__paecter` |
| `prob_inhouse` | `p_inhouse__abstract__RF__paecter` |
| `prob_licensing` | `p_license__abstract__RF__paecter` |

Each master field matches the `p_raw` field in its corresponding `{label}__abstract__RF__paecter__section_score_8m.parquet` file exactly by patent ID. The corresponding labels are `use_strict`, `inhouse`, and `license`.

The section scorer uses the positive-class output from the final Random Forest models, with separate models for CPC sections A, B, C, F, G, H, and modified Y. Original sections D and E use the Y model. Inputs are 1,024-dimensional PaECTER embeddings of title-and-abstract text. The scorer applies no isotonic calibration. The values in this release are section-specific predictions, not pooled-model predictions.

Parquet preserves the original float32 probability values. CSV writes 17 significant digits after exact promotion to float64. Reading those decimal values and converting to float32 recovers the original source values. No new statistical precision is introduced by the longer decimal representation.

## Coverage and recorded years

The full release contains every patent scored in the audited master. It includes 444,539 observations with recorded application year before 1981 and 2,211 with recorded application year 2025. These observations have `in_paper_sample = false`.

Recorded application years range from 1682 to 2025. The source has anomalous historical years and 108 cases where the application date is after the grant date. The release preserves `fyear` as recorded; it neither repairs the dates nor removes these cases. Application and grant dates themselves are not included. The paper-sample flag filters years only and is not a date-quality flag.

Within this scored universe, the flag identifies the paper's full U.S. patent population with recorded application years 1981-2024. It does not identify the 2,088,206 triadic patents within that window, survey-training observations, or the subsets used in renewal and citation validation. Those indicators and analysis covariates are outside the six-column release. The scored population includes training patents and should not be described as entirely held out.

## Interpreting the three probabilities

Overall commercialization is defined as the union of in-house use and licensing. The two uses can overlap. However, the three probabilities come from separately estimated models and are not constrained jointly. Therefore, they need not sum to one, and a predicted component can exceed predicted overall commercialization for an individual patent.

Do not treat the columns as a three-category probability distribution or mechanically normalize them. The expression `prob_inhouse + prob_licensing - prob_commercialization` is derived from separate predictions; it is not an observed joint-use label or a jointly constrained patent-level probability.

## Audited reference values

| Check | Audited value |
| --- | ---: |
| All scored rows | 8,426,952 |
| Unique patent IDs | 8,426,952 |
| Missing or duplicate patent IDs | 0 |
| Paper-window rows | 7,980,202 |
| Missing values in each probability | 0 |
| Out-of-range or nonfinite values in each probability | 0 |
| Mean `prob_commercialization`, paper window | 0.6365106735544511 |
| Mean `prob_inhouse`, paper window | 0.5767027126834042 |
| Mean `prob_licensing`, paper window | 0.15705905760144348 |

The reference means use the saved source values with double-precision aggregation. Minor displayed differences can result from using a float32 accumulator. The [release manifest](release_manifest.json) records exact Parquet and CSV verification, source provenance, environment versions, and file hashes. The [checksum file](SHA256SUMS.txt) also covers the manifest. Source verification was completed on 23 October 2026.

## Preview

`sample_patent_commercialization_probabilities.csv` contains 1,000 evenly spaced observations in numeric patent-ID order, using the same six fields and CSV encoding. It is a format preview, not a random sample and not a substitute for the full release in analysis.

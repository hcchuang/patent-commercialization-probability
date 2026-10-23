# Patent commercialization probabilities

Data accompanying **What Share of Patents Is Commercialized?** by Hui-Ching Chuang, Po-Hsuan Hsu, You-Na Lee, and John P. Walsh.

Each row provides a U.S. patent grant ID, recorded application year, paper-sample flag, and predicted probabilities of commercialization, in-house use, and licensing.

## Download

Version: [v1](https://github.com/hcchuang/patent-commercialization-probability/releases/tag/v1). Download either format; both contain the same 8,426,952 observations.

| File | Size |
| --- | ---: |
| [patent_commercialization_probabilities.parquet](https://github.com/hcchuang/patent-commercialization-probability/releases/download/v1/patent_commercialization_probabilities.parquet) | 37,253,054 bytes |
| [patent_commercialization_probabilities.csv.gz](https://github.com/hcchuang/patent-commercialization-probability/releases/download/v1/patent_commercialization_probabilities.csv.gz) | 102,804,038 bytes |

See the [data dictionary](DATA_DICTIONARY.md), [manifest](release_manifest.json), and [SHA-256 checksums](SHA256SUMS.txt). The [1,000-row preview](sample_patent_commercialization_probabilities.csv) selects evenly spaced rows in numeric patent-ID order; it is not a random sample.

## Coverage and interpretation

Select `in_paper_sample` for the paper's 7,980,202 patents with recorded application years 1981-2024. The remaining 446,750 scored patents are outside that window.

Application year is copied as recorded. Source years range from 1682 to 2025 and include anomalous historical values; 108 source records have an application date after the grant date. No dates are corrected. The paper flag filters years only.

Predictions use separate Random Forest models for each outcome and CPC section, with PaECTER title-and-abstract embeddings. Seven model sections are A, B, C, F, G, H, and modified Y; D and E use Y. These are raw probabilities without isotonic calibration. Original float32 values are preserved; CSV uses 17 significant digits after exact promotion to float64.

Commercialization means in-house use or licensing, including both. These outcomes overlap, and separately estimated probabilities need not sum to one or satisfy joint-probability constraints. A predicted component can exceed predicted commercialization. The release contains predictions, not observed survey responses, patent text, embeddings, or fitted models.

## Python

With pandas and pyarrow installed:

```python
import pandas as pd
p = pd.read_parquet("patent_commercialization_probabilities.parquet", engine="pyarrow")
paper = p.loc[p["in_paper_sample"]]
```

For compressed CSV, retain string IDs and exact decimal values:

```python
p = pd.read_csv("patent_commercialization_probabilities.csv.gz",
                dtype={"patent_id": "string"}, float_precision="round_trip")
paper = p.loc[p["in_paper_sample"].eq(1)]
```

Merge another table using the same string grant IDs:

```python
other = pd.read_csv("my_patents.csv", dtype={"patent_id": "string"})
joined = other.merge(p, on="patent_id", how="left",
                     validate="many_to_one", indicator=True)
print(joined["_merge"].value_counts())
```

Application numbers and patent-family IDs are different keys. See pandas [CSV documentation](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html).

## Stata

Extract the gzip download to `patent_commercialization_probabilities.csv` with a gzip-capable program, then:

```stata
import delimited using "patent_commercialization_probabilities.csv", clear varnames(1) ///
    stringcols(1) asdouble
isid patent_id
save "patent_commercialization_probabilities.dta"
* Optional paper window: keep if in_paper_sample == 1
```

For another dataset with string grant IDs, use `merge m:1 patent_id using "patent_commercialization_probabilities.dta"` and inspect `_merge`. Use `1:1` when both tables have unique patent IDs. See Stata's [import documentation](https://www.stata.com/manuals/dimportdelimited.pdf).

## Cite the paper

Please cite the manuscript and dataset version:

Chuang, Hui-Ching, Po-Hsuan Hsu, You-Na Lee, and John P. Walsh. *What Share of Patents Is Commercialized?*.

The same authors. 2026. *Patent commercialization probabilities*. Version v1. [Repository](https://github.com/hcchuang/patent-commercialization-probability). See [CITATION.cff](CITATION.cff) for metadata.

## License

Data and documentation are licensed under [CC BY 4.0](LICENSE.md).

A ready-to-use BibTeX entry is provided in [CITATION.bib](CITATION.bib).

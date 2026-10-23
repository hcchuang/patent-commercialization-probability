# Patent commercialization probabilities

Data accompanying **What Share of Patents Is Commercialized?** by Hui-Ching Chuang, Po-Hsuan Hsu, You-Na Lee, and John P. Walsh.

Each row provides a U.S. patent grant ID, recorded application year, paper-sample flag, and predicted probabilities of commercialization, in-house use, and licensing.

## Download

Version: [v1](https://github.com/hcchuang/patent-commercialization-probability/releases/tag/v1). Download either format; both contain the same 8,426,952 observations.

| File | Size |
| --- | ---: |
| [patent_commercialization_probabilities.parquet](https://github.com/hcchuang/patent-commercialization-probability/releases/download/v1/patent_commercialization_probabilities.parquet) | 37,253,054 bytes |
| [patent_commercialization_probabilities.csv.gz](https://github.com/hcchuang/patent-commercialization-probability/releases/download/v1/patent_commercialization_probabilities.csv.gz) | 76,699,440 bytes |

See the [data dictionary](DATA_DICTIONARY.md), [manifest](release_manifest.json), and [SHA-256 checksums](SHA256SUMS.txt). The [1,000-row preview](sample_patent_commercialization_probabilities.csv) selects evenly spaced rows in numeric patent-ID order; it is not a random sample.

## Coverage and interpretation

Select `in_paper_sample` for the paper's 7,980,202 patents with recorded application years 1981-2024. The remaining 446,750 scored patents are outside that window.

Predictions use separate Random Forest models for each outcome and CPC section, with PaECTER title-and-abstract embeddings. Seven model sections are A, B, C, F, G, H, and modified Y; D and E use Y. 

Commercialization means in-house use or licensing, including both. These outcomes overlap, and separately estimated probabilities need not sum to one or satisfy joint-probability constraints. A predicted component can exceed predicted commercialization. The release contains predictions, not observed survey responses, patent text, embeddings, or fitted models.

## Python

With pandas and pyarrow installed:

```python
import pandas as pd
p = pd.read_parquet("patent_commercialization_probabilities.parquet", engine="pyarrow")
paper = p.loc[p["in_paper_sample"]]
```

For compressed CSV, the following reproduces the full-population, section-by-section “All” rows in Tables 6, A2, and A3 using the original float32 summary calculation.

Tested with pandas 2.2.3, NumPy 2.3.5, and Bottleneck 1.4.2. Bottleneck must be installed; pandas uses it automatically without an explicit import. This reproduces the original numerical calculation; double-precision summaries (including Stata) give different standard deviations.

```python
import pandas as pd

prob_cols = [
    "prob_commercialization",
    "prob_inhouse",
    "prob_licensing",
]

p = pd.read_csv(
    "patent_commercialization_probabilities.csv.gz",
    dtype={"patent_id": "string", **{c: "float32" for c in prob_cols}},
    float_precision="round_trip",
)
paper = p.loc[p["in_paper_sample"].eq(1)]

# Reproduce the original pandas float32 summary calculation.
pd.set_option("compute.use_bottleneck", True)

summary = pd.DataFrame({
    "Count": paper[prob_cols].count(),
    "Mean": paper[prob_cols].mean(),
    "Std. Dev.": paper[prob_cols].std(),
    "Min": paper[prob_cols].min(),
    "Pct25": paper[prob_cols].quantile(0.25),
    "Median": paper[prob_cols].median(),
    "Pct75": paper[prob_cols].quantile(0.75),
    "Max": paper[prob_cols].max(),
})

formatted = summary.copy()
formatted["Count"] = summary["Count"].map("{:,}".format)
for col in summary.columns.drop("Count"):
    formatted[col] = summary[col].map(lambda x: f"{100 * x:.2f}%")

print(formatted.to_string())
```

Expected output:

```text
                            Count    Mean Std. Dev.    Min   Pct25  Median   Pct75      Max
prob_commercialization  7,980,202  63.65%    10.38%  0.00%  58.40%  64.60%  70.40%  100.00%
prob_inhouse            7,980,202  57.67%    11.58%  0.00%  51.90%  58.60%  65.10%  100.00%
prob_licensing          7,980,202  15.71%     7.48%  0.00%  10.20%  14.70%  20.10%  100.00%
```

Merge another table using the same string grant IDs:

```python
other = pd.read_csv("my_patents.csv", dtype={"patent_id": "string"})
joined = other.merge(p, on="patent_id", how="left",
                     validate="many_to_one", indicator=True)
print(joined["_merge"].value_counts())
```


## Stata

Extract the gzip download to `patent_commercialization_probabilities.csv` with a gzip-capable program, then:

```stata
import delimited using "patent_commercialization_probabilities.csv", clear varnames(1) ///
    stringcols(1) asdouble
isid patent_id
save "patent_commercialization_probabilities.dta"
* Optional paper window: keep if in_paper_sample == 1
```

For another dataset with string grant IDs, use `merge m:1 patent_id using "patent_commercialization_probabilities.dta"` and inspect `_merge`. Use `1:1` when both tables have unique patent IDs. 

## Cite the paper

Please cite our paper:

Chuang, Hui-Ching, Po-Hsuan Hsu, You-Na Lee, and John P. Walsh. *What Share of Patents Is Commercialized?*.

## License

Data and documentation are licensed under [CC BY 4.0](LICENSE.md).

A ready-to-use BibTeX entry is provided in [CITATION.bib](CITATION.bib).

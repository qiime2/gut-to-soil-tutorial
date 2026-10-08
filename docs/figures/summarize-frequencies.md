:::{figure} https://raw.githubusercontent.com/qiime2/gut-to-soil-tutorial/9599fc05f261d0e4fad03eb0eee621ad9cecb1d5/docs/figures/summarize-frequencies.svg
:label: fig-summarize-frequencies
:alt: Per-sample and per-feature summaries of the feature table

**Per-sample and per-feature summaries of the feature table.** Sample frequency summaries (panel **a**) and feature frequency summaries (panel **b**) are generated as output of `qiime feature-table summarize`. Notice that these are formatted a lot like metadata. This is not a coincidence, but rather these artifacts are of artifact class `ImmutableMetadata`, which is the class for metadata that is stored in Artifacts. These can be provided as-is as input anywhere that metadata is used, and they can be exported to `.tsv` files if needed by calling `qiime tools export`. By design, `rachis` commands only generate *Results* as output so data provenance can be embedded in all outputs. The `ImmutableMetadata` artifact class is used by `rachis` commands to output metadata as needed.

Panel **a** represents *sample metadata*, while panel **b** represents *feature metadata*.
:::

:::{figure} https://raw.githubusercontent.com/qiime2/gut-to-soil-tutorial/9599fc05f261d0e4fad03eb0eee621ad9cecb1d5/docs/figures/even-sampling.svg
:label: fig-even-sampling
:alt: Rarefying feature tables to achieve an even sampling depth across all samples

**Rarefying feature tables to achieve an even sampling depth across all samples.** Samples from a single sequencing run often differ in **sequencing depth** (panels **a** and **b**). In general, this doesn't represent the biology of the samples but rather is an artifact of the sequencing. Many of the diversity metrics that are applied to feature tables are sensitive to differences in sequencing depth, so this must be normalized. Rarefying a feature table is the most common way of achieving this normalization, and involves selecting an **even sampling depth** and then subsampling counts at random from all samples up to that depth without replacement. Samples with sequencing depth less than the even sampling depth are excluded from the resulting rarefied feature table. Panels **c** and **d** illustrate even sampling to 1800 sequences per sample. Because sample `5f08` had only 640 sequences, it is excluded from the rarefied table in panel **c**. Panels **e** and **f** illustrate even sampling to 8000 sequences per sample. All samples with fewer than 8000 sequences are excluded from the rarefied table in panel **e**.

As you can see, selecting an even sampling depth forces you to decide between excluding samples and excluding sequences. It is generally considered a "necessary evil". Subsampling one time, as illustrated here, is rarefying. The more robust approach, rarefaction, rarefies multiple times and averages the resulting metrics.

The diversity calculations that follow will work from the depth-1800 table in panel **c**.
:::

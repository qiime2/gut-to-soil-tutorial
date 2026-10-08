:::{figure} https://raw.githubusercontent.com/qiime2/gut-to-soil-tutorial/9599fc05f261d0e4fad03eb0eee621ad9cecb1d5/docs/figures/kmer-features.svg
:label: fig-kmer-features
:alt: Kmerization of ASV sequences and a feature table

**Kmerization of ASV sequences and a feature table.** An alternative approach for relatedness-based diversity metrics splits sequences into their constituent kmers (panel **a**), and then expands the feature table from ASVs to kmers (panel **b**). The count of a kmer in a sample is the summed count of every ASV whose sequence contains it. More closely related ASVs share more kmers with each other, and as a result computing identity-based diversity metrics on this kmer table results in diversity values that implicitly integrate feature relatedness.
:::

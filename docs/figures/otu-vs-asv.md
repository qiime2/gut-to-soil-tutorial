:::{figure} figures/otu-vs-asv.svg
:label: fig-otu-vs-asv
:alt: Clustering ASVs into OTUs

**Clustering ASVs into OTUs.** Our ASV-based feature table (panel **a**) could be clustered into an OTU-based feature table (panel **b**), based on the similarity of the ASV sequences ([](#fig-feature-table)b). In our constructed data, if our ASV sequences are clustered at 90% identity using `qiime vsearch cluster-features-de-novo`, ASVs 1–3 collapse into OTU1, and ASVs 9–10 collapse into OTU7. During that collapsing, the ASV counts are summed such that the OTU count is equal to the sum of the counts of the ASVs it contains. The difference in these tables illustrates how OTU clustering reduces the resolution of our feature table. In general, we don't recommend OTU clustering workflows for this reason.
:::

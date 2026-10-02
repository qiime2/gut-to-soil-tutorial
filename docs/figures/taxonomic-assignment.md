:::{figure} figures/taxonomic-assignment.svg
:label: fig-taxonomic-assignment
:alt: Assigning taxonomy to observed sequences

**Assigning taxonomy to observed sequences.** Your `FeatureData[Sequence]` artifact ([](#fig-feature-table)b) can be provided to a `TaxonomicClassifier` ([](#fig-reference-database)c) to generate taxonomic annotations in a `FeatureData[Taxonomy]` artifact. Each ASV sequence is run through the `TaxonomicClassifier`, and the most specific assignment that meets or exceeds the confidence threshold is assigned to the ASV. Some ASVs (e.g., ASV1 or ASV4) may only have acceptable assignment confidence at higher levels (genus and domain, respectively), while others (e.g., ASV2 and ASV3) may be assigned to the species level.

The ASV shading from [](#fig-otu-vs-asv) is reintroduced here to illustrate that clustering into OTUs can mask true biological diversity present in the samples.
:::

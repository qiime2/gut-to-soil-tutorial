:::{figure} figures/faith-pd.svg
:label: fig-faith-pd
:alt: Alpha diversity with Faith's Phylogenetic Diversity

**Alpha diversity with Faith's Phylogenetic Diversity.** Instead of counting a sample's features, Faith's phylogenetic diversity (PD) sums the lengths of the branches that are represented in a sample. This is done by identifying which ASVs (tips in the tree) are observed in a sample, tracing each of those to the root of the tree (panel **a**), and then summing those "observed branch lengths" (panel **b**). Those values are represented in a `SampleData[AlphaDiversity]` artifact (panel **c**), like the identity-based metrics presented in [](#fig-alpha-diversity).
:::

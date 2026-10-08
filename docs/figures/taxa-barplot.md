:::{figure} https://raw.githubusercontent.com/qiime2/gut-to-soil-tutorial/9599fc05f261d0e4fad03eb0eee621ad9cecb1d5/docs/figures/taxa-barplot.svg
:label: fig-taxa-barplot
:alt: Taxonomic composition of samples as stacked bar plots

**Taxonomic composition of samples as stacked bar plots.** Taxonomic composition barplots are a common summary visualization of samples. These represent the feature table as relative abundances (panel **a**), with ASVs grouped based on their taxonomic labels (e.g., [](#fig-taxonomic-assignment)). Taxonomic groupings can be presented at different focal levels, such as phylum (panel **b**) or genus (panel **c**). If an ASV doesn't have an assignment at the focal level (e.g., ASV4 for the genus plot), it will generally be annotated at the lowest level it was assigned (hence `d__Bacteria` in the genus-level plot).

In the constructed data, we can see from this plot that some taxa (e.g., *Blautia-A*) decrease in abundance during the composting process, while others (e.g., *Streptomyces*) increase in abundance. The choice of taxonomic level changes what is visible: at phylum level the transition is less apparent, in part because Bacillota includes the gut-associated *Blautia-A* and the compost-associated *Lysinibacillus* and *Planifilum*.
:::

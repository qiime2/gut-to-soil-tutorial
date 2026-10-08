:::{figure} https://raw.githubusercontent.com/qiime2/gut-to-soil-tutorial/9599fc05f261d0e4fad03eb0eee621ad9cecb1d5/docs/figures/reference-database.svg
:label: fig-reference-database
:alt: A reference database is used to train a taxonomic classifier

**A reference database is used to train a taxonomic classifier.** Machine-learning-based taxonomic classification in QIIME 2 requires two inputs: reference sequences in a `FeatureData[Sequence]` artifact (panel **a**), and taxonomic labels for the reference sequences in a `FeatureData[Taxonomy]` artifact (panel **b**). The RESCRIPt plugin can help you obtain these artifacts and prepare them for training a classifier. These are used to build a `TaxonomicClassifier` artifact (panel **c**), which is different from any artifacts that we've worked with to this point: rather than containing static data, it contains executable software, and that is indicated by the gear icon here.

In this illustration, the taxonomic labels are obtained from Genome Taxonomy Database (GTDB) version 232.0, while in the tutorial we work with GTDB version 202.0. GTDB version 202.0 is an older database version that is helpful for allowing the tutorial to be run on diverse computer hardware; GTDB version 232.0 is current as of this writing. The reference artifacts and the pre-trained classifier for GTDB version 232.0 are distributed at https://doi.org/10.5281/zenodo.21619532.
:::

:::{figure} figures/fastq-manifest.svg
:label: fig-fastq-manifest
:alt: Schematic view of a fastq manifest file and demultiplexed fastq data, ready for import into QIIME 2

**Schematic view of a fastq manifest file and demultiplexed fastq data, ready for import into QIIME 2.** In general, demultiplexed Illumina sequencing data is stored in a pair of files per sample corresponding to the forward and reverse reads. A fastq manifest file (panel **a**) is a sample metadata file with three specific columns that link specific samples to the absolute file paths where their forward and reverse reads can be found. Fastq files for one sample are illustrated here: panels **b** and **c** are the forward and reverse reads for sample `7b2e`. It is common for the sample ids to be embedded in the fastq file names (as is illustrated here) but that is not a requirement for QIIME 2. Rather, the file names can be anything, and the manifest file links the files to the samples.
:::

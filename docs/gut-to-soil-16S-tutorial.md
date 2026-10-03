(gut-to-soil-16S-tutorial)=
# Gut-to-soil microbiome axis 16S rRNA analysis tutorial 💩🌱

The gut-to-soil tutorial is intended to be the primary entry point for new users who are interested in working through a full analysis of microbiome data with QIIME 2, before applying it to their own data.
As we progress, you can run the commands from the tutorial on your own computer and you will be presented with exercises and solutions to encourage you to explore and interpret the results of the analysis steps.

This tutorial assumes two things:
 * First, that you've read [Getting Started with QIIME 2](https://amplicon-docs.qiime2.org/en/stable/explanations/getting-started/). This will help you understand some of the informatics-y jargon used here. Also, see [the project glossary](https://news.rachis.org/en/latest/glossary/) for help with our jargon.
 * And second, that you have a working installation of QIIME 2 (learn how to install QIIME 2 at https://install.qiime2.org). If you'd rather not install QIIME 2 before reading this document, all outputs that will be generated are linked from this document - so, the QIIME 2 installation is optional, but the tutorial is written assuming that you're running each command (including those in the Exercises) as they are presented.

(gut-to-soil-tutorial:data)=
::::{dropdown} The data used in this tutorial

(gut-to-soil-tutorial:original-data)=
### The original study data

The data used here was originally generated for [Meilander et al. (2025): Upcycling Human Excrement: The Gut Microbiome to Soil Microbiome Axis](doi:10.1093/ismeco/ycaf089), which profiles 15 biological replicates of mesophilic human excrement composting (HEC).
The full dataset is available in a Zenodo archive [](doi:10.5281/zenodo.13887456) for continued exploration by learners.

This is 16S rRNA data generated using the Earth Microbiome Project protocol [](doi:10.1038/ismej.2012.8).
Specifically, the hypervariable region 4 (V4) of the 16S rRNA gene was amplified using the F515-R806 primers—a broad-coverage primer pair for Bacteria that also amplifies some Archaea.
Paired-end sequencing was performed on an Illumina MiSeq using the 2 ✕ 250 base pair chemistry.
Full details are presented in the original publication.

Samples used to generate this dataset were collected from 15 biological replicates over a 52-week composting period from 11 human participants (note that two subjects each participated three times over the course of the experiment).
Each biological replicate was maintained in a separate bucket, and we therefore refer to the replicates as buckets in the original manuscript and here.
Three HE samples were collected from each participant using sterile cotton swabs.
Bulking material samples (1–2 per bucket, depending on the bulking materials used) were collected before composting.
Environmental samples included swabs of the area surrounding each composting toilet, the interior surface of each toilet chamber prior to use, and the interior surface of each composting bucket before material was added.
Weekly samples were then collected from each bucket for 52 weeks (composting timepoints 1–52).
Food and landscape waste compost (FLWC) samples were collected from a composting facility at Northern Arizona University, six small agricultural farms, and two backyard gardens in Flagstaff, Arizona.
These samples represented compost produced from pre- and post-consumer food waste co-composted with landscape materials, including leaves, pine needles, branches, and straw.
In addition, sequencing data from 41 soil samples from the Earth Microbiome Project (EMP) "EMP500" were obtained from publicly available Qiita data (study 13114) and used in a pooled analysis (i.e., not resequenced for this project).
Together the FLWC and soil samples served as reference samples.

(gut-to-soil-tutorial:tutorial-data)=
### The data subset for the hands-on tutorial

The data used here is a subset (a single sequencing run) of the [](doi:10.1093/ismeco/ycaf089) data, specifically selected so that this tutorial can be run quickly on a personal computer.
This subset contains 104 samples with data from 13 buckets and includes 25 HEC samples, 19 HE samples, 20 FLWC samples, 15 Bulking Material samples, 10 samples from inside toilets pre-use, 10 soil samples collected near the composting toilets, and various additional control samples.
This was a test sequencing run performed while sample collection was ongoing, and was intended to cover the different sample types that had been collected (e.g., to detect issues with extraction or sequencing protocols).
As such, most buckets don't have dense temporal information but Bucket 5 is the exception with 14 samples over the first 18 weeks of composting.
We therefore primarily focus on the cross-sectional aspects of data analysis in this tutorial, though you can visualize the time series in Bucket 5 with this subset; future longitudinal tutorials will be developed on different subsets of this study data and from metagenomic and metatranscriptomic data generated in ongoing HEC projects.
No EMP500 soils are used in this tutorial, so the FLWC (referred to as "Food Compost" in the study metadata) samples will serve as the reference samples here.

(gut-to-soil-tutorial:constructed-data)=
### Constructed data for figures

In addition to the tutorial data, beginning with [](#fig-sample-metadata-study) we'll follow a constructed example dataset through the analysis to enable visualization of microbiome data as it progresses through the workflow that we're illustrating.
This constructed dataset does not represent real samples, but it is inspired by the tutorial data.
Specifically it models seven samples: one HE sample, five HEC samples from five different timepoints over one year of composting from a single bucket, and one food compost sample.
This constructed dataset is intended to provide a way for readers to visualize and assess their understanding of the analysis process as we progress through the tutorial in a way that would be impossible even on the relatively simple tutorial data.

::::

::::{margin}
:::{tip} Why study human excrement composting?
The data used in this tutorial [originates from](#gut-to-soil-tutorial:original-data) a study of microbial succession during human excrement composting (HEC).
Get our take on why this is interesting [here](https://gut-to-soil-tutorial.readthedocs.io/en/latest/why/).
:::
::::

## Installing QIIME 2

Install QIIME 2 by following the [installation documentation](https://install.qiime2.org).
The screenshots presented in this section were taken against the JupyterLab environment in the QIIME 2 2026.7 workshop Docker container.
You can use that, or any way of installing QIIME 2 that is covered in the documentation.

After loading your QIIME 2 environment - either by calling `conda activate ...`, through Docker, or through any other supported approach, in a terminal, type `qiime info`, and press return.
The result should look like that in the command terminal of [](#fig-qiime-info).
(Note that the file browser side bar will only be there if you're working in JupyterLab.)
If you received errors with these steps, we recommend looking for similar posts on the [QIIME 2 Forum](https://forum.qiime2.org), and posting yourself if you don't find relevant troubleshooting information.

:::{include} figures/qiime-info.md
:::

(gut-to-soil-tutorial:sample-metadata)=
## Reviewing sample metadata and running a QIIME 2 command

Before starting the analysis, we'll explore the sample metadata to get familiarized with [the samples used in this study](#gut-to-soil-tutorial:tutorial-data).
Sample metadata generally contains per-sample information relevant to your study.
In this tutorial, the sample metadata includes information such as the sample type (see the `SampleType` column), the sample's pH at time of collection (see the `Compost pH` column), which of our biological replicates it came from (see the `Bucket` column), which composting timepoint the sample came from (if relevant, see the `Composting Time Point` column), and more.
To learn about metadata in QIIME 2, including how it should be formatted, refer to our documentation on the [metadata format](https://use.rachis.org/en/latest/references/metadata.html).

::::{margin}
:::{tip}
To learn more about metadata standards, you can refer to [Chloe Herman's video on this topic](https://www.youtube.com/watch?v=erklD1bofzE), which was developed in collaboration with the [National Microbiome Data Collaborative (NMDC)](https://microbiomedata.org/).
:::
::::

First, create a directory to work on this tutorial in and then change into that directory.

```shell
mkdir gut-to-soil/
cd gut-to-soil/
```

Run the following command to download the sample metadata as tab-separated text from Zenodo and save it in the file `sample-metadata.tsv`.
This `sample-metadata.tsv` file is used throughout the rest of the tutorial.

:::{describe-usage}
:scope: gut-to-soil

sample_metadata = use.init_metadata_from_url(
   'sample-metadata',
   'https://zenodo.org/records/15390940/files/gut-to-soil-tutorial-sample-metadata.tsv?download=1')
:::

QIIME 2's [q2-metadata plugin](xref:rachis-library-target#q2-plugin-metadata) provides a Visualizer called [`tabulate`](xref:rachis-library-target#q2-action-metadata-tabulate) that generates a convenient view of a sample metadata file.
Run the following command, which is the first QIIME 2 command that you should run in this tutorial:

(sample-metadata-tabulate-viz)=
:::{describe-usage}
use.action(
  use.UsageAction(plugin_id='metadata',
                  action_id='tabulate'),
  use.UsageInputs(input=sample_metadata),
  use.UsageOutputNames(visualization='sample_metadata')
)
:::

This will generate a QIIME 2 [Visualization](xref:rachis-glossary-target#term-visualization).
[](#fig-metadata-tabulate) shows this Visualization viewed inside the QIIME 2 2026.7 workshop Docker container.
Visualizations can also be viewed by loading them with [`rachis-view`](https://view.rachis.org) - navigate to `rachis-view` in a web browser, and drag and drop the `sample-metadata.qzv` file.

:::{include} figures/metadata-tabulate.md
:::

::::{margin}
:::{tip}
You can learn more about viewing Visualizations, including alternatives to `rachis-view` if needed, [in the QIIME 2 documentation](https://use.rachis.org/en/latest/how-to-guides/view-visualizations.html).
:::
::::

[](#fig-sample-metadata-study) presents a view of sample metadata for the [constructed dataset](#gut-to-soil-tutorial:constructed-data) described above.
These samples are not real, but rather represent a few samples that we'll use in the figures throughout the tutorial to assist readers with visualizing the data we are working with.
In your interactive tabulated view of the metadata, explore the metadata columns that are presented in [](#fig-sample-metadata-study).

:::{include} figures/sample-metadata-study.md
:::

## Access already-imported QIIME 2 data

This tutorial begins with paired-end read sequencing data generated on an Illumina MiSeq that has already been demultiplexed and imported into a QIIME 2 [Artifact](xref:rachis-glossary-target#term-artifact).
You can access this data by running the following command:

:::{describe-usage}
:scope: gut-to-soil

demux = use.init_artifact_from_url(
   'demux',
   'https://zenodo.org/records/15390940/files/gut-to-soil-tutorial-nano2-demux-10p.qza?download=1')
:::

Demultiplexed sequence data is data where sequences have already been assigned to the sample that they were observed in.
Paired-end read data means that for every sequence read in your dataset you have a forward read (which typically starts from the 5' end of the target amplicon sequence) and a reverse read (which typically starts from the 3' end of the targeted amplicon).
In demultiplexed paired-end read data, you'll generally start with two fastq files per sample, one for the forward read, and one for the reverse read.
For QIIME 2 to work with these data, it needs to know which of your fastq files contains the forward reads and which contains the reverse reads for each sample.
For importing these data into QIIME 2, we generally recommend the use of a fastq manifest file which maps the sample identifier to the forward and reverse read file paths.
[](#fig-fastq-manifest) presents an illustration of a fastq manifest file alongside two example fastq files.
Like in [](#fig-sample-metadata-study), these are not real data but rather the [constructed dataset](#gut-to-soil-tutorial:constructed-data) generated for illustrative purposes.

:::{include} figures/fastq-manifest.md
:::

Notice the similarity of the fastq manifest in [](#fig-fastq-manifest) and the sample metadata in [](#fig-sample-metadata-study).
A fastq manifest is a specific type of sample metadata file.
Specifically, it's one that contains `forward-absolute-filepath` and `reverse-absolute-filepath` columns, with absolute file paths present for every sample identifier in the table.

Because sequence data can be delivered to you in many different forms, it's not possible to cover the varieties here.
Instead, we refer you to [How to import data for use with QIIME 2](https://amplicon-docs.qiime2.org/en/latest/how-to-guides/how-to-import.html) to learn how to import your data.
If you want to learn why importing is necessary, refer to [Why importing is necessary](https://amplicon-docs.qiime2.org/en/latest/explanations/why-importing.html).

To simplify processing of your data, we recommend that you ask your sequencing center to provide data already demultiplexed.
This should be something they can easily do, and it simplifies processing as you don't need to know how data was barcoded for multiplexed sequencing.
You can also ask them to provide a fastq manifest file for your sequencing data, which should be easy for them to generate based on the [file specification](https://amplicon-docs.qiime2.org/en/latest/how-to-guides/how-to-import.html#import-fastq-manifest); if they're not able to do this, you can use a tool like [fq-manifestor](https://forum.qiime2.org/t/fq-manifestor-a-web-app-for-generating-fastq-manifest-files/34114) to generate one.

## Summarize demultiplexed sequences

When you have demultiplexed sequence data, the next step is typically to generate a visual summary of it.
This allows you to determine how many sequences were obtained per sample, and also to get a summary of the distribution of sequence quality at each position in your sequence data.
Run the following command to generate an interactive quality plot Visualization.
Load the visualization (for example, by using `rachis-view`).

(demux-summary-viz)=
:::{describe-usage}

use.action(
    use.UsageAction(plugin_id='demux',
                    action_id='summarize'),
    use.UsageInputs(data=demux),
    use.UsageOutputNames(visualization='demux'))
:::

:::{exercise} Exploring the demultiplexed sequence summary.
:label: demux-summary
How many samples are represented in this sequencing data?
What is the median number of sequence reads obtained per sample?
What is the median quality score at position 200 of the forward reads?
:::

:::{solution} demux-summary
:class: dropdown
There are 104 samples represented in this sequencing data, and the median sequencing depth (number of sequences per sample) is 659.5.
The median quality score at position 200 in the forward reads is 37.
:::

### Trimming of PCR primers

The protocol used for sequencing of these data results in sequences that do not include the PCR primers at the beginning and end of the reads.
This is not always the case however: sometimes you will want to remove primers before proceeding to the next step.
If the primers are definitely present and of known length, this can be done with [q2-dada2](xref:rachis-library-target#q2-plugin-dada2) during sequence quality control, which will be covered next; if the primers are not at a known position, of variable length, or may or may not be present (e.g., when running a meta-analysis), you can use QIIME 2's [cutadapt plugin](xref:rachis-library-target#q2-plugin-cutadapt).

## Sequence quality control and feature table construction

QIIME 2 plugins are available for several quality control methods, including DADA2 [](doi:10.1038/nmeth.3869), Deblur [](doi:10.1128/msystems.00191-16), and basic quality-score-based filtering [](doi:10.1038/nmeth.2276).
In this tutorial, we present this step using DADA2, which integrates quality control with the definition of the "features" that will be used to describe our samples.

The primary results of interest from our DADA2 step will be a `FeatureTable[Frequency]` QIIME 2 artifact, which contains counts (frequencies) of each amplicon sequence variant (ASV, or unique sequence post-quality-control) in each sample in the dataset, and a `FeatureData[Sequence]` QIIME 2 artifact, which maps feature identifiers in the `FeatureTable` to the sequences they represent.
Illustrative examples of these two outputs are presented in [](#fig-feature-table).
Notice that the column headers in the `FeatureTable[Frequency]` map directly onto the sequence identifiers in the `FeatureData[Sequence]`.
These are the feature identifiers.
In some cases, feature identifiers can be related across analyses, and in some cases they cannot be.

:::{include} figures/feature-table.md
:::

DADA2 is a pipeline for detecting and correcting (where possible) errors in Illumina amplicon sequence data.
As implemented in the [q2-dada2 plugin](xref:rachis-library-target#q2-plugin-dada2), this quality control process will additionally filter any phiX reads (commonly present in marker gene Illumina sequence data to diversify the base calls at each position) that are identified in the sequencing data, filter chimeric sequences, and merge paired-end reads.

The [`denoise-paired` action](xref:rachis-library-target#q2-action-dada2-denoise-paired), which we'll use here, requires you to provide values for four parameters that are used in quality filtering:
- `trim-left-f <a>`, which trims off the first `<a>` bases of each forward read
- `trim-left-r <b>`, which trims off the first `<b>` bases of each reverse read
- `trunc-len-f <c>`, which truncates each forward read at position `<c>`
- `trunc-len-r <d>`, which truncates each reverse read at position `<d>`

These parameters allow the user to remove low-quality regions of the sequences from the beginning and end of the sequences.
To determine what values to pass for these parameters, review the *Interactive Quality Plot* tab in the [`demux.qzv`](#demux-summary-viz) file that was generated above.

:::{exercise} Choosing "trim" and "trunc" parameter values.
:label: dada2-trim-trunc
Based on the plots you see in the [`demux.qzv`](#demux-summary-viz) file that was generated above, what values would you choose for `trim-left-f` and `trunc-len-f` in this case?
What about `trim-left-r` and `trunc-len-r`?
:::

:::{solution} dada2-trim-trunc
:class: dropdown
The answer to this question is subjective, and something you'll need to determine on your own.
When I review this, I notice that the quality of the initial bases seems to be high, so I choose not to trim any bases from the beginning of the sequences.
The quality also seems good all the way out to the end, though maybe dropping off after 250 bases.
I'll therefore truncate at 250.
I'll keep these values the same for both the forward and reverse reads, though that is not a requirement.
These decisions correspond to the following parameter settings:
- `trim-left-f`: 0
- `trim-left-r`: 0
- `trunc-len-f`: 250
- `trunc-len-r`: 250
:::

Now run the following DADA2 command.
This step may take up to 10 minutes to complete—it's the longest-running step in this tutorial.

:::{describe-usage}

asv_seqs, asv_table, denoising_stats, base_transition_stats = use.action(
    use.UsageAction(plugin_id='dada2',
                    action_id='denoise_paired'),
    use.UsageInputs(demultiplexed_seqs=demux,
                    trim_left_f=0,
                    trunc_len_f=250,
                    trim_left_r=0,
                    trunc_len_r=250),
    use.UsageOutputNames(representative_sequences='asv_seqs',
                         table='asv_table',
                         denoising_stats='denoising_stats',
                         base_transition_stats='base_transition_stats'))
:::

One of the outputs created by DADA2, `denoising-stats`, is a summary of the denoising run.
That is generated as an Artifact, so it can't be viewed directly.
However, this is one of many `rachis` types that can be [viewed as Metadata](https://use.rachis.org/en/latest/how-to-guides/artifacts-as-metadata.html)—a very powerful concept that we'll use again later in this tutorial.
Learning to view artifacts as Metadata creates nearly infinite possibilities for how you can explore your microbiome data with QIIME 2.

To do this, we'll again use the [q2-metadata plugin's `tabulate` visualizer](xref:rachis-library-target#q2-action-metadata-tabulate), but this time we'll apply it to the DADA2 statistics.

:::{describe-usage}
stats_as_md = use.view_as_metadata('stats_as_md', denoising_stats)

use.action(
    use.UsageAction(plugin_id='metadata',
                    action_id='tabulate'),
    use.UsageInputs(input=stats_as_md),
    use.UsageOutputNames(visualization='denoising_stats'))
:::

:::{exercise} Exploring the DADA2 denoising statistics.
:label: denoising-stats
Which three samples had the smallest percentage of input reads passing the quality filter?
Refer back to your [tabulated metadata](#sample-metadata-tabulate-viz): what do you know about those samples (e.g., what values do they have in the `SampleType` column)?
(Hint: the `sample-id` is the key that connects data across these two visualizations.)
:::

:::{solution} denoising-stats
:class: dropdown
The samples with identifiers `21f2e6d0`, `9d3aefae`, and `27c41e29` had the lowest percentage of input reads passing the quality filter.
(Sample `48dff3fa` had zero input reads.)
`21f2e6d0` is a human excrement compost (HEC) sample from Bucket 11 and Composting Time Point 28.
`9d3aefae` is a human excrement (HE) sample, also from Bucket 11.
`27c41e29` is a sample of Soil Nearby the Toilet for Bucket 12.
`48dff3fa` is a Bulking Material sample for Bucket 7.
:::

:::{exercise} Merging metadata.
:label: merge-metadata
If it's frustrating to switch back and forth between visualizations to answer the question in the last exercise, you can create a combined tabulation of the metadata.
Try to do that by adapting the instructions in [How to merge metadata](https://use.rachis.org/en/latest/how-to-guides/merge-metadata.html).

This is also useful if you want to create a large tabular summary describing your samples following analysis, as you can include as many different metadata objects as you'd like in these summaries.
We'll do this again soon.
:::

::::{solution} merge-metadata
:class: dropdown

:::{describe-usage}
sample_metadata_and_dada2_stats_md = use.merge_metadata('sample_metadata_and_dada2_stats_md', sample_metadata, stats_as_md)

use.action(
    use.UsageAction(plugin_id='metadata',
                    action_id='tabulate'),
    use.UsageInputs(input=sample_metadata_and_dada2_stats_md),
    use.UsageOutputNames(visualization='sample_metadata_w_dada2_stats'))
:::
::::

## Feature table and feature data summaries

After DADA2 completes, you'll want to explore the resulting data.
You can do this using the following two commands, which will create visual summaries of the data.
The [`feature-table summarize` action](xref:rachis-library-target#q2-action-feature-table-summarize) will give you information on how many sequences are associated with each sample and with each feature, histograms of those distributions, and some related summary statistics.

:::{describe-usage}
_, _, asv_frequencies = use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='summarize'),
    use.UsageInputs(table=asv_table,
                    metadata=sample_metadata),
    use.UsageOutputNames(summary='asv_table',
                         sample_frequencies='sample_frequencies',
                         feature_frequencies='asv_frequencies'))
:::

Two additional output artifacts are generated here, and the contents of these are illustrated in [](#fig-summarize-frequencies).
The Sample Frequencies table presents the total sequence count (Frequency) and the number of features with a count greater than zero in each sample.
The Feature Frequencies table presents the total number of sequences observed for each feature, and number of samples that each feature was observed in.
The `summarize` action that we're using here is a [Pipeline](xref:rachis-glossary-target#term-pipeline); recall that Pipelines only generate [Artifacts](xref:rachis-glossary-target#term-artifact) and/or [Visualizations](xref:rachis-glossary-target#term-visualization) as output.
Notice that the structure of the Sample Frequencies table is the same as our sample metadata in [](#fig-sample-metadata-study).
This artifact is in fact a metadata file, but in [Artifact](xref:rachis-glossary-target#term-artifact) form—the semantic type used here is `ImmutableMetadata`, which should be interpreted as metadata inside of an Artifact.
An `ImmutableMetadata` Artifact can be provided anywhere metadata is required in QIIME 2, or can be exported to a `.tsv` file.
The Feature Frequencies table is also an example of an `ImmutableMetadata` Artifact, but in this case it contains feature metadata instead of sample metadata.
The format and use of sample and feature metadata is identical, but you can think of them as describing the entities represented on the opposing axes in the table.
Sample and feature metadata are powerful concepts in the `rachis` ecosystem—understanding these, and how they are used, opens many opportunities for different types of analyses.

:::{include} figures/summarize-frequencies.md
:::

:::{exercise} Exploring the feature table summary (part 1).
:label: feature-table-summary-1
What is the total number of sequences represented in the feature table?
What is the identifier of the feature that is observed the most (i.e., has the highest frequency)?
:::

:::{solution} feature-table-summary-1
:class: dropdown
There are 39,949 total sequences represented in this feature table.
The identifier of the feature observed the largest number of times is `c6c3ab4e828fb40d6e05967b7aac9338`.
Its sequence was observed 1,334 times across 23 samples.
:::

The [`feature-table tabulate-seqs` action](xref:rachis-library-target#q2-action-feature-table-tabulate-seqs) will provide a mapping of feature IDs to sequences, and provide links to easily BLAST each sequence against the NCBI nt database.
We can also include the feature frequency information in this visualization by passing it as metadata, similar to how we merged metadata in [](#merge-metadata).
In this case, however, we're looking at [feature metadata](xref:rachis-glossary-target#term-feature-metadata), as opposed to [sample metadata](xref:rachis-glossary-target#term-sample-metadata).
As far as QIIME 2 is concerned, there is no difference between these two—in our case, it'll only be the identifiers that differ.

This visualization will be very useful later in the tutorial when you want to learn more about important specific features in the dataset.

:::{describe-usage}

asv_frequencies_as_md = use.view_as_metadata('asv_frequencies_md',
                                                 asv_frequencies)

use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='tabulate_seqs'),
    use.UsageInputs(data=asv_seqs,
                    metadata=asv_frequencies_as_md),
    use.UsageOutputNames(visualization='asv_seqs'),
)
:::

:::{exercise} Exploring feature data (part 1).
:label: feature-data-1
What is the taxonomy associated with the most frequently observed feature, based on a BLAST search?
:::

:::{solution} feature-data-1
:class: dropdown
There are many equivalently high scoring BLAST hits for this ASV sequence (100% match covering 100% of the query sequence), and the GenBank records for many of the hits suggest it's an organism associated with the human gut microbiome.
The first result returned (as of this writing on 28 August 2026; GenBank: FJ368087.1) labels this as a 16S rRNA sequence from an Uncultured bacterium clone and notes that it was identified in human fecal samples.
Other BLAST hits suggest that it's a member of the *Blautia* genus.
Conflicting species labels, including *luti* and *wexlerae*, are associated with top scoring BLAST results, suggesting that we likely don't have species resolution for this short 16S sequence.
:::

### Clustering ASVs into Operational Taxonomic Units (OTUs)

OTU clustering was historically done as a crude quality control step (i.e., assuming that slight differences were a result of sequencing error, not true biological variation) and to speed up processing of data when the software and computers we were using for these analyses were less capable.
This would generally be based on grouping sequences based on their percent identity to one another; for example, if two ASVs' sequences matched each other at ≥ 97% identity, they would be represented in the feature table as a single feature with their counts summed together.
However, we now have much better quality control methods, and more capable computers, and, as a result, OTU clustering is no longer considered best practice.
We don't recommend OTU clustering in most workflows, nor do we apply it in the gut-to-soil tutorial.

We instead recommend ASVs as features in marker gene studies because they provide higher resolution, improve reproducibility, and facilitate more accurate taxonomic classification.
Because ASVs represent exact amplicon sequences, they can be consistently identified across samples and datasets and facilitate comparisons across microbiome studies.

It is however possible to do OTU clustering with QIIME 2, and you can find instructions in [our documentation](https://amplicon-docs.qiime2.org/en/latest/how-to-guides/cluster-reads-into-otus/) if you'd like to do this.
[](#fig-otu-vs-asv) illustrates how the ASVs in the example data might be grouped into OTUs.
Notice that the resolution of the resulting feature table is lower: ASVs 9 and 10, for example, are grouped into OTU7, and that masks the observation that ASV10 is absent from several of the earlier samples in our time series.
This may therefore be masking useful information that could have been lost from our study.

:::{include} figures/otu-vs-asv.md
:::

## Filtering features from a feature table

If you review the tabulated feature sequences or the feature detail table of the feature table summary in `asv-table.qzv`, which present the same data illustrated in [](#fig-summarize-frequencies), you'll notice that there are many sequences that are observed in only a single sample.
Here we'll filter those out to reduce the number of sequences we're working with.
When working with large feature tables, this filtering step can dramatically reduce memory (RAM) requirements for analyses that load the feature table into memory.
This can be particularly helpful when computing kmer-based diversity metrics, which we'll get to later in this tutorial.
This filtering step will also speed up processes that operate on each feature in the feature table (such as taxonomic annotation).

This is a two-step process.
First, we filter our feature table, and then we use the new feature table to filter our sequences to only the ones that are contained in the new table.

:::{describe-usage}
asv_table_ms2, = use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='filter_features'),
    use.UsageInputs(table=asv_table,
                    min_samples=2),
    use.UsageOutputNames(filtered_table='asv_table_ms2'),
)
:::

:::{describe-usage}
asv_seqs_ms2, = use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='filter_seqs'),
    use.UsageInputs(data=asv_seqs,
                    table=asv_table_ms2),
    use.UsageOutputNames(filtered_data='asv_seqs_ms2'),
)
:::

You may notice that `-ms2` is included in the new output names here.
This was added to remind us that these data artifacts are filtered to contain only those features present in a **m**inimum of **2** **s**amples.
Generally speaking, file names are a convenient place to store information like this, but they're unreliable.
File names can easily be changed, and therefore could be modified to contain inaccurate information about the data they contain.
Luckily, `rachis`'s provenance tracking system records all of the information that we need about how results were generated.
We're therefore free to include information like this in file names if it's helpful for us, but we shouldn't ever rely solely on the file names.
If you're ever in doubt about how an Artifact or Visualization was produced, refer to `rachis`'s [data provenance](xref:rachis-glossary-target#term-data-provenance) which is the definitive source of truth about how a Result was created.

:::{exercise} Exploring the feature table summary (part 2).
:label: summarize-asv-table-ms2
Now that you have a second (filtered) feature table, create your own command to summarize it, like we did for the original feature table.
How many features were filtered out here?
How did that impact the total frequency (i.e., the number of sequences from our original fastq data that are represented in the table) for the feature table as a whole?
:::

::::{solution} summarize-asv-table-ms2
:class: dropdown

:::{describe-usage}
_, _, asv_frequencies_ms2 = use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='summarize'),
    use.UsageInputs(table=asv_table_ms2,
                    metadata=sample_metadata),
    use.UsageOutputNames(summary='asv_table_ms2',
                         sample_frequencies='sample_frequencies_ms2',
                         feature_frequencies='asv_frequencies_ms2'))
:::

Be sure to run this as we're going to use one of the results below.
::::

## Taxonomic annotation

Before we begin analyzing our feature table, we'll generate taxonomic annotations for our sequences using the [q2-feature-classifier plugin](xref:rachis-library-target#q2-plugin-feature-classifier) [](doi:10.1186/s40168-018-0470-z).
We're going to do this here by training a machine learning classifier, and then applying it to our data.

### Training a taxonomic classifier

Training a machine learning based taxonomic classifier in QIIME 2 requires two artifacts as input: reference sequences (in a `FeatureData[Sequence]` artifact; see [](#fig-reference-database)a), and reference taxonomy for each of those sequences (in a `FeatureData[Taxonomy]` artifact; see [](#fig-reference-database)b).
The result of this process is a `TaxonomicClassifier` artifact ([](#fig-reference-database)c).

:::{include} figures/reference-database.md
:::

:::{note}
The `TaxonomicClassifier` artifact is different from any of the artifacts that we've worked with up to this point in an important way.
All of the artifacts we've worked with so far contain static data in various formats, like fastq or tabular data.
The `TaxonomicClassifier` artifact contains executable code that is run on your data by the computer where you perform your analysis.
This means that `TaxonomicClassifier` artifacts are more like computer programs than traditional data files, and you should keep that in mind when using them.
For example, you should make sure that you trust the person who provided it to you before you apply it to sensitive data or on computers that contain private information (as just about any computer does).
As machine learning and artificial intelligence tools become more commonplace in our day-to-day lives, this paradigm is becoming increasingly prevalent: specifically, that the output of an analysis is new computer code (like a model capable of differentiating samples from healthy tissue versus cancer tissue) rather than a static data table (like results from a BLAST search).
This requires new levels of vigilance to ensure privacy and security when working with research outputs.
`rachis` Results can be [cryptographically signed](https://use.rachis.org/en/latest/how-to-guides/sign-and-verify-artifacts/#verify-signed-result), which can help you verify that the Artifact or Visualization was provided by the person who you think provided it.
:::

The taxonomic classifier used here is trained on reference sequences and taxonomy from GTDB version 202.0, which is an old version of the GTDB reference database [](doi:10.1093/nar/gkab776).
We use it here because the reference data is relatively small, enabling classifier training and application to run on most modern computers.
For comparison, GTDB version 202.0 contains 32,884 sequences (31,319 Bacteria + 1,565 Archaea) while GTDB version 232.0 (the most recent version, as of this writing on 2 October 2026) contains 93,770 sequences (88,481 Bacteria + 5,289 Archaea).

Training a taxonomy classifier can be a slow and memory-intensive step, and this is one of the slower steps in this tutorial.

First, we'll obtain the sequence data and the associated taxonomy annotations.
This is done using the [RESCRIPt plugin](xref:rachis-library-target#q2-plugin-rescript) [](doi:10.1371/journal.pcbi.1009581), which provides many utilities that are useful for training taxonomy classifiers.
Here we'll use the [`get-gtdb-data` action](xref:rachis-library-target#q2-action-rescript-get-gtdb-data), which automates the downloading of reference data from the GTDB database.
We'll apply it here to download the 16S reference from GTDB version 202.0, including both the `FeatureData[Sequence]` and `FeatureData[Taxonomy]` artifacts.

:::{describe-usage}
reference_taxonomy, reference_sequences = use.action(
    use.UsageAction(plugin_id='rescript',
                    action_id='get_gtdb_data'),
    use.UsageInputs(version='202.0',
                    db_type='SpeciesReps'),
    use.UsageOutputNames(gtdb_taxonomy='reference-taxonomy',
                         gtdb_sequences='reference-sequences'))
:::

If you'd like to inspect the reference data before using it, which is never a bad idea, you can do that by generating a summary which includes high-level characteristics and a table of the sequences and their associated taxonomy as follows:

:::{describe-usage}
reference_taxonomy_collection = use.construct_artifact_collection(
    'reference-taxonomy-collection',
    {'GTDB_r202_SpeciesReps': reference_taxonomy}
)

use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='tabulate_seqs'),
    use.UsageInputs(data=reference_sequences,
                    taxonomy=reference_taxonomy_collection,
                    merge_method='intersect'),
    use.UsageOutputNames(visualization='reference_taxonomy'),
)
:::

(suboptimal-classifier-explanation)=
We'll next use this reference data to train a Naïve Bayes taxonomy classifier ([](#fig-reference-database)c).
Note that in the following command we change two parameters, `feat-ext--n-features` and `classify--chunk-size`, from their default settings during training here to allow the classifier to be trained with less memory.
We don't recommend changing these from their defaults in practice.[^classifier-training-defaults]
Rather this change is made here because this is a tutorial: we want users to be able to quickly follow along on their personal computer, where in practice classifier training on the most up-to-date rRNA reference databases often requires the use of a high-performance computer (HPC), like a university cluster computer.
To remind readers that they shouldn't use this classifier in practice, we're going to refer to the one we build here as our *suboptimal 16S rRNA classifier*.

:::{describe-usage}
classifier, = use.action(
    use.UsageAction(plugin_id='feature_classifier',
                    action_id='fit_classifier_naive_bayes'),
    use.UsageInputs(reference_reads=reference_sequences,
                    reference_taxonomy=reference_taxonomy,
                    feat_ext__n_features=2048,
                    classify__chunk_size=2000),
    use.UsageOutputNames(classifier='suboptimal-16S-rRNA-classifier'))
:::

### Annotating an Artifact or Visualization

Another useful way we can leverage `rachis`'s provenance tracking is with the use of [Annotations](xref:rachis-glossary-target#term-annotation).
This was indirectly mentioned earlier in the context of cryptographically signing your Results, as these signatures are a type of Result Annotation (`Annotation[Signature]`).
In addition to [Signatures](xref:rachis-glossary-target#term-signature), an `Annotation[Note]` can be used to attach information to a Result, either as text or through a text file attachment.
Below is an example of how we can append an `Annotation[Note]` to our classifier.

% TODO: `qiime tools annotation-create` has no Usage API equivalent, so this is presented for the command line only.
```shell
qiime tools annotation-create \
  --input-path suboptimal-16S-rRNA-classifier.qza \
  --annotation-type Note \
  --name 'suboptimal-classifier-justification' \
  --text 'This classifier was trained with non-default parameters for demonstration purposes. '\
'It should not be used in real-world analysis.' \
  --output-path suboptimal-16S-rRNA-classifier.qza
```

You'll notice that the default output path for this annotated classifier is the same; adding an Annotation to a Result will mutate the Result rather than create a new one, so a new filepath is not required (but can be specified if you prefer to keep an un-annotated copy of the same Result).

### Identifying taxonomic classifiers in your own work

When you're ready to work on your own data, one of the choices you'll need to make is what classifier to use for your data.
You can discover pre-trained classifiers, including classifiers for the most recent versions of GTDB and SILVA, on the [`rachis-library`](https://library.rachis.org).
If you don't find a classifier that will work for you there, you may be able to [find one on the Forum](https://forum.qiime2.org/tag/taxonomy) or you can [train your own](https://github.com/bokulich-lab/RESCRIPt).
If you do plan to train your own, we recommend the use of environment-weighted classifiers [](doi:10.1038/s41467-019-12669-6), also available from the [`rachis-library`](https://library.rachis.org).

### Apply our taxonomy classifier

Next, we'll annotate our sequence data ([](#fig-feature-table)b) with our classifier ([](#fig-reference-database)c) to generate taxonomic annotations for our sequences ([](#fig-taxonomic-assignment)).

:::{describe-usage}
taxonomy, = use.action(
    use.UsageAction(plugin_id='feature_classifier',
                    action_id='classify_sklearn'),
    use.UsageInputs(classifier=classifier,
                    reads=asv_seqs_ms2),
    use.UsageOutputNames(classification='taxonomy'))
:::

:::{include} figures/taxonomic-assignment.md
:::

The [`classify-sklearn` action](xref:rachis-library-target#q2-action-feature-classifier-classify-sklearn) assigns a confidence value between 0.0 and 1.0 (inclusive) to every taxonomic level it assigns to each input sequence.
The annotation that will be generated as output is then assigned to the most specific taxonomic label for which the confidence is above a user-defined threshold (default 0.70).
When reviewing [](#fig-taxonomic-assignment), you can see that some ASVs have higher resolution (i.e., "deeper" or "more specific") assignments than others.
For example, ASV2 gets a species-level assignment, while ASV1 gets a genus-level assignment, and ASV4 only gets a domain-level assignment.
This means that ASV2's species assignment was assigned with a confidence greater than or equal to 0.70, while ASV4 received a phylum-level assignment at a confidence less than 0.70.
Low resolution assignments can happen for a few reasons including: that a close representative to the sequence in question is not represented in the database, either because you're working with an uncommon or previously unknown sequence or because you have a sequence that doesn't represent one found in nature, for example as a result of PCR error; or that there is low information content in your amplicon for differentiating certain taxa (i.e., two species in the same genus have identical 16S sequences, so you can't tell them apart from the sequencing data you have).
It's normal to have some features, like ASV4 in [](#fig-taxonomic-assignment), that have low resolution taxonomy assignments.
If many of your sequences are in that category, it may indicate that there is a problem with your sequencing data.

::::{margin}
(compare-taxonomic-annotations)=
:::{tip}
If you want to compare taxonomic annotations achieved with different classifiers, you can do that with the [`feature-table tabulate-seqs` action](xref:rachis-library-target#q2-action-feature-table-tabulate-seqs) by passing in multiple `FeatureData[Taxonomy]` artifacts.
See an example of what that result might look like [here](https://view.qiime2.org/visualization/?src=https://zenodo.org/api/records/13887457/files/asv-seqs-ms10.qzv/content).

While you have that visualization loaded, take a look at the data provenance.
The complexity of that data provenance should give you an idea of why it's helpful to have the computer record all of this information, rather than trying to embed it all in file names or keep track of it in your written notes.

What was used as the DADA2 trim and trunc parameters for the data leading to this visualization?
(Hint: use the provenance search feature).
:::
::::

To get an initial look at the taxonomic classifications, we can integrate taxonomy in the `tabulate-seqs` summary, like the one we generated above.
In this command, we are providing a label for the `taxonomy.qza` input, which will label the database source in the resulting visualization.
This is possible because this input is defined as a [Collection](xref:rachis-glossary-target#term-collection), so we could provide multiple `FeatureData[Taxonomy]` artifacts as input to this step (for example, to compare taxonomy assignments obtained by different classifiers).

:::{describe-usage}

asv_frequencies_ms2_as_md = use.view_as_metadata('asv_frequencies',
                                                 asv_frequencies_ms2)

taxonomy_collection = use.construct_artifact_collection(
    'taxonomy_collection', {'GTDB_r202_SpeciesReps': taxonomy}
)

use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='tabulate_seqs'),
    use.UsageInputs(data=asv_seqs_ms2,
                    taxonomy=taxonomy_collection,
                    metadata=asv_frequencies_ms2_as_md),
    use.UsageOutputNames(visualization='asv_seqs_ms2'),
)
:::

Recall that our `asv-seqs.qzv` visualization allows you to easily BLAST the sequence associated with each feature against the NCBI nt database.
This visualization has the same functionality, but now includes the taxonomic annotation of the ASVs as determined using QIIME 2.
Using the `asv-seqs-ms2.qzv` Visualization created here, compare the taxonomic assignments with the taxonomy of the best BLAST hit for a few features.

:::{exercise} Exploring feature data (part 2).
:label: feature-data-2
What is the taxonomy associated with the most frequently observed feature, based on a BLAST search?
How does that compare to the taxonomy assigned by our feature classifier?

In general, we tend to trust the results of our feature classifier over those generated by BLAST, though the BLAST results are good for a quick look or a sanity check.
The reason for this is that the reference databases used with our feature classifiers tend to be specifically designed for this purpose and in some cases all of the sequences included are vetted.
The BLAST databases can contain mis-annotations that may negatively impact the classifications.
:::

:::{solution} feature-data-2
:class: dropdown
The highest frequency feature, `c6c3ab4e828fb40d6e05967b7aac9338`, is labeled as follows by our classifier:

```
d__Bacteria;p__Firmicutes_A;c__Clostridia;o__Lachnospirales;f__Lachnospiraceae;g__Blautia_A
```

This aligns well with our BLAST results when viewed collectively.
In the solution to [](#feature-data-1), we noted that the best matches included hits in the *Blautia* genus, but no conclusive species label.
This is exactly what our classifier reports: notice that the assignment ends after `g__Blautia_A`, with no species level assignment.
The lack of a species-level assignment is a result of the classifier not being able to make a confident species-level assignment.

It's worth noting that our consideration of this sequence in [](#feature-data-1) as belonging to the *Blautia* genus required manual review of many top BLAST hits, as the first hit described it simply as an uncultured bacterium clone.
Additionally, several of the top scoring BLAST assignments viewed individually could have led to over-confidence in a species assignment.

One additional note: when GTDB assigns a letter to a taxonomic label (like the A in `Blautia_A` here) that means that they have observed that the previously named group (*Blautia*, in this case) is not monophyletic based on the multi-gene trees that guide their taxonomy.
The A therefore indicates one monophyletic grouping of taxa in what are known as *Blautia*.
:::

## Taxonomic analysis

At this point, we have what we need to get our first high-level look at our samples.
Specifically, we can see which taxa are present in our samples.
This is generally viewed through a taxonomic barplot ([](#fig-taxa-barplot)).
The QIIME 2 taxonomic barplots enable viewing the observed taxonomic composition of our samples by converting all feature frequencies to relative frequencies ([](#fig-taxa-barplot)a), and then grouping features and summing their counts based on their taxonomic annotation ([](#fig-taxonomic-assignment)).
Taxonomic barplots can then be presented for different taxonomic levels: in our schematic figures [](#fig-taxa-barplot)b presents the phylum-level composition, and [](#fig-taxa-barplot)c presents the genus level composition.
(Features without assignment at the current taxonomic level are grouped based on their deepest level of assignment: for example, in [](#fig-taxa-barplot)b the `d__Bacteria` level groups all features that are assigned to `d__Bacteria` but not to a phylum.)
These plots allow us to observe a pattern in the schematic data: the sample composition transitions from more human-excrement-like in the early composting timepoints to more food-compost-like in the later timepoints.
We can also see which taxa may be responsible for this change in composition or may be responding to changing composting conditions over time.
(Refer to [](#fig-sample-metadata-study) if you need to remind yourself of the sample types.)

:::{include} figures/taxa-barplot.md
:::

Generate taxonomic barplots for the tutorial data with the following command and then open the Visualization.

:::{describe-usage}

use.action(
    use.UsageAction(plugin_id='taxa',
                    action_id='barplot'),
    use.UsageInputs(table=asv_table_ms2,
                    taxonomy=taxonomy,
                    metadata=sample_metadata),
    use.UsageOutputNames(visualization='taxa_bar_plots'))
:::

:::{exercise} Taxa bar plots.
:label: taxa-bar-plots
Visualize the samples at the phylum level, and then sort and label the samples by `SampleType`.
What are the dominant phyla in the Human Excrement `SampleType`?
What are the dominant phyla in the Human Excrement Compost `SampleType`?
Add sorting and labeling by `Composting Time Point`.
Do you see patterns that align with those in [](#fig-taxa-barplot)b and [](#fig-taxa-barplot)c?
:::

:::{solution} taxa-bar-plots
:class: dropdown
The most dominant phylum in the Human Excrement samples appears to be `p__Firmicutes_A` followed by `p__Bacteroidota`.
In the Human Excrement Compost samples, the dominant phylum appears to be `p__Proteobacteria`, followed by `p__Bacteroidota`.
`p__Bacteroidota` abundance appears to be positively correlated with time, and `p__Actinobacteriota` may be negatively correlated with time, but these patterns are not perfect and somewhat hard to discern from this plot.
:::

While taxa barplots can help us explore our data at a high level, such as comparing dominant phyla across sample types, in practice they are limited in their ability to help us understand more nuanced differences between our samples.
And, as our data increases in size, these plots become increasingly hard to interpret.
For example, try comparing the genus-level composition of adjacent timepoints: for a few different values of x, can you tell if timepoint x is more similar to timepoint x-1 or x+1 at the genus level?
For this type of question, we need more quantitative approaches, and this is where alpha and beta diversity metrics help us.
Generating a taxonomy barplot is almost always an early step in an analysis, and something that is referred back to when considering other results.
It is also useful for assessing whether your sample compositions align with your expectations, so can be used as a sanity check of the data.
It rarely, if ever, represents a key finding on its own.

## Even sampling

One additional topic must be considered before we move on to alpha and beta diversity metrics, and that is the variation in the number of sequences obtained per sample (the [sequencing depth](xref:rachis-glossary-target#term-sequencing-depth)) that is observed across samples in modern DNA sequencing runs.
Refer to [](#fig-even-sampling)a, which presents our feature table alongside the sequencing depth (labeled Total frequency) for each sample following quality control ([](#fig-even-sampling)b).
Notice that the sequencing depth for sample `4ac2` is approximately 20 times that of sample `5f08`.
Differences in the sequencing depth across samples in an amplicon-based microbiome study generally are thought to be stochastic, rather than representative of true differences across samples (with an exception being very low microbial biomass samples, which can have very low sequencing depth because there are few microbial cells to sequence).
This difference can however impact the diversity metrics that we calculate on those samples, such that they may appear to be different as a result of the sampling depth obtained in each, rather than as a result of true biological differences between the samples.

:::{include} figures/even-sampling.md
:::

An analogy can be helpful here.
Imagine a plant biologist (and not a very good one, for the sake of this example) who is interested in comparing the number of plant species found in the Sonoran Desert to that observed in the Costa Rican rainforest.
To do this, they first survey 60 m² of the desert.
Then, maybe because it's harder to move around, they sample only 3 m² of the rainforest.
When they get back to the lab, they tally up their results and conclude that there are more species of plant in the desert than there are in the rainforest.
(Remember—they are not a very good biologist!)
While this observation sounds quite surprising, we know there is a methodological flaw: they expended more effort counting species in the desert, so it's not surprising that they found more species.

The land area sampled is an analog for the number of sequences collected in our microbiome study: if we obtain more sequences for some samples, we're effectively expending more effort sampling them, and it shouldn't be surprising if that impacts findings such as the count of the number of ASVs observed in each sample.
Differences in sampling depth also impact more complex metrics of sample diversity.

This is generally addressed in microbiome studies by selecting a sequencing depth to normalize all samples for diversity metric computation.
This is referred to as the [even sampling depth](xref:rachis-glossary-target#term-even-sampling-depth) and selecting this is a common source of questions; we'll provide guidance on this next.
[](#fig-even-sampling)c-d illustrates selecting an even sampling depth of 1800 for the example feature table.
When this is done, the sequence counts are resampled for each sample (typically without replacement) until the total count for that sample is 1800.
This is repeated for all samples, and if a sample has fewer than 1800 sequences, that sample will be dropped from the resulting table.
This is the case for sample `5f08` in [](#fig-even-sampling)c: it is the sample with the lowest sampling depth in this feature table.
At an even sampling depth of 1800, it would be excluded from diversity metric calculations.

Performing a single resampling of the feature table to a specific even sampling depth (e.g., as illustrated in either [](#fig-even-sampling)c or [](#fig-even-sampling)e) is referred to as [rarefying](xref:rachis-glossary-target#term-rarefaction) the feature table.
The resulting diversity metrics however are sensitive to the specific sampling that was performed, and as is clear by comparing the "Total" rows in [](#fig-even-sampling)d and [](#fig-even-sampling)f to that in [](#fig-even-sampling)b, it results in excluding a lot of the data that was collected during sequencing.
The approach that we'll use here, [rarefaction](xref:rachis-glossary-target#term-rarefaction), performs the resampling step n times, and then averages downstream results.
This results in more robust diversity metric calculations, and ensures that those diversity metrics have accounted for more of the underlying data.
We'll apply that approach here using the [q2-boots plugin](xref:rachis-library-target#q2-plugin-boots), after illustrating the simpler case of computing diversity metrics from the rarefied feature table in [](#fig-even-sampling)c.

If our plant biologist wanted to normalize their sampling effort by these approaches, they could consider a few options assuming they recorded the specific location(s) where each species was observed during their sampling.
They could apply an approach like rarefying by selecting at random a 3 m² subplot from the desert, and tallying the desert plant species count in that subplot for comparison to the rainforest plant count.
Or, more robustly, they could randomly sample multiple 1 m² subplots from both the desert and rainforest, tally the average number of plant species in each subplot, and compare those averages.
(Even in this case however, our plant biologist is to be pitied: without multiple replicates per site, any results they come up with are not likely to be very convincing.)

(gut-to-soil-tutorial:selecting-an-even-sampling-depth)=
## Selecting an even sampling depth

Selecting an even sampling depth is a balance between discarding sequences and discarding samples.
If a lower even sampling depth is chosen to avoid excluding samples, each sample is represented by fewer sequences.
If a higher depth is chosen to represent each sample by more sequences (see [](#fig-even-sampling)e-f), then more samples may be discarded.

Generally, selecting an even sampling depth requires a few important considerations.
First, you want it to exclude samples where the sequencing seems to have failed—this might be those where the sampling depth is in the ones, tens, or hundreds.
It's not a bad idea to remove these from your feature table entirely, using [`qiime feature-table filter-samples`](xref:rachis-library-target#q2-action-feature-table-filter-samples).
Then, you want to set it low enough that you're losing an acceptable number of samples, which is a painful decision to make but is a choice that needs to be made on a per-study basis.
The `asv-table-ms2.qzv` that you created for [](#summarize-asv-table-ms2) can help you explore the trade-off between number of sequences and number of samples discarded.
Experiment with the dropdown box and the slider bar in the *Interactive Sample Detail* tab.
Selecting the `SampleType` metadata category will group your samples by their type allowing you to see how many samples you'll lose from each category for a given sampling depth.
Think about which categories are the ones that you need to compare, and ensure that you're selecting an even sampling depth that retains a sufficient number of samples in each of those categories.
Finally, you ideally want to choose an even sampling depth where you expect your diversity metrics to begin to stabilize, which is something you can assess with alpha and beta rarefaction analyses.

[](#fig-alpha-rarefaction)a illustrates a simplified alpha rarefaction plot.
In an alpha rarefaction analysis, you step through a series of even sampling depths and generate n feature tables at each step.
You then compute an alpha diversity metric (observed features, in [](#fig-alpha-rarefaction)a), and you observe where those values begin to plateau for each of your samples or sample categories.
That gives you an idea of where you expect your alpha diversity values to stabilize, and in general selecting a value in that range that retains as many samples as possible is ideal.
Notice that the observed features values have stabilized in [](#fig-alpha-rarefaction) at the two example even sampling depths that we explored in [](#fig-even-sampling), but an even sampling depth of 1800 retains six samples while an even sampling depth of 8000 retains only three samples.
(Because the constructed data used in the figures is so simplistic, our alpha diversity values stabilize very quickly and indicate that a lower depth could be selected for this data to retain all seven samples.
For the sake of illustration though, we'll select the value of 1800 so that our downstream results represent the more common scenario where some samples that were sequenced must be excluded from diversity analysis.)

:::{include} figures/alpha-rarefaction.md
:::

[](#fig-alpha-rarefaction)b-c mirror the plots generated by QIIME 2's `alpha-rarefaction` action, which we will run next.
In the QIIME 2 alpha rarefaction plots, samples are often grouped by metadata (e.g., all HEC samples are grouped into one line in [](#fig-alpha-rarefaction)b), to help with visualizing these plots for large collections of samples.
The secondary plot (illustrated in [](#fig-alpha-rarefaction)c) presents how many samples are retained for each group at each even sampling depth.

We can generate an alpha rarefaction plot for the tutorial data using the [`alpha-rarefaction` action](xref:rachis-library-target#q2-action-diversity-alpha-rarefaction).
This visualizer computes one or more alpha diversity metrics at multiple sampling depths, in steps between 1 (optionally controlled with `min-depth`) and the value provided as `max-depth`.
At each sampling depth step, 10 rarefied tables will be generated, and the diversity metrics will be computed for all samples in the tables.
The number of iterations (rarefied tables computed at each sampling depth) can be controlled with the `iterations` parameter.
The diversity values will be plotted for each sample at each even sampling depth, and samples can be grouped based on metadata in the resulting visualization if sample metadata is provided.

The value that you provide for `max-depth` should be determined by reviewing the "Frequency per sample" information presented in the `asv-table-ms2.qzv` file.
In general, choosing a value that is somewhere around the median frequency seems to work well, but you may want to increase that value if the lines in the resulting rarefaction plot don't appear to be leveling out, or decrease that value if you seem to be losing many of your samples due to low total frequencies closer to the minimum sampling depth than the maximum sampling depth.

:::{describe-usage}
use.action(
    use.UsageAction(plugin_id='diversity',
                    action_id='alpha_rarefaction'),
    use.UsageInputs(table=asv_table_ms2,
                    max_depth=260,
                    metadata=sample_metadata),
    use.UsageOutputNames(visualization='alpha_rarefaction'))
:::

The Visualization will have two plots.
The top plot is an alpha rarefaction plot, and is primarily used to determine if the richness of the samples has been fully observed or sequenced.
If the lines in the plot appear to "level out" (i.e., approach a slope of zero) at some sampling depth along the x-axis, that suggests that collecting additional sequences beyond that sampling depth would be unlikely to result in the observation of additional features.
If the lines in the plot don't level out, this may be because the richness of the samples hasn't been fully observed yet (because too few sequences were collected), or it could be an indicator that a lot of sequencing error remains in the data (which is being mistaken for novel diversity).

The bottom plot in this Visualization is important when grouping samples by metadata.
It illustrates the number of samples that remain in each group when the feature table is rarefied to each sampling depth.
If a given sampling depth *d* is larger than the sequencing depth of a sample *s* (i.e., the number of sequences that were obtained for sample *s*), it is not possible to compute the diversity metric for sample *s* at sampling depth *d*.
If many of the samples in a group have lower total frequencies than *d*, the average diversity presented for that group at *d* in the top plot will be unreliable because it will have been computed on relatively few samples.
When grouping samples by metadata, it is therefore essential to look at the bottom plot to ensure that the data presented in the top plot is reliable.

:::{exercise} Alpha rarefaction.
:label: alpha-rarefaction
When grouping samples by "SampleType" and viewing the alpha rarefaction plot for the "observed_features" metric, which sample types (if any) appear to exhibit sufficient diversity coverage (i.e., their rarefaction curves level out)?
:::

:::{solution} alpha-rarefaction
:class: dropdown
The Microbe Mix sample appears to level out quickly.
The Human Excrement, Human Excrement Compost, and Food Compost additionally appear to level out, though there appears to be some positive slope remaining.
Other sample types appear unstable, but by comparing with the plot underneath the rarefaction plot we can see that those groups are based on relatively few samples at the higher sequencing depths.
:::

There are many [QIIME 2 Forum](https://forum.qiime2.org) posts on the topic of selecting an even sampling depth, and some [video content on the QIIME 2 YouTube channel](https://youtu.be/q-S2qVMyCVs).
You can refer to those, and discuss with others on the forum if you have questions when making this decision for your analysis.
We recommend keeping in mind a couple of points when deciding on a sampling depth.
First, relative diversity metrics tend to be quite stable to low even sampling depths.
For example, if you see a pattern of similarity between your samples at a relatively high even sampling depth, that pattern tends to also be present at relatively lower even sampling depths within reason.
It never hurts to experiment with multiple even sampling depths and compare the results to assess whether your conclusions change.
You can even present the results from different even sampling depths as supplementary analysis in a research paper—for example, by focusing on a lower even sampling depth that retains more samples in your main text figures, but then presenting results obtained with higher even sampling depths in a supplement to confirm that the patterns you presented are still apparent.
Second, if you're using the rarefaction approach in q2-boots, which we illustrate in this tutorial, you can feel confident knowing that even if you choose a lower even sampling depth that discards a large fraction of your sequences, because the rarefy step will be run multiple times, those sequences will be used to define representative sample composition.
In other words, even though an individual rarefy step may consider only a small fraction of your sequences, the multiple iterations (default of 100) will consider a much larger fraction.

## Alpha and beta diversity analysis background

(gut-to-soil-tutorial:diversity-metrics)=
::::{dropdown} Background: alpha and beta diversity metrics  ([](#fig-alpha-diversity) - [](#fig-kmer-features))

Alpha (α) and beta (β) diversity analysis are common next steps in a microbiome amplicon analysis, and facilitate interpretation of the relative similarities and differences across samples.
α diversity metrics are computed from a single sample at a time, and as a result are often referred to as "within-sample" diversity.
β diversity metrics on the other hand are computed on pairs of samples, and are therefore often referred to as "between-sample" diversity.

Experimental choices ranging from DNA extraction protocol through PCR primer choice and bioinformatics processing, and experimental artifacts such as sequencing depth, all impact our observation of microbiome sample composition, and α and β diversity should therefore always be considered a function of those parameters in addition to our observed sample composition.
For this reason, care must always be taken if trying to compare microbiome compositions across studies.

There are many diversity metrics implemented in QIIME 2—about 30 α metrics and 25 β diversity metrics.
Each tells us about a different aspect of the data, and computing and comparing multiple diversity metrics can help you understand different features of your data.
For example, stepping back from biology for a moment, imagine that you want to compute the physical distance between two cities.
There are various ways that you could measure this, and the way you choose to measure it depends on what you need to do with that distance.
You could draw a straight line between the two cities on a map, measure that line, and you'd have a distance between the cities.
But, if you're driving between the cities, that distance won't be very relevant unless there is a road that connects the cities by the line that you've drawn.
In this case, a more relevant distance will be that of the shortest drivable route between the two cities, achieved by measuring the lengths of the roads composing the route.
Just as there are many ways to compute the distance between two cities, there are many ways to assess the diversity of biological communities.

In this section, we're going to briefly cover three independent categorizations of diversity metrics and then mention specific examples of these that can be computed in QIIME 2.
Detailed discussion of each metric is beyond the scope of this tutorial document, but there is plenty of online content about these.

### α Diversity versus β Diversity

The first category of diversity metrics we'll discuss is α diversity versus β diversity.
As described above, α diversity is computed on a single sample.
These methods generally encompass measures of richness and/or evenness of a sample.
The simplest of these is observed features, which we discussed above in the context of [](#fig-alpha-rarefaction).
This is a simple count of the features observed in a sample with a frequency of greater than zero.
[](#fig-alpha-diversity)a presents this for each sample based on the evenly sampled feature table in [](#fig-even-sampling)c.
To ensure that you understand it, as well as the other diversity metrics that are presented in this section, you can compute them by hand on the [](#fig-even-sampling)c feature table and confirm that you obtain the same results that you see here.
The result of computing an α diversity metric on a set of samples is a vector of alpha diversity values.

:::{include} figures/alpha-diversity.md
:::

β diversity is computed on pairs of samples, and most commonly represents a distance between samples.
A lower β diversity value between a pair of samples suggests that they are more similar; a higher β diversity value suggests that they are more dissimilar.
A simple β diversity metric is Jaccard dissimilarity, which is illustrated in [](#fig-beta-diversity)a-b.
The Jaccard dissimilarity between samples A and B is computed as one minus the number of features present in both samples A and B divided by the number of features present in either sample A or B.
Again, you can work through the formula presented in [](#fig-beta-diversity)a on the feature table in [](#fig-even-sampling)c, and check your answers against [](#fig-beta-diversity)b to confirm that you understand how this metric is computed.
The result of computing a β diversity metric on a set of samples is a distance matrix, where each cell stores the distance between a corresponding pair of samples.
True measures of distances between samples (a category that most but not all β diversity metrics fall into) result in distance matrices that guarantee four specific criteria:

1. The distance matrix is hollow (or has zeros on the diagonal), indicating that the distance between a sample and itself will always be zero.
2. The distance matrix is symmetric, indicating that the distance between sample A and B will always be equal to the distance between sample B and A.
3. Distances between samples will always be greater than or equal to zero.
4. Distances adhere to the triangle inequality, such that the distance between samples A and C will always be less than or equal to the distance between sample A and B plus the distance between B and C.

Some operations that are applied to distance matrices downstream assume these characteristics, so it's worth being aware of these criteria.

:::{include} figures/beta-diversity.md
:::

### Qualitative versus quantitative

The next category we'll discuss here is qualitative versus quantitative diversity metrics.
Qualitative diversity metrics consider only the presence or absence of features in the feature table, not their actual counts.
Quantitative diversity metrics, on the other hand, consider the counts of the features.
The two metrics we discussed in the previous section, observed features and Jaccard dissimilarity, are both qualitative diversity metrics: notice that each only considers which features are and are not observed on a per-sample basis.
[](#fig-alpha-diversity)c-f and [](#fig-beta-diversity)c-d present quantitative α and β diversity metrics, respectively.
While it might seem that qualitative diversity metrics add little, remember that the extraction, PCR, and sequencing steps in generating our data can all obscure the true abundances of features (e.g., because of preferential extraction of some taxon's DNA over another).
It is common to compute both qualitative and quantitative metrics and present the two side-by-side, and to remember that neither is a perfect representation of the biology of our samples.

### Identity-based vs. relatedness-based

The final category we'll cover here is identity-based diversity metrics versus relatedness-based diversity metrics.
An identity-based diversity metric treats all features as independent, ignoring any possible relationship between them.
These metrics are identity-based in the sense that a feature's identity is the only information they use: two observations either represent the same feature or different features, and all different features are treated as equally distinct, whether their sequences differ by one nucleotide or many.
Relatedness-based metrics, on the other hand, incorporate relationships between features.
The metrics represented in [](#fig-alpha-diversity) and [](#fig-beta-diversity) are all identity-based metrics.

The most widely used approach for incorporating feature relatedness in diversity metric computation in microbiome studies is through phylogenetic inference: in addition to the feature table, a phylogenetic tree where the feature ids are the tips in the tree ([](#fig-phylogeny)) is included as input to the diversity metric, and the distance between the tips in the tree is interpreted as the relatedness of features by the diversity metric.
QIIME 2 supports several phylogenetic diversity metrics, including Faith's Phylogenetic Diversity ([](#fig-faith-pd)), unweighted UniFrac ([](#fig-unifrac)b-c), and weighted UniFrac ([](#fig-unifrac)d-e).

:::{include} figures/phylogeny.md
:::

:::{include} figures/faith-pd.md
:::

:::{include} figures/unifrac.md
:::

Relatedness can alternatively be integrated in other ways, such as through shared kmer composition as in the [q2-kmerizer](xref:rachis-library-target#q2-plugin-kmerizer) [](doi:10.1128/msystems.01550-24) plugin.
Briefly, this works by decomposing each ASV sequence into its constituent length-k words, or kmers, as illustrated in [](#fig-kmer-features)a.
The feature table is then updated such that the features become these kmers, and their per-sample counts are the sum of the counts of the ASV(s) in which each was observed ([](#fig-kmer-features)b).
This encodes feature relatedness directly in the feature table: evolutionarily closer sequences share more kmers, so diversity metrics computed on it integrate that relatedness.
A point worth remembering is that, as currently implemented in QIIME 2, identity-based metrics are applied to this kmer feature table, so feature relatedness is integrated through the table, not the metrics themselves.

:::{include} figures/kmer-features.md
:::

These approaches achieve the same goal of integrating feature relatedness, but in different ways: phylogenetic relationships are inferred from the data using a series of models (for multiple sequence alignment, phylogenetic reconstruction, and tree rooting), while kmer-based relatedness is a directly measured property of the observed data.
The kmer approach is objectively simpler: it's a deterministic alignment-free approach with a single free parameter, k, that provides only rough estimates of feature relatedness.
Phylogenetic inference involves many parameters and yields an explicit hypothesis about the evolutionary relationships between the features.
The phylogenetic tree, when correct, has considerably more value (branch lengths, evolutionary placement of novel organisms), but each step carries assumptions and errors can propagate forward into the diversity metric.
In practice, the simpler kmer-based diversity calculations appear to perform no worse than phylogeny-based methods [](doi:10.1128/msystems.01550-24).

Relatedness-based diversity metrics and identity-based diversity metrics provide different perspectives for resolving differences between microbiome samples, and both are common in practice.

### Phylogenetic inference

QIIME 2 offers a few ways to build phylogenetic trees for use in computing phylogenetic diversity metrics, including a reference-based approach in the [q2-fragment-insertion plugin](xref:rachis-library-target#q2-plugin-fragment-insertion) [](doi:10.1128/mSystems.00021-18) and *de novo* (i.e., reference-free) approaches in the [q2-phylogeny plugin](xref:rachis-library-target#q2-plugin-phylogeny).
Each of these methods has benefits and drawbacks.
The reference-based approach, by default, is specific to 16S rRNA marker gene analysis.
This uses a pre-computed reference tree and then inserts the observed sequences into that tree.
*De novo* tree building, an alternative, doesn't rely on any outside information so can be used with any phylogenetically informative marker gene (not just 16S).
Reference-based tree building is generally considered to result in a higher-quality tree than *de novo* tree building, but you need to have a tree that you trust to begin with.
Reference-based tree building is illustrated in the [Parkinson's Mouse tutorial](https://docs.qiime2.org/2024.10/tutorials/pd-mice/), and *de novo* tree building is illustrated in the [Moving Pictures tutorial](https://amplicon-docs.qiime2.org/en/latest/tutorials/moving-pictures.html).
In the gut-to-soil tutorial, we opt for the simpler kmer-based approach for computing relatedness-based diversity metrics so we will skip the phylogenetic reconstruction step.

::::

(gut-to-soil:kmer-based-diversity-analysis)=
### Running the kmer-diversity Pipeline

In the past few sections we've covered a lot of background while running very few commands.
We're going to pull this together in this section by running rarefaction (i.e., multiple iterations of rarefying) and using the resulting evenly sampled feature tables to compute qualitative and quantitative relatedness-based α and β diversity metrics.
This is a complex but very common workflow, so QIIME 2 provides a single [Pipeline](xref:rachis-glossary-target#term-pipeline), [`kmer-diversity`](xref:rachis-library-target#q2-action-boots-kmer-diversity), to do all of this work in a single step in the [q2-boots](xref:rachis-library-target#q2-plugin-boots) [](doi:10.12688/f1000research.156295.1) plugin.
The q2-boots plugin also provides all the constituent actions, so you can run the steps that are described here separately, but in practice that is not common.

::::{margin}
:::{note}
When using [q2-kmerizer](xref:rachis-library-target#q2-plugin-kmerizer), normalization should be done at the ASV level before kmerization, not at the kmer level.
This is automated by the [`kmer-diversity`](xref:rachis-library-target#q2-action-boots-kmer-diversity) action that we're using here.
This is because you are trying to normalize by sequencing depth, not kmer complexity.
In the end, the difference should not be huge but this distinction could be important for some metrics.
:::
::::

While `kmer-diversity` is accessed through q2-boots, the Pipeline leverages other QIIME 2 plugins including [q2-kmerizer](xref:rachis-library-target#q2-plugin-kmerizer) [](doi:10.1128/msystems.01550-24), [q2-vizard](xref:rachis-library-target#q2-plugin-vizard), and [q2-diversity](xref:rachis-library-target#q2-plugin-diversity).
You can learn more about the rarefaction-based workflows in q2-boots and the kmer-based diversity analysis methods in q2-kmerizer in the respective papers about those plugins [](doi:10.12688/f1000research.156295.1) [](doi:10.1128/msystems.01550-24).
The following list describes the steps of the `kmer-diversity` Pipeline, with specific parameters that the user can set presented in monospace font (e.g., `sampling-depth`).

1. Resample the input feature table ([](#fig-feature-table)a) to contain exactly `sampling-depth` sequences per sample, either by bootstrapping or rarefaction, `n` times.
   Samples with fewer than `sampling-depth` sequences will be removed from the feature table and not included in the subsequent steps, as illustrated in [](#fig-even-sampling)c and [](#fig-even-sampling)f.
   This will result in `n` feature tables ([](#fig-boots-kmer-diversity)a).
2. For each feature table resulting from step 1, using the input sequences ([](#fig-feature-table)b), kmerize all sequences (e.g., [](#fig-kmer-features)a) into kmers of length `kmer-size`.[^iab-database-searching]
   Use this information to create one kmer table per resampled feature table (e.g., [](#fig-kmer-features)b).
   This will result in `n` kmer-based feature tables.
3. Compute `alpha-metrics` and `beta-metrics` on each of the kmer tables resulting from step 2.[^forum-diversity-metrics]
   This will result in `n` diversity metric computations per diversity metric ([](#fig-boots-kmer-diversity)b and [](#fig-boots-kmer-diversity)d).
   The metrics computed by default are:
    * Alpha diversity
      * Observed Features (a qualitative measure of community richness; [](#fig-alpha-diversity)a)
      * Shannon's diversity index (a quantitative measure of community richness; [](#fig-alpha-diversity)c)
      * Evenness (i.e., Pielou's Evenness; a measure of community evenness; [](#fig-alpha-diversity)e)
    * Beta diversity
      * Jaccard distance (a qualitative measure of community dissimilarity; [](#fig-beta-diversity)a)
      * Bray-Curtis distance (a quantitative measure of community dissimilarity; [](#fig-beta-diversity)c)
4. For each diversity metric, average its `n` results ([](#fig-boots-kmer-diversity)c and [](#fig-boots-kmer-diversity)e).
   These results can be used in subsequent analysis steps (e.g., ordination, statistical modeling, machine learning).
5. Perform PCoA ordination on the averaged beta diversity distance matrices resulting from Step 4.[^iab-machine-learning]
6. Generate an interactive [q2-vizard scatter plot](xref:rachis-library-target#q2-action-vizard-scatterplot-2d) that contains all user-provided sample metadata, all averaged alpha diversity metrics, and the first three ordination axes for all PCoA matrices computed in step 5 (e.g., [](#fig-kmer-pcoa)).

:::{include} figures/boots-kmer-diversity.md
:::

There are three required parameters that the user must set to define this run.
The most difficult to determine is the value to provide for the `sampling-depth` parameter.
Refer back to [Selecting an even sampling depth](#gut-to-soil-tutorial:selecting-an-even-sampling-depth) above for discussion of this topic.
The user must also provide a value for `n`, the number of iterations of even sampling to run.
You should expect runtime to increase linearly with the setting of this parameter (such that `n=1000` should take 100 times longer to run than `n=10`).
We recommend `n=100` as a good starting point.
Finally, the user must additionally indicate whether each individual resampling step should occur with or without replacement, through the `replacement` parameter.
Sampling without replacement is rarefaction, and is most widely used.
Sampling with replacement is bootstrapping (the origin of the name q2-boots).
These produce nearly identical results [](doi:10.12688/f1000research.156295.1) and we recommend the field does additional work to determine if one or the other of these methods is better in practice.
As of this writing (on 2 October 2026), we recommend sampling without replacement to align with the most commonly used approach.

:::{exercise} Choosing an even sampling depth.
:label: choosing-sampling-depth
View the `asv-table-ms2.qzv` that you created for [](#summarize-asv-table-ms2), and in particular the *Interactive Sample Detail* tab in that visualization.
What value would you choose to pass for `sampling-depth`?
How many samples will be excluded from your analysis based on this choice?
How many total sequences will you be analyzing in the `kmer-diversity` command?
:::

:::{solution} choosing-sampling-depth
:class: dropdown
Importantly, remember that the even sampling depths we're viewing here are extremely low because this is a small subset of a single sequencing run.
At an even sampling depth of 96, we would retain 74 samples (75%) and 7,104 (24%) of our sequences.
At 140 sequences per sample, we would retain 62 samples (63%) and 8,680 (29%) of our sequences.
At 180 sequences per sample, we would retain 55 samples (56%) and 9,900 (33%) of our sequences.
Our alpha rarefaction curve suggests that a higher value (in the 180 range) would be more appropriate because the richness begins to stabilize.
This is likely the most reasonable value to choose, but in this dataset it unfortunately would result in losing nearly 50% of our samples.
For the purpose of the tutorial, we'll select 96 to retain 75% of our samples.
Because we're going to use rarefaction-based diversity calculations here, I'm less concerned about a lower number of sequences per sample.
:::

Let's now run `kmer-diversity` on the tutorial data, which will involve setting the three required parameters mentioned above.
Remember that we're working with a small subset of a full sequencing run in this tutorial, to keep the runtime short for the tutorial.
As a result, the value used for `sampling-depth` here is very low.
Often, this might be closer to 10,000 for an Illumina run (as of 2026), but this is highly dependent on the sequencing run, the number of samples included, and other factors.
Additionally, to keep the runtime short, we set `n` to 10, and to align with the recommendation made earlier, we'll generate the evenly sampled feature tables without replacement.

:::{describe-usage}
use.action(
    use.UsageAction(plugin_id='boots',
                    action_id='kmer_diversity'),
    use.UsageInputs(table=asv_table_ms2,
                    sequences=asv_seqs_ms2,
                    metadata=sample_metadata,
                    sampling_depth=96,
                    n=10,
                    replacement=False),
    use.UsageOutputNames(
        resampled_tables='resampled_tables',
        kmer_tables='kmer_tables',
        alpha_diversities='alpha_diversities',
        distance_matrices='distance_matrices',
        pcoas='pcoas',
        scatter_plot='kmer_diversity_scatter_plot')
)
:::

After computing diversity metrics, we can begin to explore the microbial composition of the samples in the context of the sample metadata.
As you're interpreting the results, remember that q2-kmerizer decomposes each sequence into its constituent kmers.
This should be carefully considered when interpreting alpha diversity in particular, as the number of observed features (for example) would correspond to the number of unique kmers observed in a sample (representing the genetic diversity), not the number of unique sequences or taxa.
For more information, read the q2-kmerizer paper [](doi:10.1128/msystems.01550-24).

:::{include} figures/kmer-pcoa.md
:::

:::{exercise} Learn to use the interactive scatter plot.
:label: scatter-plot
Open the scatter plot that was generated by the previous command, and plot the first two ordination axes computed from the Bray-Curtis distances by selecting them for the *xField* and *yField* dropdowns, respectively.
Cycle through the different metadata columns available in the *colorBy* drop-down; this provides you with the ability to view color-coded sample grouping for any categorical metadata columns in your data.

After exploring *colorBy* groupings, which of the metadata categories results in samples grouped most by color?
:::

:::{solution} scatter-plot
:class: dropdown
The samples appear to exhibit some clustering by `SampleType`.
:::

If you want a more targeted view of any trends you see in your data, you can use the [`lineplot` visualizer](xref:rachis-library-target#q2-action-vizard-lineplot).
This provides a slightly less exploratory view of your data, with a fixed value on the x axis.
You can still cycle through all available numeric metadata columns on the y axis, utilize grouped coloring for all categorical metadata columns, and view the mean or median values for grouped data (if you have replicates in the metadata column you're plotting on the x axis).

:::{exercise} Interpreting ordination plots.
:label: ordination-plots
When plotting Bray-Curtis PCoA axes 1 and 2 and coloring by `SampleType`, are the HEC samples more similar to the food compost or HE samples?

What sample type is the Microbe Mix most similar to?
The inside of the toilet pre-use?
The bulking material?

What other interesting relationships do you see when changing the x- and y-axes and sample coloring?
Which α and β diversity metric outcomes (PCoA and/or alpha diversity results) appear to be correlated with one another?

Which sample has the lowest microbiome richness?
:::

:::{solution} ordination-plots
:class: dropdown
On Axis 1, the HEC samples cluster closely with the Food Compost samples.
These separate from each other on Axis 2.
Because Axis 1 explains more variation than Axis 2, and because there is some overlap of the HEC and Food Compost samples on Axis 2, this suggests that the HEC is more similar to the Food Compost samples than the HE samples.
Whew!

The Microbe Mix sample is most similar to one of the Food Compost samples.
The Inside Toilet Pre Use samples also seem to cluster among the Food Compost samples, though one does appear to cluster clearly with the HE samples.
The Bulking Material samples cluster between the HEC and Food Compost samples.

Bray-Curtis Axis 2 and Jaccard Axis 2 appear to be strongly correlated with observed features (and with each other).
(Because the direction of axes is arbitrary in PCoA, differences in positive and negative correlation are not interesting.)
Bray-Curtis Axis 1 and Jaccard Axis 1 appear to be strongly correlated with each other.
For the HEC samples, there may be a weak negative correlation between Composting Time Point and observed features.

The Microbe Mix has the lowest microbial richness.
This is easily observed by plotting observed features versus itself in this scatter plot.
:::

## Differential abundance testing with ANCOM-BC2

The final analysis step that we'll cover here is identifying individual features that are differentially abundant across sample types in microbiome data, or differential abundance testing, without an *a priori* hypothesis about which feature(s) are differentially abundant.
This is a challenging problem and an open area of research, in part because the number of features observed is generally a lot larger than the number of samples collected.

If you have an *a priori* hypothesis about which feature(s) are differentially abundant across your sample groups, you should test that hypothesis with more traditional distribution comparison methods, remembering to correct for multiple comparisons.
That type of test will be more statistically powerful for testing hypotheses about individual features, but will be too false positive prone to apply to all features in your feature table.

ANCOM-BC2 [](doi:10.1038/s41592-023-02092-7) is a compositionally-aware linear regression model that allows testing for differentially abundant features across sample groups while also implementing bias correction.
This can be accessed using the [`ancombc2` action](xref:rachis-library-target#q2-action-composition-ancombc2) in the [q2-composition plugin](xref:rachis-library-target#q2-plugin-composition), and we'll apply it here to determine which features differ in abundance between our HE, HEC, and Food Compost sample types.

Differential abundance testing with ANCOM-BC2 operates on a feature table that has not undergone even sampling ([](#fig-feature-table)a), and in general you'll want to use the sample metadata ([](#fig-sample-metadata-study)) to filter samples that are irrelevant to the analysis from the feature table.
This can be done using the `where` parameter to the [`filter-samples` action](xref:rachis-library-target#q2-action-feature-table-filter-samples) in q2-feature-table.
The syntax for this parameter is a bit complex, but it's because it is very expressive.
Specifically, it uses [SQL](xref:rachis-glossary-target#term-sql) [where clause](xref:rachis-glossary-target#term-where-clause) syntax.
In the example that follows, we filter samples from our feature table if their sample type in the metadata isn't "Human Excrement Compost" or "Human Excrement" or "Food Compost".
The constructed data in the figures doesn't have any samples with another `SampleType`, but the tutorial data does.

:::{describe-usage}

asv_table_ms2_dominant_sample_types, = use.action(
    use.UsageAction(plugin_id='feature_table',
                    action_id='filter_samples'),
    use.UsageInputs(table=asv_table_ms2,
                    metadata=sample_metadata,
                    where='[SampleType] IN ("Human Excrement Compost", "Human Excrement", "Food Compost")'),
    use.UsageOutputNames(filtered_table='asv_table_ms2_dominant_sample_types'))
:::

Next, we'll apply ANCOM-BC2 to see which ASVs are differentially abundant across those sample types.
We specify a reference level here as this defines what each group is compared against.
Since the focus of this study is HEC, we chose that as the reference level.
That will let us see what ASVs are over- or under-represented in the other two sample groups (Human Excrement and Food Compost) relative to HEC, as HEC defines the "global intercept" that will be measured against.

:::{describe-usage}
ancombc2_results, = use.action(
    use.UsageAction(plugin_id='composition',
                    action_id='ancombc2'),
    use.UsageInputs(table=asv_table_ms2_dominant_sample_types,
                    metadata=sample_metadata,
                    fixed_effects_formula='SampleType',
                    reference_levels=['SampleType::Human Excrement Compost']),
    use.UsageOutputNames(ancombc2_output='ancombc2_results'))
:::

Finally, we'll visualize the results.
Taxonomic annotations ([](#fig-taxonomic-assignment)) can optionally be provided here to aid in interpretation.

:::{describe-usage}
use.action(
    use.UsageAction(plugin_id='composition',
                    action_id='da_barplot'),
    use.UsageInputs(data=ancombc2_results,
                    taxonomy=taxonomy),
    use.UsageOutputNames(visualization='ancombc2-barplot'))
:::

The output of these steps is a diverging barplot, similar to that illustrated in [](#fig-differential-abundance).
This allows you to observe which features are more or less abundant in a specified `SampleType` (in our example) relative to another.
False positive corrected p-values (referred to here as q-values) are presented to help you identify which features are significantly differentially abundant, while log-fold change (LFC) values allow you to interpret the direction and magnitude of that change.

:::{include} figures/differential-abundance.md
:::

:::{warning} Differential abundance testing is easy to get wrong! ☠️
Accurately identifying individual features that are differentially abundant across sample types in microbiome data is a challenging problem and an open area of research, particularly if you don't have an *a priori* hypothesis about which feature(s) are differentially abundant.
A q-value that suggests that you've identified a feature that is differentially abundant across sample groups should be considered a hypothesis, not a conclusion, and you need new samples to test that new hypothesis.

In addition to the methods contained in the [composition plugin](xref:rachis-library-target#q2-plugin-composition), new approaches for differential abundance testing are regularly introduced.
It's worth assessing the current state of the field when performing differential abundance testing to see if there are new methods that might be useful for your data.
If in doubt, consult a statistician.
:::

:::{exercise} Interpreting ANCOM-BC2 results.
:label: ancombc2-results
Which ASV is most enriched in Human Excrement relative to HEC?
Which ASV is most depleted in Human Excrement relative to HEC?
Which ASV is most enriched in Food Compost relative to HEC?
Which ASV is most depleted in Food Compost relative to HEC?
Describe these results in full sentences that indicate the ASV identifier and their taxonomic annotations.
:::

:::{solution} ancombc2-results
:class: dropdown
ASV `c6c3ab4e828fb40d6e05967b7aac9338`, an ASV classified to the *Blautia_A* genus, is most enriched in Human Excrement relative to HEC.
Its log fold change (LFC) was 1.84, and it was significantly different across HE and HEC with a q-value (q) of 0.04.
ASV `2e4f2b53b856c4def6d021d01f5abb70`, an ASV classified to the species level as *Dorea_A longicatena_B*, is most depleted in Human Excrement relative to HEC.
Its LFC was -0.71, and it was not significantly different across HE and HEC (q=1).

ASV `8b5884acc8c736df09c4260b50dc9297`, an ASV classified as *Pseudomonas_E ovata*, is most enriched in Food Compost relative to HEC.
Its LFC was 0.97, and it was not significantly different across HEC and Food Compost after multiple comparisons correction (p=0.04; q=1).
ASV `e335f74033bc634af43ee6baa84fa247`, an ASV classified as GCA-900066495 (a genus in the *Firmicutes_A* phylum), is most depleted in Food Compost relative to HEC.
Its LFC was -1.93, and it was significantly different across HEC and Food Compost with q=0.004.
:::

:::{exercise} Perform ANCOM-BC2 on genera, instead of ASVs.
:label: ancombc2-genera
You might be interested in performing ANCOM-BC2 (or other analyses) with ASVs grouped based on the genus they are derived from, rather than on the ASVs themselves.
This may or may not increase your statistical power—for example, if all organisms in a genus have a similar impact on their host or environment, you may want to know if that genus is differentially abundant across some category of interest.
Take a look at the documentation for the [`collapse` action](xref:rachis-library-target#q2-action-taxa-collapse) in the [q2-taxa plugin](xref:rachis-library-target#q2-plugin-taxa).
How would you use this to collapse our ASV table at the genus level, and then run ANCOM-BC2 on that table?
:::

::::{solution} ancombc2-genera
:class: dropdown

To collapse our ASVs into genera (i.e., level 6 of the GTDB version 202.0 taxonomy), we can use the following command.

:::{describe-usage}
genus_table_ms2_dominant_sample_types, = use.action(
    use.UsageAction(plugin_id='taxa',
                    action_id='collapse'),
    use.UsageInputs(table=asv_table_ms2_dominant_sample_types,
                    taxonomy=taxonomy,
                    level=6),
    use.UsageOutputNames(collapsed_table='genus_table_ms2_dominant_sample_types'))
:::

We can then provide the resulting table as the input to ANCOM-BC2.

:::{describe-usage}
genus_ancombc2_results, = use.action(
    use.UsageAction(plugin_id='composition',
                    action_id='ancombc2'),
    use.UsageInputs(table=genus_table_ms2_dominant_sample_types,
                    metadata=sample_metadata,
                    fixed_effects_formula='SampleType',
                    reference_levels=['SampleType::Human Excrement Compost']),
    use.UsageOutputNames(ancombc2_output='genus_ancombc2_results'))
:::

And finally, we can visualize the results.
Notice that in this case, we're not providing the taxonomy because we've already integrated that information by collapsing at the genus level.

:::{describe-usage}
use.action(
    use.UsageAction(plugin_id='composition',
                    action_id='da_barplot'),
    use.UsageInputs(data=genus_ancombc2_results),
    use.UsageOutputNames(visualization='genus-ancombc2-barplot'))
:::

::::

In addition to the methods contained in the [q2-composition plugin](xref:rachis-library-target#q2-plugin-composition), new approaches for differential abundance testing are regularly introduced.
It's worth assessing the current state of the field when performing differential abundance testing to see if there are new methods that might be useful for your data.

## Ensuring bioinformatics reproducibility and adapting the tutorial workflow for your own data

As a final step in the tutorial, we're going to apply the `rachis` [Provenance Replay](xref:rachis-glossary-target#term-provenance-replay) [](doi:10.1371/journal.pcbi.1011676) functionality to the results that were just generated.
This will generate a script that documents the analysis steps that you ran, and in general this could be submitted as a detailed supplementary methods document with a paper presenting your results.
You can also use the resulting script to adapt the commands presented in this tutorial to your own data, adjusting parameter settings and metadata column headers as is relevant.

Change back to your home directory by running `cd` with no arguments (i.e., simply run: `cd`).
Assuming that you ran all of the steps above in a directory called `gut-to-soil/`, run the following command to generate a template script that you can adapt for your workflow:

% TODO: `qiime tools replay-provenance` has no Usage API equivalent, so this is presented for the command line only.
```shell
qiime tools replay-provenance \
  --in-fp gut-to-soil/ \
  --recurse \
  --out-fp g2s-replayed.bash
```

:::{exercise} Analyze your data provenance through the provenance replay script.
:label: provenance-replay-script
Open the `g2s-replayed.bash` script that was generated by the last command and explore it to find the commands that you ran during the tutorial.
Referring only to this script, remind yourself what trimming and truncation parameters you used for DADA2.
What even sampling depth was used when calling `kmer-diversity`?
What confidence threshold was used when calling `classify-sklearn` (this one is a default value, not one that you provided on the command line)?
Are there commands in this script that you did not run?
If so, what do you think they are?
Notice that all of this information is needed to reproduce your analysis workflow, and you didn't have to keep track of it yourself!
:::

:::{solution} provenance-replay-script
:class: dropdown
The trimming and truncation parameters used for DADA2 are indicated by the lines:

```
qiime dada2 denoise-paired \
  …
  --p-trunc-len-f 250 \
  --p-trunc-len-r 250 \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  …
```

The even sampling depth used when calling `kmer-diversity` is indicated by the lines:

```
qiime boots kmer-diversity \
  …
  --p-sampling-depth 96 \
  …
```

The confidence threshold used when calling `classify-sklearn` is indicated by the lines:

```
qiime feature-classifier classify-sklearn \
  …
  --p-confidence 0.7 \
  …
```

The `qiime tools import` and `qiime demux subsample-paired` commands were not run in this tutorial.
Rather, they were data preparation steps that someone else ran prior to this tutorial.
Because they are in data provenance, it's still possible to know exactly what those commands were and how they could be re-run.
:::

:::{exercise} Analyze your data provenance through rachis-view.
:label: provenance-rachis-view
Load your `gut-to-soil/kmer-diversity-scatter-plot.qzv` file with [`rachis-view`](https://view.rachis.org) and select the *Provenance* tab.
Compare the information presented in that view with the information presented in the provenance replay script generated in this section.
What information is present in `rachis-view` that is not present in the provenance replay script?
What information is present in the provenance replay script that is not present in `rachis-view`?
:::

:::{solution} provenance-rachis-view
:class: dropdown
`rachis-view` includes information on the run time of jobs, and the software and software versions that were installed in the environment when the command was run.
Provenance Replay presents commands formatted for re-running as a script.

Importantly, neither contains input or output file names.
`rachis` doesn't track that information because it's not stable.
For example, a file can be renamed after it is created, which means that a file name (if saved in provenance) would be out of date.
Instead, every `rachis` Result is assigned a UUID which cannot be changed without invalidating the Result.
You can find a Result's UUID in `rachis-view`.
:::

## Conclusion

In this tutorial we presented a reproducible workflow and supporting data for applying amplicon sequencing and bioinformatics tools to study HEC.
This workflow offers insight into the typical steps in an amplicon-sequencing-based microbiome analysis.
The outcomes presented here are those that are most common to all amplicon analysis workflows, and from this point, analysis tends to diverge to focus on study-specific questions.
The QIIME 2 documentation covers an increasingly broad selection of these analyses, and our developer and support community is available to advise as needed.
To explore some of the more study-specific analyses with the tutorial data presented here, a few next steps could be:

1. Build a machine learning classifier that classifies samples according to the three dominant sample types in the feature table that we used with ANCOM-BC2.
   (Hint: see [`classify-samples`](xref:rachis-library-target#q2-action-sample-classifier-classify-samples) in the [q2-sample-classifier plugin](xref:rachis-library-target#q2-plugin-sample-classifier) [](doi:10.21105/joss.00934).)
2. Perform a longitudinal analysis that tracks which taxa change most with time in different buckets.
   (Hint: see [`feature-volatility`](xref:rachis-library-target#q2-action-longitudinal-feature-volatility) using the [q2-longitudinal plugin](xref:rachis-library-target#q2-plugin-longitudinal) [](doi:10.1128/mSystems.00219-18)).
3. Identify a more modern taxonomy classifier using the resources [described earlier](#suboptimal-classifier-explanation) and apply it to the tutorial data.
   How does it change the taxonomic assignments?
   (Here's a [hint](#compare-taxonomic-annotations) on how to compare taxonomic annotations obtained from different classifiers.)
4. The full dataset (five sequencing runs) is available in the gut-to-soil Artifact Repository [](doi:10.5281/zenodo.13887456).
   Download one of the larger sequencing runs (we worked with a small sequencing run that was generated as a preliminary test), and adapt the commands in the provenance replay script to analyze a bigger dataset.
5. The `rachis` developers provide a utility, [artifinder](https://github.com/rachis-org/artifinder), designed to help you find and identify Artifacts that are relevant to your analysis from a directory that might contain a mix of relevant and irrelevant Artifacts and Visualizations.
   Install this, and try it out by following the instructions in the artifinder documentation.
6. `rachis` recently enabled the generation of Reports, which allow for integration of multiple `.qzv`s into one, for easy sharing with colleagues, in manuscripts, and elsewhere.
   Create a report from all of the `.qzv`s that you generated using the following command.
   Then, curate a more specific one that you think presents the most relevant information and interesting results from the tutorial analysis.
   Experiment with customizing your report in Python, as illustrated in [this example](https://gist.github.com/ebolyen/4773c744400072215ebc3161f197956d).

   ```shell
   qiime tools make-report --report-path gut-to-soil-visualizations.qzv \
     $(find gut-to-soil/ -name '.*' -prune -o -name '*.qzv' -print)
   ```

Now that you've completed this tutorial, you should be able to adapt the commands presented here to perform your own microbiome marker gene data analysis.
You can find additional information and learning materials in our documentation starting from the [`rachis-library`](https://library.rachis.org), and if you need additional guidance the [QIIME 2 Forum](https://forum.qiime2.org) is an excellent resource containing over 10 years of questions and answers related to QIIME 2 and microbiome data science.
Thanks for your interest, and we hope to see you on the QIIME 2 Forum!

### A final word on why we chose this tutorial data

A final word on the [data used in this tutorial](#gut-to-soil-tutorial:data): broader adoption of HEC as a waste management strategy would offer wide-ranging benefits, including for fresh water conservation, reduction of environmental contamination, improvement of public health nearly everywhere on Earth, and the advancement of the technologies that will someday enable human settlement off-Earth.
Microbiomes drive the HEC reaction, and we postulate that HEC microbiome science and engineering can help optimize composting conditions for efficiency and safety, support bioprospecting for thermostable biotechnologically relevant enzymes (such as those that can degrade problematic waste materials), and inform accessible protocols for ensuring stringent safety standards are consistently met.

Additionally, we think these data are great for learning.
They embody highly dynamic microbiomes which consistently change from compositions associated with the human microbiome to those that look more like soil microbiomes.
As a result, we hope that regardless of where your interests lie in microbiome science, the techniques and the microbes represented here will be relevant to your work.

As you start your journey in microbiome science we urge you to keep HEC systems in mind.
Broader adoption of HEC technology isn't something we envision happening quickly or universally, but because of the scale of the problems, even small advances can have far-reaching impacts.
If you're interested in HEC-related problems and solutions, feel free to connect with us through the [Compost Microbiome Lab (CML) 🐪](https://caplab.dev).
Find a longer discussion of this [here](https://gut-to-soil-tutorial.readthedocs.io/en/latest/why/).

:::{note} Citation
This tutorial can be cited as:

Caporaso JG, Herman C, Wood C, Gehret L, Simard A, Bolyen E, Dubois B, Bokulich NA, Meilander J.
Microbiome marker gene analysis with QIIME 2: the "gut-to-soil microbiome axis" tutorial.
In review, 2026.
:::

[^iab-database-searching]: kmerization of biological sequences is described in the [*Database Searching* chapter of *An Introduction to Applied Bioinformatics*](https://readiab.org/database-searching.html#kmer-content).
[^iab-machine-learning]: This process is discussed in the [*Machine Learning in Bioinformatics* chapter of *An Introduction to Applied Bioinformatics*](https://readiab.org/machine-learning.html#unsupervised-learning).
[^forum-diversity-metrics]: Learn more about the available metrics in [this QIIME 2 Forum post](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282).
[^classifier-training-defaults]: The two non-default parameters passed to `fit-classifier-naive-bayes` here are `feat_ext__n_features` (default 8192) and `classify__chunk_size` (default 20000).
 `feat_ext__n_features` sets the size of the hashed k-mer feature space: every 7-mer in a reference sequence is hashed into one of this many buckets, so a smaller value means more distinct k-mers share a bucket and the classifier has less information to separate similar taxa.
 `classify__chunk_size` sets how many reference sequences are passed to the classifier at a time during training; each chunk is expanded into a dense sequences × classes label matrix, so this controls a transient memory peak during training.
 `chunk_size` has no effect on the resulting classifier: models trained with different chunk sizes are identical.
 `n_features` does affect the classifier.
 With this reference, reducing it from 8192 to 2048 roughly halves the memory needed to train and to run the classifier, but on the sequences in this tutorial it also reduces the number of ASVs classified to genus level by about a quarter and to species level by about half; the assignments that are made agree with the default classifier's, so the cost is resolution rather than error.
 When training your own classifier, leave both parameters at their defaults unless you run out of memory.
 If you do, note that reducing `chunk_size` alone made little difference to peak memory in our testing; the memory saving comes mainly from `n_features`, and the two together reduce it further.
 Reducing the number of classes (for example, by collapsing the reference taxonomy to genus level with `rescript edit-taxonomy`) or using a pre-trained classifier from the [QIIME 2 Library](https://library.qiime2.org) are alternatives that avoid this trade-off.
 This footnote text, the parameter modifications made for classifier training in the tutorial, and the analyses described here were prepared with AI assistance.

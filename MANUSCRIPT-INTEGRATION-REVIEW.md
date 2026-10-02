# Manuscript integration: review list

Working notes for the `manuscript-integration` branch, which ports the manuscript text into `docs/gut-to-soil-16S-tutorial.md`.
Everything here is for @gregcaporaso to review; delete items as they are settled, and delete the file when the branch merges.
Every `% TODO(greg)` comment in the page source corresponds to an item below.

## Decisions already made (2026-09-23 and 2026-10-02)

- Glossary terms link as `[text](xref:rachis-glossary-target#term-...)`, not bold.
- Citations are `[](doi:...)` references.
- One page: the manuscript's "The data" section and its diversity-metric discussion are dropdowns on the tutorial page, collapsed by default (2026-10-02; they were separate pages on 2026-09-23).
- The install section links to the install docs instead of the Docker walkthrough; figures 1 and 2 stay and say they came from the 2026.7 workshop container.
- Figures are SVG, full width after their first citation; placement is to be decided per figure.
- Version qualifiers are dropped when a sentence states a current default; dated remarks are left as written with a TODO comment.
- Mechanical substitutions are allowed but every one is listed here.
- Captions that say "manuscript" are left as they are.
- `kmer-diversity` keeps individually named outputs; Exercise 17's path follows.
- No online-only exercise survives, so exercise numbers match the manuscript's 1 to 17.

## Changes from the 2026-10-02 manual review (applied, confirm in the build)

- "alongside four example fastq files" became "two" (figure 4 shows two read files).
- The "constructed dataset" link now targets the "Constructed data for figures" heading inside the collapsed "The data" dropdown, so the hover preview should show that subsection rather than the whole dropdown; check the hover, and check what clicking it does while the dropdown is collapsed.
- The `TaxonomicClassifier` paragraph is a `{note}` admonition.
- The "Differential abundance testing is easy to get wrong!" warning moved from the margin to the body, replacing the manuscript's "As a word of caution ..." paragraph, which is therefore no longer on the page.
- The Conclusion's "Now that you've completed this tutorial" paragraph moved ahead of a new H3, "A final word on the tutorial data", over the HEC paragraph. The request said "A final work"; "word" was assumed.
- Plugin-link hovers (`xref:rachis-library-target#...`) showing "Loading..." indefinitely: the library site answers the hover request with a redirect to `amplicon-docs.readthedocs.io/en/stable/references.plugins.<plugin>.json`; whether that redirect target exists and allows cross-origin reads decides it (see the session notes of 2026-10-02). It is a library/amplicon-docs infrastructure matter, not this page's markup, and the same links were on the page before the migration.

- A "Citation" note at the top of the page gives an "In review, 2026" citation with the full author list, in the manuscript's author order; the lead-in sentence and the omission of the publisher's name are Claude's choices.

## To check in the built site

- The two dropdowns ("The data" near the top, "Background: alpha and beta diversity metrics" before the kmer-diversity section) contain H3 headings; check how those appear in the contents sidebar and whether clicking one opens the dropdown.
- The dropdown titles are Claude's wording.
- Figure numbering: with the dropdowns on the page, figures 1 to 21 should match the manuscript; confirm after the build.
- Per-figure placement: every figure is full width after its first citation. Decide which, if any, should move to the margin or sit beside text.
- The `describe-usage` "unexpected body" warnings in `make fast-preview` are pre-existing fast-mode noise.

## Sentences left as written but print- or version-specific (each has a TODO comment)

- "beginning with installation of QIIME 2" in the opening paragraph; the install section is now a link.
- "GTDB version 232.0 (the most recent version, as of this writing)".
- "As of this writing, we recommend sampling without replacement".
- "Often, this might be closer to 10,000 for an Illumina run (as of 2026)".
- "In this chapter, we presented ..." in the Conclusion.
- "Detailed discussion of each metric is beyond the scope of this chapter" in the diversity dropdown.
- Exercise 13 keeps the manuscript's "MicrobeMix"; the metadata value and the solution say "Microbe Mix".
- "Demultiplexed sequence data" is bold in the manuscript but has no glossary entry, so it is plain bold here.

## Mechanical substitutions Claude made

- "HEC" expanded to "human excrement composting (HEC)" at first use in "The data" and in the Exercise 3 solution, because the manuscript's background section is not ported.
- Install section: the Docker walkthrough replaced by one link sentence; "click the button for Terminal" became "In a terminal"; the figure 1 sentence says the screenshot was generated in the 2026.7 workshop container.
- Figure 2 sentence: "viewed directly in the Docker container by clicking on it in the file browser side bar" became "shows this Visualization viewed inside the QIIME 2 2026.7 workshop Docker container"; the JupyterLab right-click download sentence was dropped.
- "Load the visualization (either by double clicking it in the container file browser or by using rachis-view)" became "(for example, by using rachis-view)".
- "default 0.70 in QIIME 2 2026.7" became "default 0.70".
- The q2-vizard 2026.7 versus 2026.10 parenthetical in Exercise 12 was dropped (quoted in the TODO comment).
- Links use `en/latest` and `use.rachis.org` rather than the manuscript's `2026.7` pins and `use.qiime2.org`; footnotes 17, 19, 20 and 21 of the export have a missing slash in their URLs and footnote 37 points at `docs.qiime2.org/2024.10`.
- The even-sampling YouTube link is the manuscript's video with its tracking parameter stripped; the old margin tip with a different video is gone.
- `kmer-diversity` passes `replacement=False`; its outputs are `resampled_tables`, `kmer_tables`, `alpha_diversities`, `distance_matrices`, `pcoas` and `kmer_diversity_scatter_plot` instead of the old `bootstrap_*` names; Exercise 17 cites `gut-to-soil/kmer-diversity-scatter-plot.qzv`.
- `qiime tools annotation-create`, `replay-provenance --recurse`, `make-report` and `mkdir`/`cd` are plain shell blocks, so they render for the command line only.
- Manuscript section headings are H2 on the page, subsections H3; the old upstream/downstream H2s are gone.
- The manuscript's numbered pipeline steps and distance-matrix criteria are Markdown lists; metadata values, parameter names and sample ids are in code font; taxon names are italic; the Exercise 8 taxonomy string is a code block.
- Exercise 5 and 6 titles use the manuscript's "(part 1)" form instead of the old "-- part 1".
- The classifier-training footnote is attached to "We don't recommend changing these from their defaults in practice."
- The `suboptimal-classifier-explanation` label, cited by the Conclusion's next-steps list, sits on the classifier-training paragraph.
- The RESCRIPt "train your own" link points at the RESCRIPt GitHub repository, as in the manuscript's footnote.

## Online-only content: kept and dropped

Kept: the "Why study human excrement composting?" margin tip; the NMDC metadata-standards video tip; the classifier-training footnote on `n_features` and `chunk_size`; the compare-taxonomic-annotations margin tip; the ASV-level-normalization margin note; the IAB and forum footnotes; the ANCOM-BC2 "easy to get wrong" margin warning.

Dropped: the upstream/downstream framing; the `_ms2` provenance tip and the performance tip (folded into manuscript prose); the Read the Docs build-system rationale and footnote; the "Question." tip after `asv-seqs-ms2`; the "Which input table?" exercise (its premise breaks under the manuscript's order); the stats-plugin note and roadmap paragraph; the replay work-in-progress warning; the sequencing-center margin tip (folded into manuscript prose); the old even-sampling video tip.

## Follow-ups outside this branch

- In the figures repository, `make tutorial-figures` regenerates `docs/figures/` from the current figures and captions; re-run it whenever figures or captions change.
- Once this page is deployed, delete the hard-coded departures in the figures repository's `scrape_commands.py` (annotation, make-report, `cd ..`, no-replacement, the renamed section) and re-scrape the command sheet, checking that the `# Command:` slugs are unchanged.

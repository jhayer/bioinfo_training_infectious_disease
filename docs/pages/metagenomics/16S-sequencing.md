---
title: "Building and exploring an ASV table from 16S data using QIIME 2 in Galaxy"
---

::: {.callout-note title="Lesson overview"}
**Questions**

- What are the primary steps involved in generating an ASV table from 16S rRNA gene sequencing data with QIIME 2?
- How does denoising (DADA2) improve the accuracy of microbial community analysis compared to traditional OTU-based methods?
- Why are quality inspection and read truncation important, and how do they impact downstream analyses?
- How can we assign taxonomy to ASVs and explore the diversity of the samples?

**Objectives**

- Import paired-end FASTQ files into QIIME 2 artifacts in Galaxy
- Choose truncation parameters from the read quality profiles
- Denoise the reads with **qiime2 dada2 denoise-paired** to produce an ASV table
- Assign taxonomy with a pre-trained naive Bayes classifier
- Compute and compare alpha and beta diversity between groups of samples
:::

::: {.callout-note title="Sources"}
This tutorial is adapted from the Galaxy Training Network tutorial [Building an amplicon sequence variant (ASV) table from 16S data using DADA2](https://training.galaxyproject.org/training-material/topics/microbiome/tutorials/dada2/tutorial.html) (Bérénice Batut et al.), rewritten to use the **QIIME 2** tools available on [Galaxy Australia](https://usegalaxy.org.au/). It also draws on the [QIIME 2 "Moving Pictures" tutorial](https://docs.qiime2.org/).
:::

# Introduction

The investigation of environmental microbial communities and microbiomes has been revolutionized by the
development of high-throughput amplicon sequencing. In amplicon sequencing a particular genetic locus,
for example the 16S rRNA gene (or a part of it) in bacteria, is amplified from DNA extracted from the community of interest,
and then sequenced on a next-generation sequencing platform. This technique removes the need to culture
microbes in order to detect their presence, and cost-effectively provides a deep census of a microbial community.

To process 16S amplicon sequencing data, several tools exist: **QIIME 2** ([Bolyen et al. 2019](https://doi.org/10.1038/s41587-019-0209-9)), **Mothur**, **Lotus2** and **DADA2** ([Callahan et al. 2016](https://doi.org/10.1038/nmeth.3869)). All these tools can be used in Galaxy. In this tutorial we will use **QIIME 2**.

**QIIME 2** is not a single algorithm but a platform that wraps many methods as *plugins*. Here we will use, among others:

- **q2-dada2**, to denoise the reads with the DADA2 algorithm
- **q2-feature-classifier** ([Bokulich et al. 2018](https://doi.org/10.1186/s40168-018-0470-z)), to assign taxonomy
- **q2-phylogeny** and **q2-diversity**, to build a tree and compute diversity metrics

## QIIME 2 artifacts and visualizations

QIIME 2 does not work directly on FASTQ or tabular files. Every piece of data is stored in an **artifact**, a zipped archive with the extension **`.qza`**. An artifact contains the data, its *semantic type* (what the data represents, e.g. `FeatureTable[Frequency]`) and its *provenance* (the full history of how it was produced).

Results meant to be looked at are stored in **visualizations**, with the extension **`.qzv`**.

| Extension | Name | Contains | How to look at it |
|---|---|---|---|
| `.qza` | Artifact | Data (reads, tables, trees…) | Used as input of other QIIME 2 tools |
| `.qzv` | Visualization | Plots, tables, interactive views | 👁️ (eye) icon in Galaxy, or [view.qiime2.org](https://view.qiime2.org) |

::: {.callout-note title="Viewing a .qzv file"}
In Galaxy, click the 👁️ (eye) icon of a `.qzv` dataset to open it. If it does not display, download it (💾 icon) and drag it onto [view.qiime2.org](https://view.qiime2.org): the file is opened in your browser, nothing is uploaded.
:::

## ASVs instead of OTUs

The output of the denoising step is a feature table of amplicon sequence variants (an ASV table): a matrix with columns corresponding to samples and rows to ASVs, in which the value of each entry is the number of times that ASV was observed in that sample. This table is analogous to the traditional OTU table, except that one gets the actual sequences of the "true" sequence variants instead of rather anonymous and abstract entities called OTU1, OTU2, etc.

The process of amplicon sequencing introduces errors into the sequencing data, and these errors
severely complicate the interpretation of the results. **DADA2** implements an algorithm that models the errors introduced during amplicon sequencing, and uses that error model to infer the true sample composition. As shown in the DADA2 paper and in [further benchmarking](http://benjjneb.github.io/dada2/SotA.html), this method is more sensitive and specific than traditional OTU methods: it detects real biological variation missed by OTU methods while outputting fewer spurious sequences.

In QIIME 2, the whole DADA2 pipeline (filter and trim, learn error rates, denoise, merge pairs, remove chimeras) is run by **one** tool: **qiime2 dada2 denoise-paired**.

## The dataset

The data come from a gut microbiome study of **pediatric ulcerative colitis (UC)**, a chronic inflammatory bowel disease:

> Zuo W, Wang B, Bai X, Luan Y, Fan Y, Michail S, Sun F. *16S rRNA and metagenomic shotgun sequencing data revealed consistent patterns of gut microbiome signature in pediatric ulcerative colitis.* Scientific Reports (2022). [doi:10.1038/s41598-022-07995-7](https://www.nature.com/articles/s41598-022-07995-7)

The authors collected stool samples from **19 children and young adults with UC** and **23 healthy controls** (7–21 years old) at the Children's Hospital of Los Angeles. Each sample was sequenced in two ways: **16S rRNA amplicon sequencing** and **shotgun metagenomic sequencing**, to check whether both approaches give the same picture of the gut microbiome.

| | 16S data used in this tutorial |
|---|---|
| Sample | Human stool (fecal DNA extracted with the QIAamp PowerFecal kit) |
| Target | **V4** region of the 16S rRNA gene |
| Primers | 515FB `GTGYCAGCMGCCGCGGTAA` / 806RB `GGACTACNVGGGTWTCTAAT` |
| Sequencing | Illumina MiSeq, **2 × 150 bp** paired-end |
| Data | NCBI BioProject [PRJNA759642](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA759642) |

### What did the authors do?

Their 16S workflow is very close to the one we will follow:

1. **QIIME 2** with **DADA2** to denoise the reads into ASVs
2. Taxonomy assigned with a **naive Bayes classifier trained on SILVA 138 (99%, 515F/806R region)**, the same classifier we will use
3. **Alpha diversity** with the Shannon index
4. **Beta diversity** with Bray–Curtis distances, PCoA and a **PERMANOVA** test
5. Differential abundance (edgeR) and a random forest model to predict disease status (not covered here)

They found that UC patients have a **lower alpha diversity** (Shannon index) than healthy controls, and that UC samples are **more different from each other** than healthy samples are (higher beta diversity). Several families, such as *Lachnospiraceae*, *Oscillospiraceae*, *Clostridiaceae* and *Akkermansiaceae*, were depleted in UC, while species of the *Enterobacteriaceae* were enriched.

### Our subset

To keep the analysis short, we will use **4 samples** out of the 42:

| SRA run | Sample name |
|---|---|
| [SRR15702564](https://www.ncbi.nlm.nih.gov/sra/SRR15702564) | 012618.SM92 |
| [SRR15702563](https://www.ncbi.nlm.nih.gov/sra/SRR15702563) | 012618.SM97 |
| [SRR15702530](https://www.ncbi.nlm.nih.gov/sra/SRR15702530) | PT.36 |
| [SRR15702529](https://www.ncbi.nlm.nih.gov/sra/SRR15702529) | PT.40 |

<!-- TODO: add the group (UC / healthy control) of each sample -->

::: {.callout-note title="A small dataset"}
With only 4 samples, we can learn every step of the workflow, but statistical tests (e.g. comparing groups) will have very little power. The results will not necessarily reproduce the conclusions of the paper, which were obtained on 42 samples.
:::

# Prepare Galaxy and data

## Create a history

::: {.callout-tip title="History creation"}
1. Create a new history for this analysis
2. Rename it, e.g. `16S QIIME 2`
:::

## Import the reads from SRA

QIIME 2 expects a set of **demultiplexed** FASTQ files: one forward and one reverse file per sample. It also expects that there are no non-biological bases (primers, adapters) in the reads: in this dataset, the primers are not present in the reads.

As in the Galaxy introduction, we download the reads directly from the SRA with their accession numbers.

::: {.callout-tip title="Import the FASTQ files from SRA"}
1. Create a new file with **Upload** → **Paste/Fetch data**, containing the 4 accessions, one per line:

    ```
    SRR15702564
    SRR15702563
    SRR15702530
    SRR15702529
    ```

    Set the datatype to `txt` and name it `SRA accessions`.

2. Open **Faster Download and Extract Reads in FASTQ format from NCBI SRA** with:
    - *"select input type"*: `List of SRA accession, one per line`
    - *"sra accession list"*: `SRA accessions`
3. Click **Run Tool**
4. Rename the `Pair-end data (fasterq-dump)` collection to `Raw Reads`
:::

The tool creates a **list of dataset pairs**: one element per sample, named with its SRA accession (e.g. `SRR15702564`), containing a forward and a reverse file.

::: {.callout-note title="Sample names matter"}
QIIME 2 links reads, tables and metadata through **sample IDs**. Here the sample IDs are the **SRA accessions** (the names of the elements of the collection). We will use exactly the same IDs in the metadata table later.
:::

:::: {.callout-important title="Question: Read lengths" collapse="true"}
Click on the collection and look at the reads of a sample of each batch (`SRR15702564` and `SRR15702530`). Are all reads 150 bp long?

::: {.callout-tip title="Answer" collapse="true"}
No. The two pairs of samples were not sequenced in the same run: the reverse reads of `SRR15702564` and `SRR15702563` are shorter (140 bp) than those of `SRR15702530` and `SRR15702529` (~150 bp). We need to keep this in mind when choosing the truncation lengths: with DADA2, reads **shorter** than the truncation length are discarded.
:::
::::

## Import the collection into QIIME 2

::: {.callout-tip title="Import the reads as a QIIME 2 artifact"}
1. Open the **qiime2 tools import** tool with the following parameters:
    - *"Type of data to import"*: `SampleData[PairedEndSequencesWithQuality]`
    - *"Type of format"*: `CasavaOneEightSingleLanePerSampleDirFmt`
    - *"Import"*: select the `Raw Reads` collection
2. Click **Run Tool**
3. Rename the output `demux.qza`
:::

::: {.callout-note title="Import errors"}
- If the tool does not accept the collection, check that the datasets are `fastqsanger` (or `fastqsanger.gz`) and that the collection is a **list:paired**.
- The exact labels of the import form can differ slightly between versions of the tool. The important choices are always the **type** (`SampleData[PairedEndSequencesWithQuality]`) and a **paired-end per-sample** format.
:::

# Inspect read quality

Before denoising, we look at the quality of the reads to decide where to truncate them.

::: {.callout-tip title="Summarize the reads"}
1. Open **qiime2 demux summarize** with:
    - *"data"*: `demux.qza`
2. Click **Run Tool**
3. Open the output `.qzv` with the 👁️ icon
:::

The visualization has two tabs:

- **Overview**: the number of reads per sample
- **Interactive Quality Plot**: for each position, a box plot of the quality scores of a random subsample of reads, for the forward (left) and reverse (right) reads. Hovering over a position gives the exact quality values.

:::: {.callout-important title="Question: Number of reads per sample" collapse="true"}
In the **Overview** tab:

1. Do all samples have a similar number of reads?
2. Which sample has the fewest reads?

::: {.callout-tip title="Answer" collapse="true"}
1. The number of reads varies between samples, which is common in amplicon sequencing. This is why we will later need to normalise (rarefy) the samples before comparing their diversity.
2. Look at the bottom of the per-sample table, sorted by number of reads.
<!-- TODO: add the sample name and read count from your own run -->
:::
::::

:::: {.callout-important title="Question: Where to truncate?" collapse="true"}
In the **Interactive Quality Plot** tab:

1. Are the forward reads good quality?
2. We generally advise trimming the last few nucleotides to avoid less well-controlled errors that can arise there. What would you recommend as truncation position for the forward reads?
3. Are the reverse reads good quality?
4. What would you recommend as truncation position for the reverse reads?

::: {.callout-tip title="Answer" collapse="true"}
1. The forward reads are good quality.
2. The profiles do not suggest that additional trimming is needed. We will truncate the forward reads at position **240** (trimming the last 10 nucleotides).
3. The reverse reads are of significantly worse quality, especially at the end, which is common in Illumina sequencing. DADA2 incorporates quality information into its error model, but truncating where the quality crashes improves its sensitivity to rare sequence variants.
4. We will truncate the reverse reads at position **160**, where the quality distribution crashes.
:::
::::

::: {.callout-note title="Considerations for your own data"}
Your reads must still overlap after truncation in order to be merged! This tutorial uses 2x250 V4 sequence data (amplicon of ~253 bp), so the forward and reverse reads almost completely overlap and the truncation can be guided by the quality scores only. With a less-overlapping primer set, like V1-V2 or V3-V4, the truncated reads must keep at least **12 nucleotides of overlap** (plus the biological length variation of the amplicon), otherwise most reads will fail to merge.
:::

# Denoising with DADA2

**qiime2 dada2 denoise-paired** runs the whole DADA2 pipeline on each sample:

1. **Filter and trim**: truncate the reads at the chosen positions and remove reads with too many *expected errors*
2. **Learn the error rates**: DADA2 learns a parametric error model from the data itself (every sequencing run has different error rates)
3. **Denoise**: the core algorithm uses this model to infer the true sequences present in each sample
4. **Merge pairs**: forward and reverse denoised reads are aligned and merged into full-length sequences. By default they must overlap by at least 12 bases and be identical in the overlap
5. **Remove chimeras**: chimeras are artefactual sequences made from two "parent" sequences during PCR. Thanks to the accuracy of ASVs, they can be identified as exact combinations of a left and a right segment of two more abundant sequences

::: {.callout-tip title="Denoise the reads"}
1. Open **qiime2 dada2 denoise-paired** with the following parameters:
    - *"demultiplexed_seqs"*: `demux.qza`
    - *"trunc_len_f"*: `240`
    - *"trunc_len_r"*: `160`
    - *"trim_left_f"*: `0`
    - *"trim_left_r"*: `0`
    - Leave the other parameters at their default values (*"max_ee_f"* / *"max_ee_r"*: `2`, *"chimera_method"*: `consensus`, *"pooling_method"*: `independent`)
2. Click **Run Tool**
3. Rename the outputs:
    - `table.qza`: the ASV table (`FeatureTable[Frequency]`)
    - `rep-seqs.qza`: the sequence of each ASV (`FeatureData[Sequence]`)
    - `denoising-stats.qza`: the number of reads kept at each step
:::

::: {.callout-note title="The DADA2 parameters"}
| Parameter | Meaning |
|---|---|
| `trunc_len_f` / `trunc_len_r` | Position at which forward / reverse reads are truncated. Reads shorter than this are discarded |
| `trim_left_f` / `trim_left_r` | Number of bases removed at the start of the reads (e.g. to remove primers) |
| `max_ee_f` / `max_ee_r` | Maximum number of **expected errors** allowed in a read, [a better filter than averaging quality scores](https://academic.oup.com/bioinformatics/article/31/21/3476/194979) |
| `chimera_method` | How chimeras are detected: `consensus` (per sample, then consensus), `pooled`, or `none` |
| `pooling_method` | `independent` processes each sample separately; `pseudo` shares information between samples to detect rare variants |

These values are starting points, not set in stone. If too few reads pass the filter, consider relaxing `max_ee_r` (e.g. to `5`) or truncating the reverse reads more, while keeping enough overlap to merge.
:::

## Track reads through the pipeline

As a sanity check, we look at the number of reads that made it through each step.

::: {.callout-tip title="Inspect the denoising statistics"}
1. Open **qiime2 metadata tabulate** with:
    - *"input"*: `denoising-stats.qza`
2. Click **Run Tool** and open the `.qzv` output
:::

The table contains, for each sample, the number of reads: `input`, `filtered`, `denoised`, `merged` and `non-chimeric`, with the percentage kept.

:::: {.callout-important title="Question: Where are reads lost?" collapse="true"}
1. Have we kept the majority of the raw reads for each sample?
2. Is there any exceptionally large drop associated with a single step for any sample?

::: {.callout-tip title="Answer" collapse="true"}
1. Yes, most of the raw reads are kept as non-chimeric reads.
2. No. The largest loss is at the filtering step, which is expected. Merging and chimera removal only remove a small fraction of reads.
<!-- TODO: add the percentages from your own run -->
:::
::::

::: {.callout-note title="What if many reads are lost?"}
Outside of filtering, there should be no step in which a majority of reads are lost.

- If most reads **failed to merge**, the truncation lengths are probably too short: the reads no longer overlap.
- If most reads were **removed as chimeric**, primers were probably not removed: ambiguous nucleotides in primers interfere with chimera identification. Use `trim_left_f` / `trim_left_r`, or **qiime2 cutadapt trim-paired** before denoising.
:::

## Summarise the ASV table and sequences

::: {.callout-tip title="Summarise the feature table and the ASVs"}
1. Open **qiime2 feature-table summarize** with:
    - *"table"*: `table.qza`
2. Open **qiime2 feature-table tabulate-seqs** with:
    - *"data"*: `rep-seqs.qza`
3. Open both `.qzv` outputs
:::

:::: {.callout-important title="Question: Exploring the ASV table" collapse="true"}
1. How many ASVs (features) have been identified?
2. What is the total number of reads (frequency) in the table?
3. What is the range of ASV lengths? Does it make sense for a V4 amplicon?

::: {.callout-tip title="Answer" collapse="true"}
1. and 2. In the **Overview** tab of the `feature-table summarize` visualization, look at *Number of features* and *Total frequency*.
3. In the `tabulate-seqs` visualization, the *Sequence Length Statistics* show lengths around **251–255 bp**, the expected size of the V4 region.
<!-- TODO: add the number of features and total frequency from your own run -->
:::
::::

The **Interactive Sample Detail** tab of the `feature-table summarize` visualization shows the number of reads per sample after denoising. **Keep it open**: we will use it to choose the rarefaction depth for the diversity analysis.

::: {.callout-note title="Sequences too long or too short"}
Sequences much longer or shorter than expected may be the result of non-specific priming. They can be removed with **qiime2 feature-table filter-seqs** / **filter-features**, the "cutting a band" in-silico equivalent.
:::

# Assign taxonomy

It is common at this point, especially in 16S/18S/ITS amplicon sequencing, to assign a taxonomy to the sequence variants. QIIME 2 uses a **naive Bayes classifier** (the same family of method as the RDP classifier) trained on a reference database of sequences with known taxonomy.

The classifier works best when it is trained on the exact region amplified by your primers. Pre-trained classifiers are provided by QIIME 2 for **SILVA** and **Greengenes2**, both for full-length 16S and for the **V4 515F/806R** region used in this dataset.

::: {.callout-tip title="Get a pre-trained classifier"}
1. Go to the [QIIME 2 data resources](https://docs.qiime2.org/) page **for the QIIME 2 version installed on Galaxy Australia** (shown in the help section of the QIIME 2 tools)
2. Copy the link of the **Silva 138 99% OTUs from 515F/806R region of sequences** naive Bayes classifier (`.qza`)
3. Upload it in Galaxy with **Upload** → **Paste/Fetch data**, datatype `qza`
4. Rename it `classifier.qza`
:::

::: {.callout-note title="Classifier version"}
A classifier only works with the version of QIIME 2 (more precisely of scikit-learn) it was trained with. If **classify-sklearn** fails with a version error, download the classifier matching the QIIME 2 version of the Galaxy tool. If your server provides a built-in reference classifier, you can use it instead of uploading one.
:::

::: {.callout-tip title="Assign taxonomy"}
1. Open **qiime2 feature-classifier classify-sklearn** with:
    - *"reads"*: `rep-seqs.qza`
    - *"classifier"*: `classifier.qza`
2. Click **Run Tool** and rename the output `taxonomy.qza`
3. Open **qiime2 metadata tabulate** with:
    - *"input"*: `taxonomy.qza`
4. Open the `.qzv` output
:::

The table lists, for each ASV, its taxonomy (`d__` domain, `p__` phylum, `c__` class, `o__` order, `f__` family, `g__` genus, `s__` species) and the **confidence** of the assignment.

:::: {.callout-important title="Question: Taxonomic assignments" collapse="true"}
1. Which domain(s) have been identified?
2. Which phyla are the most represented?
3. Are many ASVs assigned down to species level? Why?

::: {.callout-tip title="Answer" collapse="true"}
1. Bacteria.
2. **Firmicutes** (Bacillota) and **Bacteroidetes** (Bacteroidota) are well represented, which is expected for fecal samples.
3. Few ASVs have a species assignment, both because it is often not possible to distinguish species from a ~250 bp fragment of the 16S gene, and because the indigenous mouse gut microbiota is poorly covered by reference databases.
:::
::::

::: {.callout-note title="Wrong kingdom"}
If your reads do not seem to be appropriately assigned, for example lots of bacterial 16S sequences assigned as `Eukaryota` or `Unassigned`, check that the classifier matches your amplified region and that your reads are in the same orientation as the reference database.
:::

# Prepare the metadata table

To compare groups of samples, QIIME 2 needs a **metadata table**: a tab-separated file whose first column contains the sample IDs.

The samples are named by the gender (`F`), mouse subject number (`3`) and day post-weaning (`D0`): `F3D0`. We create a table with these factors and an extra column `when`, which is `Early` (day < 100) or `Late` (day > 100).

::: {.callout-tip title="Create the metadata table"}
1. Click **Upload** → **Paste/Fetch data** and paste the following table:

    ```
    sample-id	gender	mouse	day	when
    F3D0	Female	3	0	Early
    F3D1	Female	3	1	Early
    F3D2	Female	3	2	Early
    F3D3	Female	3	3	Early
    F3D5	Female	3	5	Early
    F3D6	Female	3	6	Early
    F3D7	Female	3	7	Early
    F3D8	Female	3	8	Early
    F3D9	Female	3	9	Early
    F3D141	Female	3	141	Late
    F3D142	Female	3	142	Late
    F3D143	Female	3	143	Late
    F3D144	Female	3	144	Late
    F3D145	Female	3	145	Late
    F3D146	Female	3	146	Late
    F3D147	Female	3	147	Late
    F3D148	Female	3	148	Late
    F3D149	Female	3	149	Late
    F3D150	Female	3	150	Late
    ```

2. In the upload settings, set the datatype to `tabular` and check **Convert spaces to tabs** (in case the tabs were replaced by spaces when copying)
3. Rename it `Metadata table`
4. Check it with **qiime2 metadata tabulate**, using `Metadata table` as input
:::

::: {.callout-note title="Metadata rules"}
- The first column must be named `sample-id` (or `id`, `#SampleID`…) and contain exactly the sample IDs used in the artifacts.
- Columns with only numbers (like `day`) are treated as **numeric**, the others as **categorical**.
- See the [QIIME 2 metadata documentation](https://docs.qiime2.org/) for the complete format.
:::

# Visualise the taxonomic composition

::: {.callout-tip title="Taxonomy barplot"}
1. Open **qiime2 taxa barplot** with:
    - *"table"*: `table.qza`
    - *"taxonomy"*: `taxonomy.qza`
    - *"metadata"*: `Metadata table`
2. Open the `.qzv` output
3. Set:
    - *"Taxonomic Level"*: `Level 5` (family)
    - *"Sort Samples By"*: `when`
:::

:::: {.callout-important title="Question: Taxonomic composition" collapse="true"}
Is there an obvious change in taxonomy between the early and late samples?

::: {.callout-tip title="Answer" collapse="true"}
Not really: the same main families are found in early and late samples, with variations in their relative abundance.
:::
::::

::: {.callout-note title="Heatmaps instead of barplots"}
Stacked barplots become hard to read with many taxa. A heatmap is often clearer: collapse the table at family level with **qiime2 taxa collapse** (*"level"*: `5`), then run **qiime2 feature-table heatmap** on the collapsed table with the metadata column `when`.
:::


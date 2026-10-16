---
title: "SARS-CoV-2 genome reconstruction and lineage assignment from ONT tiled-amplicon data"
---

::: {.callout-note title="Lesson overview"}
**Questions**

- How can a complete analysis, including viral consensus sequence reconstruction and lineage assignment, be performed on a batch of samples?
- How are tiled-amplicon (ARTIC) ONT data handled differently from other sequencing data?
- What are key results beyond consensus genomes and lineage assignments that need to be understood to avoid inappropriate conclusions about samples?

**Objectives**

- Obtain the recommended Galaxy workflows for SARS-CoV-2 ONT tiled-amplicon data
- Run a workflow to discover mutations in a batch of viral samples amplified with the ARTIC v4 primers
- Run a workflow to summarise and visualise the mutations of a batch of samples
- Run a workflow to construct viral consensus sequences
- Assign the samples to lineages with **Pangolin** and **Nextclade**
- Combine information from different analysis steps to draw appropriate conclusions about individual samples and batches
:::

::: {.callout-note title="Sources"}
This tutorial is adapted from the Galaxy Training Network tutorial [Mutation calling, viral genome reconstruction and lineage/clade assignment from SARS-CoV-2 sequencing data](https://training.galaxyproject.org/training-material/topics/variant-analysis/tutorials/sars-cov-2-variant-discovery/tutorial.html) (Wolfgang Maier, Bérénice Batut), restricted to the analysis of **tiled-amplicon ONT data**.
:::

# Introduction

Sequence-based monitoring of global infectious disease crises, such as the COVID-19 pandemic, requires capacity to generate and analyse large volumes of sequencing data in near real time. These data have proven essential for surveilling the emergence and spread of new viral variants, and for understanding the evolutionary dynamics of the virus.

For efficient genome surveillance, we want to:

- go beyond per-sample mutation calls in variant call format (VCF): to keep track of large numbers of samples sequenced in batches, we need concise reports and visualisations of the results at the sample and batch level
- use the mutations of each sample to construct its consensus genome
- use the consensus genomes to assign the samples to SARS-CoV-2 lineages, as defined by the major lineage classification systems ([Nextstrain clades](https://nextstrain.org/blog/2022-04-29-SARS-CoV-2-clade-naming-2022) and [PANGO](https://www.pango.network/))
- decrease hands-on time and data manipulation errors by combining analysis steps into workflows

In this tutorial, we use a set of workflows developed by the [Galaxy Covid-19 project](https://galaxyproject.org/projects/covid19/), together with a handful of additional tools:

![Analysis flow in the tutorial. We follow the "tiled-amplicon ONT" branch.](images/sars-cov-2/sars-cov2-wf.png)

1. The **variation analysis** workflow discovers the mutations of each sample of the batch. Different versions of this workflow exist for different types of data; we use the one for **tiled-amplicon ONT** data.
2. The **variation reporting** workflow generates per-sample and batch-level reports and visualisations of the mutations.
3. The **consensus construction** workflow reconstructs the full viral genome of each sample by modifying the SARS-CoV-2 reference genome with the mutations of the sample.
4. **Pangolin** and **Nextclade** assign the consensus genomes to lineages.

A few highlights of these workflows:

- The reporting workflow produces a batch-level overview plot of the mutations and their allele frequencies, which helps to spot batch effects such as sample cross-contamination, and outlier samples that are different from the rest of the batch.
- The consensus workflow expresses uncertainty about any base position of the consensus sequence by masking it with an **N**, according to user-defined thresholds.
- All the workflows are openly developed and available as defined releases through public workflow registries.

## Tiled-amplicon sequencing with ARTIC primers

The SARS-CoV-2 genome is ~30 kb long. In clinical samples, viral RNA is often present in low amounts and mixed with a lot of human RNA. To sequence it efficiently, the viral genome is first amplified by **multiplex PCR** with a set of primers producing short, overlapping amplicons that together tile the whole genome.

The most widely used primer sets were designed by the [ARTIC network](https://artic.network/). With **ARTIC v4**, the genome is covered by ~100 amplicons of ~400 bp, split into two pools so that overlapping amplicons are not amplified in the same reaction. ARTIC v4 was released in 2021 to restore the amplification of regions where mutations of new variants (e.g. Delta) prevented some v3 primers from binding.

This approach has consequences for the analysis:

- **The primer sequences must be removed** from the reads: they come from the synthetic primers, not from the virus, and would hide the mutations located in primer binding sites. To do so, the workflow needs to know the positions of the primers on the genome (a **BED** file).
- Reads that look like duplicates are expected, because all reads of an amplicon start and end at the same positions. Unlike for whole-genome sequencing data, they are **not removed**.
- With **ONT** sequencing, each read usually covers a whole amplicon. Reads much shorter or longer than an amplicon (e.g. chimeric reads formed by two ligated amplicons) can be removed by filtering on read length.

## Analysis of ONT data

The workflow for ONT tiled-amplicon data runs:

- **fastp** for quality control, and filtering of reads on quality and length
- **minimap2** to map the reads to the reference genome
- **ivar trim** to remove the primer sequences from the mapped reads, using the primer BED file
- **medaka** to call the mutations, with a model of the errors of ONT sequencing
- **SnpEff** (its 4.5covid19 version) to annotate the effects of the mutations on the viral genes and proteins

# Prepare Galaxy and data

## Prepare a new Galaxy history

::: {.callout-tip title="History creation"}
1. Create a new history for this analysis
2. Rename it, e.g. `SARS-CoV-2 ONT`
:::

## Get sequencing data

We will get the sequencing data from [a household cohort study published in the Lancet Southeast Asia](https://pmc.ncbi.nlm.nih.gov/articles/PMC12810560).
If we look at the data sharing in the paper, we can see that the data is available under [the ENA Project PRJEB104853](https://www.ebi.ac.uk/ena/browser/view/PRJEB104853).

The workflow expects a **list** collection with one FASTQ file per sample (ONT reads are single reads, not pairs). You need at least two samples, sequenced with ONT after amplification with the ARTIC v4 primers.

We will take the first 3 

::: {.callout-tip title="Import the reads from SRA"}
1. Create a new file with **Upload** → **Paste/Fetch data**, containing the SRA accessions of the samples, one per line:

    ```
    ERR16054783
    ERR16054815
    ERR16054829  
    ERR16054845
    ERR16054945
    ERR16054982
    ERR16055005
    ERR16055067
    ERR16055110
    ```

    Set the datatype to `txt` and name it `SRA accessions`.

2. Open **Faster Download and Extract Reads in FASTQ format from NCBI SRA** with:
    - *"select input type"*: `List of SRA accession, one per line`
    - *"sra accession list"*: `SRA accessions`
3. Click **Run Tool**
4. Rename the `Single-end data (fasterq-dump)` collection to `ONT reads`
:::

::: {.callout-note title="Using your own data"}
If your reads are on your computer, upload them with **Upload** → **Collection**, with *"Collection Type"*: `List` and *"File Type"*: `fastqsanger.gz`. If your samples were amplified with a primer set other than ARTIC v4, you will need the primer BED file matching it (see below).
:::

## Import the reference sequence and auxiliary datasets

Besides the reads, we need The reference sequence to call and annotate the mutations:

- the SARS-CoV-2 reference sequence, to map the reads and compare our samples against. We use the original Wuhan-Hu-1 genome, known as [MN908947.3](https://www.ebi.ac.uk/ena/browser/view/MN908947.3) in the INSDC databases and as [NC_045512.2](https://www.ncbi.nlm.nih.gov/nuccore/NC_045512.2) in NCBI RefSeq


::: {.callout-tip title="Get the reference sequence"}
1. Upload the reference sequence with **Upload** → **Paste/Fetch data**, using this link, and set the datatype to `fasta`:

    ```
  
    https://raw.githubusercontent.com/pha4ge/primer-schemes/main/schemes/artic-sars-cov-2/400/v5.3.2/primer.bed
    https://zenodo.org/record/5888324/files/NC_045512.2_reference.fasta
   https://zenodo.org/record/5888324/files/NC_045512.2_feature_mapping.tsv
    ```
2. Click on "Start". 

:::

You can also retrieve the reference sequence on [NCBI](https://www.ncbi.nlm.nih.gov/nuccore/NC_045512)   . 

#
# Analysis

The analysis consists of a workflow

## Import the workflows

All workflows of the Galaxy Covid-19 project are published by the **IWC** (Intergalactic Workflow Commission) on [Dockstore](https://dockstore.org/) and [WorkflowHub](https://workflowhub.eu/), and can be imported directly into Galaxy.

::: {.callout-tip title="Import the workflows"}
1. In Galaxy, click on **Workflows** in the top menu, then **Public Workflows**
2. Type "COVID-19 variation analysis of ARTIC ONT data  Version 1", or click on the [workflow link](https://usegalaxy.org.au/published/workflow?id=47ce6a2d30887637)
3. Click on "Import Worflow".  
:::

## From sequencing data to annotated mutations per sample

::: {.callout-tip title="Run the variation analysis workflow"}
1. Open the **COVID-19: variation analysis of ARTIC ONT data** workflow and set:
    - *"ONT-sequenced reads"*: `ONT reads`
    - *"NC_045512.2 FASTA sequence of SARS-CoV-2"*: `SARS-CoV-2 reference`
    - *"Primer binding sites info in BED format"*: `ARTIC v4 primers`
2. Click **Run Workflow**
:::

::: {.callout-note title="Read length filtering"}
The optional parameters *"Minimum read length"* and *"Maximum read length"* should match the amplicon sizes of the primer scheme. The workflow defaults are appropriate for the ARTIC primers. For another primer scheme, the [ARTIC network's recommendations](https://artic.network/ncov-2019/ncov2019-bioinformatics-sop.html) (section "Read filtering") are a good starting point: set the minimum to the size of the smallest amplicon, and the maximum to the size of the largest amplicon plus 200 nucleotides, to exclude obviously chimeric reads.
:::

The workflow takes a while to complete. One of the last datasets added to the history is the **Preprocessing and mapping reports**, the first overview report of the analysis. It contains statistics on the quality of the reads and on the **coverage** of the reference genome by the mapped reads. Technical issues with some samples of a batch are typically visible in this report already, and spotting them early prevents overinterpreting later reports.

:::: {.callout-important title="Question: Preprocessing and mapping" collapse="true"}
1. Are there samples with a much lower number of reads or a lower coverage than the others?
2. In the *Cumulative genome coverage* plot, which fraction of the genome is covered by at least 20 reads in each sample?

::: {.callout-tip title="Answer" collapse="true"}
1. Look at the *General Statistics* section. Samples with a critically low coverage will give unreliable mutation calls, and probably incomplete consensus sequences.
2. A curve that drops early means that large parts of the genome are poorly covered, for example because some amplicons failed.
<!-- TODO: add the problematic samples from your own run -->
:::
::::

## From mutations per sample to reports and visualisations

Open the [variant analysis reporting workflow](https://usegalaxy.org.au/published/workflow?id=f3a804758d5d9252). 

The main output of the first workflow is a collection with the annotated mutations of each sample. It is the input of the **reporting** workflow.

::: {.callout-tip title="Run the reporting workflow"}
1. Open the **COVID-19: variation analysis reporting** workflow and set:
    - *"Variation data to report"*: `Final (SnpEff-) annotated variants`
    - *"gene products translations"*: `SARS-CoV-2 feature mapping`
    - *"Number of Clusters"*: `3`
2. Click **Run Workflow**
:::

::: {.callout-warning title="Use the right collection of annotated variants"}
The variation analysis workflow generates **two** collections of annotated variants: `Final (SnpEff-) annotated variants`, and `Final (SnpEff-) annotated variants with strand-bias soft filter applied`. For tiled-amplicon data, the strand-bias filter is experimental: use `Final (SnpEff-) annotated variants`.
:::

*"Number of Clusters"* sets the number of main clusters of samples in the variant frequency plot. With a small batch, try a lower value; generating an optimal plot sometimes requires some experimenting, and you can rerun only the job that generates the plot with another value. The other parameters (*"AF Filter"*, *"DP Filter"*, *"DP_ALT_FILTER"*) define the criteria to report a mutation as reliable and can be left at their default values.

The three key results of the reporting workflow are:

1. **Combined Variant Report by Sample**: one line per mutation detected in a sample, with its key statistics. It is a faster and more legible way to study the mutations than the per-sample VCF files.

    | Column | Field | Meaning |
    |---|---|---|
    | 1 | `Sample` | Sample ID |
    | 2 | `POS` | Position of the mutation on the reference genome |
    | 3 | `FILTER` | Whether the mutation passed the *AF*, *DP* and *DP_ALT* filters in this sample |
    | 4–5 | `REF` / `ALT` | Reference and alternative bases |
    | 6 | `DP` | Sequencing depth at the mutation site |
    | 7 | `AF` | Allele frequency: fraction of the reads carrying the mutation |
    | | `IMPACT`, `FUNCLASS`, `EFFECT` | Impact and effect of the mutation (from SnpEff) |
    | | `GENE`, `CODON`, `AA` | Affected gene, codon and amino acid change (from SnpEff) |
    | | `TRID` | Name of the affected viral protein (from the feature mapping) |
    | | `min(AF)` / `max(AF)` | Minimum and maximum allele frequency of this mutation across all samples |

2. **Combined Variant Report by Variant**: one line per mutation, aggregated *across* samples, with, among others, the number of samples carrying it (`countunique(Sample)`), the list of samples in which it passed the filters (`SAMPLES(above-thresholds)`), the list of all samples in which it was found (`SAMPLES(all)`) and the corresponding allele frequencies (`AFs(all)`).

3. **Variant frequency plot**: a heatmap of the allele frequencies (cell color) of the mutations (columns) in the samples (rows). The mutations are grouped by viral gene (colors of the first row), and their impact on the viral proteins is color-coded on the second row. The tree on the left shows the clustering of the samples.

:::: {.callout-important title="Question: Exploring the reports" collapse="true"}
1. Are there samples outside of the main cluster(s) of the variant frequency plot? Did you already notice them in the **Preprocessing and mapping reports**?
2. Is there a sample in which most mutations have an intermediate allele frequency (light colors), instead of a frequency close to 1 (black)? What could explain it?
3. How would you list the mutations found **only** in one sample?

::: {.callout-tip title="Answer" collapse="true"}
1. Samples with a low coverage often appear as outliers: mutations may be missing from them only because there were not enough reads to detect them, not because the virus does not carry them. We will check this with the consensus sequences.
2. A sample of a single virus should carry its mutations on (almost) all reads. Many mutations at intermediate frequencies suggest a **mixture** of viruses: a contamination with material from another sample of the batch, or a co-infection of the patient with two lineages.
3. Use **Filter data on any column using simple expressions** on the **Combined Variant Report by Variant**, with *"With following condition"*: `c15 == "SAMPLE_ID"` (column 15 is `SAMPLES(all)`) and *"Number of header lines to skip"*: `1`.
<!-- TODO: add the observations from your own run -->
:::
::::

## From mutations per sample to consensus sequences

Here is the link to the last workflow, the [COVID-19 CONSTRUCTION WORKFLOW](https://usegalaxy.org.au/published/workflow?id=61a21c9928182ae4). 

The last workflow generates consensus sequences according to transparent criteria:

- Each consensus sequence contains all the filter-passing mutations of its sample that reach a user-defined **consensus allele frequency** threshold.
- Filter-failing mutations, and mutations below a second, **minimal allele frequency** threshold, are ignored.
- Positions of filter-passing mutations with an allele frequency between the two thresholds are masked with **N**.
- Positions with a coverage below a **depth threshold** are masked with **N** too, unless they are sites of consensus mutations.

::: {.callout-tip title="Run the consensus construction workflow"}
1. Open the **COVID-19: consensus construction** workflow and set:
    - *"Variant calls"*: `Final (SnpEff-) annotated variants`
    - *"aligned reads data for depth calculation"*: `Fully processed reads for variant calling` (the BAM files produced by the first workflow)
    - *"Reference genome"*: `SARS-CoV-2 reference`
2. Leave *"min-AF for consensus variant"*, *"min-AF for failed variants"* and *"Depth-threshold for masking"* at their default values
3. Click **Run Workflow**
:::

The main outputs of the workflow are:

- a collection of consensus sequences, useful to look at the consensus sequence of one sample
- a **Multisample consensus FASTA** with all sequences, used as input for **Pangolin** and **Nextclade**

### Exploring consensus sequence quality

Not all consensus sequences have the same quality: uncertain mutations and low coverage lead to N-masking of single positions or of whole stretches of the genome. As Ns are hard to find by scrolling through a FASTA file, we report their positions explicitly.

::: {.callout-tip title="Report masked positions"}
1. Open **Fasta regular expression finder** with:
    - *"Input"*: the `Consensus sequence with masking` collection produced by the consensus workflow
    - *"Regular expression"*: `N+` (stretches of one or more Ns)
    - In *"Specify advanced parameters"*:
        - *"Do not search the reverse complement"*: `Yes` (an N is an N on both strands)
        - *"Maximum length of the match to report"*: `1` (to report the stretches of Ns concisely)
2. Click **Run Tool**
:::

:::: {.callout-important title="Question: Masked positions" collapse="true"}
1. Do the samples with a low coverage have more and longer stretches of Ns?
2. Do the N stretches have a typical size? Why?
3. For an outlier sample of the variant frequency plot, are the "missing" mutations really absent, or are they in N-masked regions?

::: {.callout-tip title="Answer" collapse="true"}
1. Yes: a low coverage leads to large masked regions, sometimes most of the genome.
2. With tiled-amplicon data, a masked stretch often corresponds to one or several **failed amplicons** (~400 bp with ARTIC v4), for example because a mutation prevents a primer from binding.
3. If a mutation falls in an N-masked region, it cannot be determined: there is no objective reason to think that the sample lacks it, and therefore that it belongs to a different lineage.
<!-- TODO: add the observations from your own run -->
:::
::::

## From consensus sequences to lineage assignments

Two tools assign lineages to the samples from their consensus sequences: **Pangolin** and **Nextclade**.

### Lineage assignment with Pangolin

**Pangolin** (Phylogenetic Assignment of Named Global Outbreak LINeages) assigns a SARS-CoV-2 genome to its most likely lineage of the [PANGO nomenclature](https://www.pango.network/).

::: {.callout-tip title="Assign lineages with Pangolin"}
1. Open **Pangolin** with:
    - *"Input FASTA File(s)"*: `Multisample consensus FASTA`
    - *"Include header line in output file"*: `Yes`
2. Inspect the output
:::

Pangolin generates a table with one line per sample. The columns are explained in the help of the Galaxy tool and in the [pangolin documentation](https://cov-lineages.org/resources/pangolin/output.html).

With many samples, a summary of the observed lineages is useful.

:::: {.callout-important title="Question: Lineage summary" collapse="true"}
Can you configure **Datamash** to produce a three-column lineage summary from the pangolin output, in which each line lists:

- an assigned lineage (column 2 of the pangolin output),
- the number of samples with that lineage,
- the comma-separated identifiers of these samples (column 1 of the pangolin output)?

::: {.callout-tip title="Answer" collapse="true"}
Configure **Datamash** like this:

- *"Input tabular dataset"*: the output of Pangolin
- *"Group by fields"*: `2` (we group by lineage)
- *"Sort input"*: `Yes` (grouping only works on data sorted by the grouping values)
- *"Input file has a header line"*: `Yes`
- *"Print header line"*: `Yes`
- In *"Operation to perform on each group"*:
    - *"Type"*: `count`, *"On column"*: `Column: 1`
    - *"Type"*: `Combine all values`, *"On column"*: `Column: 1`
:::
::::

:::: {.callout-important title="Question: Pangolin results" collapse="true"}
1. Which lineage(s) were assigned to most samples?
2. Were some samples left unassigned? Why?
3. Is the assignment of a sample with many mutations at intermediate allele frequencies reliable?

::: {.callout-tip title="Answer" collapse="true"}
1. Read it from the lineage summary.
2. Look at the `qc_status` and `qc_notes` columns: Pangolin refuses to assign a lineage when the proportion of Ns (*Ambiguous content*) is above the *"Maximum proportion of Ns allowed"* (`0.3` by default), i.e. when there is not enough sequence information.
3. Not necessarily. Only the mutations above the consensus allele frequency threshold are in the consensus sequence. For a mixed sample, these may be only the mutations shared by both viruses (e.g. ancestral mutations), and Pangolin may then assign an ancestral lineage that does not reflect either virus.
<!-- TODO: add the lineages from your own run -->
:::
::::

### Lineage assignment with Nextclade

**Nextclade** assigns clades, calls mutations and checks the quality of SARS-CoV-2 genomes.

::: {.callout-tip title="Assign clades with Nextclade"}
1. Open **Nextclade** with:
    - *"FASTA file with input sequences"*: `Multisample consensus FASTA`
    - *"Version of database to use"*: `Download latest available database version from web`
    - *"Output options"*: `Tabular format report`
    - *"Include header line in output file"*: `Yes`
2. Inspect the output
:::

Nextclade assigns lineages with an algorithm different from Pangolin's, and uses its own nomenclature of **clades**. It also translates its clades into PANGO lineage names and into the coarser **WHO** classification (e.g. Delta, Omicron). The details of the clade assignment are explained in the [Nextclade documentation](https://docs.nextstrain.org/projects/nextclade/en/stable/user/algorithm/06-clade-assignment.html#clade-assignment).

:::: {.callout-important title="Question: Nextclade summary" collapse="true"}
Can you configure **Datamash** to summarise the Nextclade results by `Nextclade_pango` (column 3), including the `clade` (column 2) and `clade_who` (column 6) columns, with the number and identifiers of the samples, to compare them with the Pangolin summary?

::: {.callout-tip title="Answer" collapse="true"}
Configure **Datamash** like this:

- *"Input tabular dataset"*: the output of Nextclade
- *"Group by fields"*: `3,2,6` (grouping on columns 2 and 6 as well keeps their values in the summary)
- *"Sort input"*: `Yes`
- *"Input file has a header line"*: `Yes`
- *"Print header line"*: `Yes`
- In *"Operation to perform on each group"*:
    - *"Type"*: `count`, *"On column"*: `Column: 1`
    - *"Type"*: `Combine all values`, *"On column"*: `Column: 1`
:::
::::

:::: {.callout-important title="Question: Pangolin vs Nextclade" collapse="true"}
Compare the Pangolin and Nextclade summaries. Were some samples assigned differently? Why?

::: {.callout-tip title="Answer" collapse="true"}
The two tools should mostly agree. Differences often concern low-quality samples: Nextclade does not leave samples unassigned, so it can produce unreliable assignments (sometimes even a recombinant lineage) for samples that Pangolin refused to assign. Check the `qc.overallStatus` column of Nextclade: samples classified as `bad` should not be trusted.
<!-- TODO: add the differences from your own run -->
:::
::::

# Conclusion

In this tutorial, we used a set of Galaxy workflows to analyse a batch of SARS-CoV-2 samples sequenced with ONT after amplification with the ARTIC v4 primers:

1. The **variation analysis** workflow filtered the reads, mapped them to the reference genome, removed the primers and called the mutations of each sample
2. The **reporting** workflow summarised the mutations per sample and across the batch
3. The **consensus construction** workflow reconstructed the genome of each sample, masking the uncertain positions
4. **Pangolin** and **Nextclade** assigned the samples to lineages

The workflows can analyse many samples with little hands-on time, while still allowing detailed dives into individual samples. To scale up further, workflow runs can be automated through the Galaxy API, as the Galaxy Covid-19 project did to analyse hundreds of thousands of public SARS-CoV-2 sequencing datasets ([Infectious Diseases Toolkit](https://www.infectious-diseases-toolkit.org/showcase/covid19-galaxy)).

::: {.callout-tip title="Key Points"}
- For tiled-amplicon data, the workflow needs the **primer scheme** (BED file) to remove the primer sequences, which would otherwise hide mutations in primer binding sites.
- The ONT workflow uses **minimap2** for mapping and **medaka** for variant calling; reads are filtered on length to match the amplicon sizes.
- Always check the coverage of the samples: low coverage leads to missed mutations, N-masked consensus sequences and unreliable lineage assignments.
- Mutations at intermediate allele frequencies can reveal contaminations or co-infections, which a single consensus sequence cannot represent.
- Pangolin and Nextclade use different algorithms and nomenclatures; comparing them, together with their quality columns, helps to spot unreliable assignments.
:::

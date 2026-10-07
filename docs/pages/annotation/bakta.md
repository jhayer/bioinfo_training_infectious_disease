# Bakta

::: {.callout-note}
**Time:** 15 min

**Objectives:** Be able to run bakta annotation tool

**Questions:**

- How to run bakta?

**Tools:** [bakta](https://github.com/oschwengers/bakta)
:::

## Foreword

For annotating bacterial contigs, several tools exists to do that: Prokka (Seemann 2014), Bakta (Schwengers et al. 2021), etc. Here, we use Bakta as recommended by Torsten Seemann avatar Torsten Seemann as the successor of Prokka.

Bakta is a tool for the rapid & standardized annotation of bacterial genomes and plasmids from both isolates and metagenome-assembled genomes (MAGs). It implements a comprehensive annotation workflow for coding and non-coding genes (i.e. tRNA, rRNA).

It is also able to detect and annotate small proteins (sORF). Predicted CDS are annotated using an alignment-free protein sequence identification approach with cross-references to public databases via stable identifiers.


![bakta](../images/bakta_wf.png)

## Running bakta

Launch bakta in genome mode with keeping the contig header

```bash
    # get help
    bakta --help
    # get databases list
    bakta_db list
    # downloading a database
    bakta_db download --output path/to/bd/ --type light
    # annotating a bacteria genome
    bakta --db /path/to/db/db-light/ assembly.fa --threads 8 --output annot_bakta
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1
# call the help
docker run --rm quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1 bakta
# get databases list
docker run --rm -v .:/data quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1 bash -c "cd data && bakta_db list"
# downloading a database
docker run --rm -v .:/data quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1 bash -c "cd data && bakta_db download --output path/to/bd/ --type light"
# annotating a bacteria genome
docker run --rm -v .:/data quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1 bash -c "cd data && bakta_db download --output path/to/bd/ --type light"
```
:::

::: {.callout-note title="singularity"}
```bash
# get the container
singularity build bakta_v1.9.2.sif docker://quay.io/biocontainers/bakta:1.9.2--pyhdfd78af_1
# call the help
singularity run bakta_v1.9.2.sif bakta
# get databases list
singularity run bakta_v1.9.2.sif bakta_db list
# downloading a database
singularity run bakta_v1.9.2.sif bakta_db download --output path/to/bd/ --type light
# annotating a bacteria genome
singularity run bakta_v1.9.2.sif bakta --db /path/to/db/db-light/ assembly.fa --threads 8 --output annot_bakta
```
:::

::: {.callout-note title="module load"}
```bash
# load module 
module load bakta/1.9.2
# get databases list
bakta_db list
# downloading a database
bakta_db download --output path/to/bd/ --type light
# annotating a bacteria genome
bakta --db /path/to/db/db-light/ assembly.fa --threads 8 --output annot_bakta
```
:::

On the analysis summary output ... 

::: {.callout-question}
How many contigs have been provided as input?
How long is the draft genome?
How many CDSs have been found?
How many small proteins?
Which other components have been found?
:::

Observe the annotation gff3 file

::: {.callout-question}
How many features are annotated?
:::


::: {.callout-note}
gff3 created by bakta contains features and also sequence fasta information
:::


Control the annotation quality using the [QC post](qc_post.md) annotation tuto.

Bakta gives a lot of information already, especially regarding CDSs or RNAs, but some structural annotation might be missing, e.g. plasmids, or interesting to identify independently.

* Plasmids: To identify plasmids in contigs, use PlasmidFinder (Carattoli and Hasman 2020), a tool for the identification and typing of plasmid sequences in Whole-Genome Sequencing. It uses the plasmidfinder database with hundreds of sequences to predict the plasmid in the data.

* Integrons : Integrons are genetic mechanisms that allow bacteria to adapt and evolve rapidly through the stockpiling and expression of new genes. To detect integrons, you can use IntegronFinder (Néron et al. 2022)

* Insertion Sequence elements IS : is a short DNA sequence that acts as a simple transposable element. IS are the smallest but most abundant autonomous transposable elements in bacterial genomes. They only code for proteins implicated in the transposition activity. They play then a key role in bacterial genome organization and evolution. To detect IS elements, use ISEScan (Xie and Tang 2017). ISEScan is a highly sensitive software pipeline based on profile hidden Markov models constructed from manually curated IS elements. 
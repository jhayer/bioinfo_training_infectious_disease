# Pangenome and core-genome phylogeny

::: {.callout-note}
**Time:** 120 min

**Objectives:** Be able to build the pangenome of a set of bacterial genomes with Roary in Galaxy, to identify the core and accessory genes, and to reconstruct a core-genome phylogeny with IQ-TREE.

**Questions:**

- Which genes are shared by all my genomes, and which are specific to some of them?
- How are my isolates related to each other?
- How can I use the core genome (genes shared by all isolates) to construct a phylogeny?
- How confident can I be in the branches of my tree?

**Tools:** [Bakta](https://github.com/oschwengers/bakta), [Roary](https://sanger-pathogens.github.io/Roary/), [IQ-TREE](http://www.iqtree.org/), Newick Display and iTOL phylogeny annotation
:::

## Foreword

The **pangenome** is the entire set of genes found in a group of genomes (here, several *E. coli* isolates). It is divided into:

- the **core genome**: genes present in all (or nearly all) genomes. It mostly contains essential "housekeeping" genes (replication, transcription, translation, central metabolism);
- the **accessory genome**: genes present in only some genomes. It often contains genes acquired by horizontal gene transfer (plasmids, phages, genomic islands), such as AMR genes or virulence factors.

Roary clusters the proteins of all genomes by similarity (BLASTp + MCL) and reports, for each cluster (gene), the genomes in which it is present. It classifies the genes according to the proportion of genomes that carry them:

| Category | Present in |
| --- | --- |
| Core genes | 99% ≤ genomes ≤ 100% |
| Soft core genes | 95% ≤ genomes < 99% |
| Shell genes | 15% ≤ genomes < 95% |
| Cloud genes | 0% ≤ genomes < 15% |

Because core genes are mostly inherited vertically (from mother cell to daughter cell), they are a good basis to reconstruct the evolutionary relationships between isolates. Roary aligns each core gene (MAFFT) and concatenates all these alignments into a single **core gene alignment**, from which we will build a maximum likelihood tree with IQ-TREE.

## Prerequisites

We will compare five *E. coli* assemblies from the [Data preparation](/pages/annotation/get_data.md) list:

| Accession | Isolate |
| --- | --- |
| GCA_964199995.1 | 364_BP0073U |
| GCA_964200015.1 | 323_BP0014P |
| GCA_964200025.1 | 372_BP0083B |
| GCA_964200035.1 | 368_BP0078U |
| GCA_964200045.1 | 366_BP0076U |

Roary needs one annotation file per genome, in **GFF3 format with the genome sequence at the end of the file** (after a `##FASTA` line).

::: {.callout-warning}
Two kinds of GFF files do **not** work with Roary because they do not contain the genome sequence:

- the GFF files downloaded from NCBI;
- the GFF3 files produced by Bakta with `Force Genbank/ENA/DDJB compliance` activated, as we did in the [Structural annotation](/pages/annotation/bakta.md) practical.

We therefore need to annotate the genomes again with Bakta, **without** the compliance option.
:::

::: {.callout-tip}
Annotating five genomes with Bakta takes some time. To save time during the session, you can directly import the history containing the Bakta annotations already prepared for you: [Pangenome Roary](https://usegalaxy.org.au/u/rahi_praveen/h/pangenome-roary) (click on `Import this history` at the top). Use the collection `Bakta on collection 31: Annotation and sequences (GFF3)` and go to [Running Roary](#running-roary).
:::

## Prepare the data

### Upload the assemblies and build a collection

* Upload the five assemblies (see [Data preparation](/pages/annotation/get_data.md)).

* Click on the `Select items` icon at the top of your history, select the five assemblies, then in the drop-down menu `All 5 selected`, choose `Build Dataset List`.

* Give the elements short and clear names: the isolate names (`364_BP0073U`, `323_BP0014P`, ...). These names will be used by Roary and will appear in the final tree.

* Name the collection `Ecoli_genomes` and click on `Build`.

::: {.callout-note}
A **dataset collection** groups several datasets in a single item of your history. When you run a tool on a collection, Galaxy runs the tool once per element and gathers the outputs in a new collection, in the same order and with the same names.
:::

### Annotate the genomes with Bakta

* Search for the `Bakta` tool in Galaxy ![](/pages/images/galaxy/tools.png)

* In `Select genome in fasta format`, click on the collection icon and select `Ecoli_genomes`.

* (Optional) In the `Optional annotation` section, set `Specify genus name` to `Escherichia` and `Specify species name` to `coli`.

* Make sure `Force Genbank/ENA/DDJB compliance` is **not** activated.

* In the `Selection of the output files` section, make sure `Annotation and sequences (GFF3)` is selected.

* Click on `Run tool`.

=> You get a collection `Bakta on collection ...: Annotation and sequences (GFF3)` with one GFF3 file per genome. Open one of them and scroll to the end: you should see the `##FASTA` line followed by the genome sequence.

## Running Roary

* Search for the `Roary` tool in Galaxy.

* In `Individual gff files or a dataset collection`, select `Dataset collection`, and select the GFF3 collection produced by Bakta.

* Keep the default values: `minimum percentage identity for blastp` = 95 and `percentage of isolates a gene must be in to be core` = 99.

* Click on `Run tool`.

Roary produces three main outputs:

| Output | Content |
| --- | --- |
| `Summary statistics` | Number of genes in each category (core, soft core, shell, cloud) |
| `Gene Presence Absence` | One line per gene cluster, with its annotation, the number of genomes that carry it, and one column per genome with the gene ID (empty if absent) |
| `Core Gene Alignment` | Concatenated alignment of all the core genes (one sequence per genome), in FASTA format |

=> Look at the `Summary statistics` by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg>.

::: {.callout-caution collapse="true"}
## How many genes are in the pangenome of these five isolates? How many are core genes?
<!-- TODO: fill in with the Summary statistics of the run -->
Look at the `Total genes` and `Core genes` lines of the summary statistics.
:::

::: {.callout-caution collapse="true"}
## Why are there no soft core genes and no cloud genes?
With only five genomes, each genome represents 20% of the dataset. A gene present in 4 genomes is present in 80% of them (shell), and a gene present in 1 genome is present in 20% of them, above the 15% limit of the cloud genes (shell too). The soft core (95–99%) and cloud (< 15%) categories can only be filled with larger datasets: with five genomes, the accessory genome is entirely in the shell category.
:::

::: {.callout-caution collapse="true"}
## What proportion of the pangenome is the core genome? What does it say about *E. coli*?
<!-- TODO: fill in with the Summary statistics of the run -->
Divide the number of core genes by the total number of genes. An *E. coli* genome contains about 4,500–5,500 genes, but the *E. coli* pangenome is huge (tens of thousands of genes): it is an **open pangenome**, meaning that new genes keep being discovered as new genomes are sequenced. Each new genome brings its own accessory genes, often acquired by horizontal gene transfer.
:::

Now open the `Gene Presence Absence` table. The `No. isolates` column gives the number of genomes carrying each gene.

::: {.callout-caution collapse="true"}
## Can you find the AMR genes detected with ABRicate (e.g. `blaCTX-M-55`, `tet(A)`, `sul2`) in the table? Are they core or accessory?
<!-- TODO: fill in with the Gene Presence Absence of the run -->
Use the search box (or the `Filter` tool on the `Annotation` column). AMR genes carried by plasmids are generally part of the accessory genome: they are present in only some of the isolates.
:::

::: {.callout-caution collapse="true"}
## Many accessory genes are annotated as "hypothetical protein". Why?
Accessory genes often come from mobile genetic elements (phages, plasmids, genomic islands) which are less well studied than the core genes, so their function is often unknown. This is one of the challenges of pangenome analysis.
:::

## Core-genome phylogeny with IQ-TREE

* Search for the `IQ-TREE` tool in Galaxy.

* In `Specify input alignment file in PHYLIP, FASTA, NEXUS, CLUSTAL or MSF format.`, select the Roary `Core Gene Alignment`.

* In `Specify sequence type as either of DNA, AA, BIN, MORPH, CODON or NT2AA for DNA, amino-acid, binary, morphological, codon or DNA-to-AA-translated sequences`, select `DNA`.

* In `Modelling Parameters` → `Automatic model selection` → `Do you want to use a custom model`, select `No, a custom model is not needed`, then select `Extended model selection followed by tree inference` (`MFP`).

* In `Bootstrap Parameters` → `Ultrafast bootstrap parameters` → `Specify number of bootstrap replicates (>=1000).`, type `1000`.

* In `General options` → `Seed`, type `12345`. With the same seed, everybody gets exactly the same tree and bootstrap values.

* Click on `Run tool`.

::: {.callout-note}
- **MFP** (ModelFinder Plus) tests many substitution models and selects the one that best fits the data, according to the Bayesian Information Criterion (BIC), then builds the tree with this model.
- **Ultrafast bootstrap** (UFBoot) resamples the alignment 1000 times to estimate the support of each branch of the tree. A branch with a support ≥ 95% can be considered reliable.
:::

IQ-TREE produces several outputs. The most important ones are:

| Output | Content |
| --- | --- |
| `MaxLikelihood Tree` | The final tree in Newick format, with the bootstrap supports |
| `Report and Final Tree` | The report: alignment statistics, best-fit model, and a text representation of the tree |

::: {.callout-caution collapse="true"}
## What is the length of the core gene alignment, and how many sites are parsimony-informative?
<!-- TODO: fill in with the IQ-TREE report of the run -->
Look at the beginning of the `Report and Final Tree` (`Input data` section). Most of the sites are constant: the isolates belong to the same species and their core genes are very similar. Only the variable sites (and, among them, the parsimony-informative ones) carry information about the relationships between isolates.
:::

::: {.callout-caution collapse="true"}
## Which substitution model was selected by ModelFinder?
<!-- TODO: fill in with the IQ-TREE report of the run -->
Look for `Best-fit model according to BIC` in the `Report and Final Tree`.
:::

## Visualise the tree

* Search for the `Newick Display` tool in Galaxy.

* In `Newick file`, select the IQ-TREE `MaxLikelihood Tree`.

* In `Branch support`, select `Display branch support`.

* Click on `Run tool`, then click on the eye icon of the output to display the tree.

### Annotate the tree with iTOL

[iTOL](https://itol.embl.de) (interactive Tree Of Life) is a web tool to display and annotate phylogenetic trees.

* In your Galaxy history, click on the IQ-TREE `MaxLikelihood Tree` and on the download icon to save it on your computer.

* Go to the [iTOL upload page](https://itol.embl.de/upload.cgi), click on `Choose file`, select the downloaded tree and click on `Upload`.

* In the `Advanced` tab of the control panel on the right, set `Bootstrap / metadata` to `Display` to show the bootstrap supports on the branches.

* The tree is unrooted. To display it with a root, click on a branch and choose `Tree structure` → `Reroot the tree here`, or use the midpoint rooting option.

::: {.callout-tip}
You can colour the isolates by any information you have (sample type, AMR profile, ...): click on a leaf label and choose `Label` → `Color`, or drag and drop an annotation file (see [iTOL datasets](https://itol.embl.de/help.cgi#datasets)). The free iTOL account lets you keep your trees online; [phylo.io](https://phylo.io/) is another free alternative.
:::

::: {.callout-caution collapse="true"}
## Which isolates are the most closely related? Are the branches well supported?
<!-- TODO: fill in with the tree of the run -->
Look at the topology of the tree and at the bootstrap values on the branches. Remember that this tree is **unrooted**: it shows the relationships between the isolates, but not which one diverged first.
:::

::: {.callout-caution collapse="true"}
## Do closely related isolates share the same AMR genes?
<!-- TODO: fill in by comparing the tree with the ABRicate results of the five isolates -->
Compare the tree with the presence/absence of AMR genes (Gene Presence Absence table, or ABRicate results). AMR genes carried by plasmids can be shared by distantly related isolates (horizontal transfer), and absent from very closely related isolates (loss of the plasmid): the accessory genome does not always follow the core-genome phylogeny.
:::

## Run the whole analysis as a workflow

The annotation, pangenome and phylogeny steps have been bundled into a Galaxy workflow: `Genome assemblies` (collection) → `Bakta` → `Roary` → `IQ-TREE`, with the same parameters as above. The tree visualisation (Newick Display or iTOL) is done afterwards.

* Import the workflow from: [Pangenome Workflow](https://usegalaxy.org.au/u/rahi_praveen/w/pangenome-workflow)

* Click on `Workflows` in the activity bar on the left, then on the run icon next to the imported workflow.

* In `Genome assemblies`, select your collection of genome assemblies (`Ecoli_genomes`) and click on `Run workflow`.

::: {.callout-warning}
The input of the workflow must be a **collection** (list) of FASTA files, not individual datasets. If your collection does not appear in the list, check that it was built from the FASTA assemblies (see [Upload the assemblies and build a collection](#upload-the-assemblies-and-build-a-collection)).
:::

::: {.callout-note}
To learn how to build your own workflow from a history, see [Create a Galaxy workflow](/pages/course/create_galaxy_workflow.md).
:::

## Remarks

- Roary is fast and easy to use, but it is no longer maintained and it is sensitive to annotation errors (fragmented genes, contamination). [Panaroo](https://github.com/gtonkinhill/panaroo), also available in Galaxy Australia, is a more recent alternative that corrects some of these errors.
- Use the same annotation tool, with the same parameters, for all your genomes: otherwise, differences in annotation will appear as differences in gene content.
- Recombination can blur the phylogenetic signal of the core genome in bacteria. For more accurate phylogenies, recombinant regions can be detected and removed with tools such as [Gubbins](https://github.com/nickjcroucher/gubbins) or [ClonalFrameML](https://github.com/xavierdidelot/ClonalFrameML).
- In the next practical, we will explore pangenomes interactively with [PanExplorer](/pages/panexplorer/panexplorer.md).

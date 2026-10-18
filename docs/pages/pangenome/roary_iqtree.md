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

::: {.callout-note}
The answers given in this page were obtained with these five complete genomes (one chromosome and 2 to 6 plasmids each). In the results, isolate 364_BP0073U appears as `364_BP0073U-1`, the name of its file.

You can look at the complete results here: [Pangenome Roary](https://usegalaxy.org.au/u/rahi_praveen/h/pangenome-roary).
:::

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
Look at the `Total genes` and `Core genes` lines of the summary statistics.

| Category | Number of genes |
| --- | --- |
| Core genes (99% ≤ genomes ≤ 100%) | 3,393 |
| Soft core genes (95% ≤ genomes < 99%) | 0 |
| Shell genes (15% ≤ genomes < 95%) | 4,376 |
| Cloud genes (0% ≤ genomes < 15%) | 0 |
| **Total genes** | **7,769** |

The pangenome of these five isolates contains **7,769 genes**, of which **3,393 are core genes**, present in all five isolates. Each isolate carries between 4,527 and 5,207 genes.
:::

::: {.callout-caution collapse="true"}
## Why are there no soft core genes and no cloud genes?
Indeed, both categories are empty (0 genes). With only five genomes, each genome represents 20% of the dataset. A gene present in 4 genomes is present in 80% of them (shell), and a gene present in 1 genome is present in 20% of them, above the 15% limit of the cloud genes (shell too). The soft core (95–99%) and cloud (< 15%) categories can only be filled with larger datasets: with five genomes, the accessory genome is entirely in the shell category.
:::

::: {.callout-caution collapse="true"}
## What proportion of the pangenome is the core genome? What does it say about *E. coli*?
Divide the number of core genes by the total number of genes: 3,393 / 7,769 = **44%**. In other words, more than half of the pangenome (4,376 genes) is accessory, although we only compared five isolates of the same species:

| Present in | Number of genes |
| --- | --- |
| 5 isolates (core) | 3,393 |
| 4 isolates | 246 |
| 3 isolates | 612 |
| 2 isolates | 1,106 |
| 1 isolate (isolate-specific) | 2,412 |

An *E. coli* genome contains about 4,500–5,500 genes, but the *E. coli* pangenome is huge (tens of thousands of genes): it is an **open pangenome**, meaning that new genes keep being discovered as new genomes are sequenced. Each new genome brings its own accessory genes, often acquired by horizontal gene transfer.
:::

Now open the `Gene Presence Absence` table. The `No. isolates` column gives the number of genomes carrying each gene, and the last five columns give the ID of the gene in each isolate (empty if absent).

::: {.callout-caution collapse="true"}
## Which isolate has the most isolate-specific genes? Can you explain why?
Count the genes with `No. isolates` = 1 for each isolate (e.g. with the `Filter` tool, then look at which column is not empty):

| Isolate | Genes | Isolate-specific genes | Plasmids |
| --- | --- | --- | --- |
| 323_BP0014P | 4,815 | 161 | 4 |
| 372_BP0083B | 4,870 | 242 | 2 |
| 364_BP0073U | 4,527 | 388 | 2 |
| 368_BP0078U | 4,990 | 785 | 6 |
| 366_BP0076U | 5,207 | 836 | 6 |

366_BP0076U and 368_BP0078U have the most specific genes. They also carry the most plasmids (6 each, see the sequence names in the FASTA files): plasmids, together with phages and genomic islands, bring many accessory genes. 323_BP0014P and 372_BP0083B have few specific genes because they are very closely related to each other (see the tree below): most of their accessory genes are shared.
:::

::: {.callout-caution collapse="true"}
## Can you find the AMR genes detected with ABRicate (e.g. `blaCTX-M-55`, `tet(A)`, `sul2`) in the table? Are they core or accessory?
Use the search box (or the `Filter` tool on the `Annotation` column). AMR genes carried by plasmids are generally part of the accessory genome: they are present in only some of the isolates.

All the acquired AMR genes found with ABRicate in 364_BP0073U (`blaCTX-M-55`, `blaTEM-1`, `floR`, `catA2`, `sul2`, `sul3`, `dfrA14`, `tet(A)`, `aadA1`, `aph(3')-Ia`, `aph(3'')-Ib`, `aph(6)-Id`, `aac(3)-IIe`, `estX-3`, `qacL`) are in the **accessory** genome. The only resistance gene of the core genome is `blaEC`, the chromosomal AmpC beta-lactamase intrinsic to *E. coli*.

Looking at the five isolates, the main acquired AMR genes are:

| Isolate | Beta-lactamases | Other AMR genes |
| --- | --- | --- |
| 323_BP0014P | `blaCTX-M-27` (ESBL) | `aadA5`, `aph(3'')-Ib`, `aph(6)-Id`, `dfrA17`, `mph(A)`, `sul1`, `sul2`, `tet(A)` |
| 372_BP0083B | `blaCTX-M-27` (ESBL) | `aadA5`, `aph(3'')-Ib`, `aph(6)-Id`, `dfrA17`, `mph(A)`, `sul1`, `sul2`, `tet(A)` |
| 364_BP0073U | `blaCTX-M-55` (ESBL), `blaTEM-1` | `aac(3)-IIe`, `aadA1`, `aph(3')-Ia`, `aph(3'')-Ib`, `aph(6)-Id`, `catA2`, `dfrA14`, `floR`, `sul2`, `sul3`, `tet(A)` |
| 368_BP0078U | `blaOXA-48` (**carbapenemase**), `blaCTX-M-27` (ESBL), `blaDHA-1` (AmpC) | `aadA1`, `aadA5`, `aph(3'')-Ib`, `aph(6)-Id`, `dfrA1`, `dfrA17`, `erm(B)`, `mph(A)`, `qnrB`, `sul1`, `sul2`, `tet(A)` |
| 366_BP0076U | `blaNDM-5` (**carbapenemase**), `blaCTX-M-15` (ESBL), `blaOXA-1` | `aac(3)-IIe`, `aac(6')-Ib`, `aadA5`, `dfrA17`, `sul1`, `tet(B)` |

Two isolates carry a carbapenemase gene (`blaNDM-5` or `blaOXA-48`), conferring resistance to carbapenems, antibiotics of last resort.
:::

::: {.callout-caution collapse="true"}
## 364_BP0073U carries two copies of `blaCTX-M-55` (see the ABRicate practical), but only one appears in the `blaCTX-M-55` line. Where is the second one?
The second copy is in the line `blaCTX-M-15`, together with the `blaCTX-M-15` gene of 366_BP0076U. Roary clusters proteins with at least 95% identity, and CTX-M-15 and CTX-M-55 differ by a single amino acid, so they can end up in the same cluster. In addition, Roary separates the copies of a gene present several times in the same genome (paralogs) into different clusters. Be careful when interpreting gene names in a pangenome: one line is a cluster of similar genes, not necessarily one exact gene variant.
:::

::: {.callout-caution collapse="true"}
## Are genes of unknown function ("hypothetical protein") more frequent in the core or in the accessory genome? Why?
In the accessory genome. With Bakta, 1.4% of the accessory genes are annotated as `hypothetical protein`, against 0.1% of the core genes (14 times more). Accessory genes often come from mobile genetic elements (phages, plasmids, genomic islands) which are less well studied than the core genes, so their function is more often unknown. This is one of the challenges of pangenome analysis.
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
Look at the beginning of the `Report and Final Tree` (`SEQUENCE ALIGNMENT` section):

```
Input data: 5 sequences with 3312502 nucleotide sites
Number of constant sites: 3.18469e+06 (= 96.1416% of all sites)
Number of parsimony informative sites: 70491
```

The core gene alignment is **3.3 Mb** long (3,393 genes), and **96%** of the sites are constant: the isolates belong to the same species and their core genes are very similar. Only the variable sites (about 128,000) carry information about the relationships between isolates, and among them 70,491 are parsimony-informative (at least two different nucleotides, each present in at least two isolates).
:::

::: {.callout-caution collapse="true"}
## Which substitution model was selected by ModelFinder?
Look for `Best-fit model according to BIC` in the `Report and Final Tree`: **GTR+F+I**.

- `GTR` (General Time Reversible): each of the 6 types of substitution (A↔C, A↔G, ...) has its own rate. Look at the `Rate parameter R` section: transitions (A↔G = 4.5, C↔T = 7.2) are much more frequent than transversions (≈ 1), as usual in DNA.
- `+F`: the base frequencies are estimated from the alignment.
- `+I`: a proportion of the sites are invariable (here 89%).
:::

## Visualise the tree

* Search for the `Newick Display` tool in Galaxy.

* In `Newick file`, select the IQ-TREE `MaxLikelihood Tree`.

::: {.callout-warning}
Do not select the `Report and Final Tree`: it is a text report, not a tree file, and Newick Display fails with the error `missing ';' at end of tree`.
:::

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
Look at the topology of the tree and at the bootstrap values on the branches. Remember that this tree is **unrooted**: it shows the relationships between the isolates, but not which one diverged first.

```
(323_BP0014P:0.0000674141,((364_BP0073U-1:0.0061152445,368_BP0078U:0.0085744840)100:0.0056828866,366_BP0076U:0.0111294983)100:0.0187722743,372_BP0083B:0.0000879027);
```

The isolates form two groups separated by a long branch:

- **323_BP0014P and 372_BP0083B** are almost identical: their branches are very short, and their core genomes differ by only **510 SNPs** over 3.3 Mb. They belong to the same lineage.
- **364_BP0073U, 368_BP0078U and 366_BP0076U** form the second group. Within this group, 364_BP0073U and 368_BP0078U are the closest (43,279 SNPs).
- The two groups differ by about 80,000 SNPs.

Both internal branches have a bootstrap support of **100%**: the tree is very well supported, which is expected with such a long alignment. The consensus tree of the 1000 bootstrap trees is identical to the ML tree.

To count the SNPs between two isolates, you can compare their sequences in the `Core Gene Alignment`. The `MaxLikelihood Distance Matrix` gives the corresponding distances corrected by the model.
:::

::: {.callout-caution collapse="true"}
## Do closely related isolates share the same AMR genes?
Compare the tree with the presence/absence of AMR genes (Gene Presence Absence table, or ABRicate results). AMR genes carried by plasmids can be shared by distantly related isolates (horizontal transfer), and absent from very closely related isolates (loss of the plasmid): the accessory genome does not always follow the core-genome phylogeny.

In our dataset, both situations are visible (see the AMR table above):

- 323_BP0014P and 372_BP0083B, the two almost identical isolates, have exactly the **same AMR genes**, including the ESBL `blaCTX-M-27`: they probably inherited them from their common ancestor.
- `blaCTX-M-27`, `mph(A)`, `sul1`, `dfrA17` and `aadA5` are also found in 368_BP0078U, which belongs to the other group (about 80,000 SNPs away): these genes were probably acquired by horizontal gene transfer, e.g. on a plasmid.
- 364_BP0073U and 368_BP0078U are the closest isolates of their group, but they have very different AMR genes (`blaCTX-M-55` vs `blaOXA-48` and `blaCTX-M-27`).
- The two carbapenemase genes are each found in only one isolate (`blaNDM-5` in 366_BP0076U, `blaOXA-48` in 368_BP0078U), in different branches of the tree: they were acquired independently.

To confirm that genes are carried by the same plasmid in different isolates, the plasmids themselves need to be compared (e.g. with PlasmidFinder replicons, or by aligning the plasmid sequences).
:::

## Run the whole analysis as a workflow

All the steps above have been bundled into a Galaxy workflow: `Genome assemblies` (collection) → `Bakta` → `Roary` → `IQ-TREE` → `Newick Display`, with the same parameters as above. The iTOL annotation is done afterwards, on the downloaded `MaxLikelihood Tree`.

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

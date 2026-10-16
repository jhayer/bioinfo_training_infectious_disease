# Quality Check of a Genome Assembly Before Annotation

::: {.callout-note}
**Time:** 30 min

**Objectives:** Be able to check a reference assembly.

**Tools:** [seqkit](https://github.com/shenwei356/seqkit), [BUSCO](https://busco.ezlab.org), [checkM2](https://github.com/chklovski/CheckM2)

Related ressources : [Galaxy Training!](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/ERGA-post-assembly-QC/tutorial.html#generate-interactive-plots-with-blobtoolkit)
:::

## Foreword

We won’t focus on how to build the best assembly here, which is an challenging task. Instead, we’ll check that the assembly you have is suitable for annotation. Running quality control ensures your genome is clean and reliable before moving on.

## Statistics

We will start by generating some basic statistics about our genome assembly using seqkit. This will give us an overview of the assembly's quality and characteristics. For example, we can check the total number of sequences, their lengths, GC content, and the presence of undefined bases.
A fragmented genome assembly will typically have many short contigs, which can indicate issues with the assembly process or the quality of the input data. By examining these statistics, we can get an initial sense of the assembly's reliability.

* Search for the `SeqKit statistics` tool in Galaxy ![](/pages/images/galaxy/tools.png)

* Select your genome assembly.

* Activate the `Output all statistics` option.

* Run the tool to generate the statistics report.

![](/pages/annotation/images/seqkit.png)

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `SeqKit statistics` line in your history on the right side.

::: {.callout-caution}
What is the length of the assembly?  
Do you have any gap in your assembly?  
What is your N50 value?  
What is the GC content of your assembly?  
:::

## BUSCO


[BUSCO](https://busco.ezlab.org) (Benchmarking Universal Single-Copy Orthologs) searches for sets of conserved genes that are expected to be present in nearly all genomes of a given lineage. The results indicate how many of these genes are complete, duplicated, fragmented, or missing (Simão et al. 2015).  

* Search for the `BUSCO` tool in Galaxy ![](/pages/images/galaxy/tools.png)

The full lineage for E. coli is: 
```bash
Lineage (full): cellular organisms; Bacteria; Pseudomonadati; Pseudomonadota; Gammaproteobacteria; Enterobacterales; Enterobacteriaceae; Escherichia; Escherichia coli
```

### BUSCO using Bacteria lineage

In this first run we will use the **Bacteria** lineage to assess the completeness of our genome assembly. 

* Pay attention that you run BUSCO in genome mode.  
* Select the appropriate lineage for your genome assembly.  
* Run the tool to generate the BUSCO report.

![](/pages/annotation/images/busco_g1.png)

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `Busco Short summary` line in your history on the right side.

::: {.callout-caution collapse="true"}
## Question: Can you tell me how many BUSCO genes contains the data set used? And how many genome have been used to determine them?
Number of genomes: 3130, number of BUSCOs: 116)
:::

::: {.callout-caution collapse="true"}
## What is the Completeness of the assembly?
C:98.3%[S:98.3%,D:0.0%],F:0.0%,M:1.7%,n:116,E:8.8%
:::

### BUSCO using Bacteria Enterobacteriaceae lineage

![](/pages/annotation/images/busco_g2.png)

::: {.callout-caution collapse="true"}
## Question: Can you tell me how many BUSCO genes contains the data set used? And how many genome have been used to determine them?
number of genomes: 107, number of BUSCOs: 874
:::

::: {.callout-caution collapse="true"}
## What is the Completeness of the assembly?
C:100.0%[S:99.8%,D:0.2%],F:0.0%,M:0.0%,n:874,E:1.0%
:::

::: {.callout-caution}
## Do you have an idea why the result is less good at Bacteria level while we are checking against less BUSCOs genes?
This is inherent  to the way BUSCO genes are selected. They have to be in at least 90% Of the genomes within the lineage to be considered as BUSCOs. So more species we have in a lineage less speci 
:::

## CheckM2

CheckM2 is a machine-learning tool designed to assess the quality of metagenome-assembled genomes (MAGs) by estimating their completeness and contamination.
First, it uses Prodigal to predict coding sequences and proteins, and it extracts general genomic features such as genome size, coding density, GC content, and contig N50. It also uses DIAMOND to compare predicted proteins against a reference database associated with KEGG Orthology functional annotations.
These genomic and functional features are then used as inputs to machine-learning models trained to predict genome completeness and contamination.
For example, the model can learn from genomes with known quality levels and use the relationships between their genomic features and their completeness or contamination to make predictions for new genomes.
Therefore, unlike traditional approaches that rely heavily on specific marker genes, CheckM2 uses a broader range of genomic and functional information to assess genome quality.

* Search for the `CheckM2` tool in Galaxy ![](/pages/images/galaxy/tools.png)

* Run CheckM2 with default values on your assembly.

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `checkm2 Quality report` line in your history on the right side.

::: {.callout-caution collapse="true"}
## What is the Completeness of the assembly?
100.0	
:::
::: {.callout-caution collapse="true"}
## What is the contamination of the assembly?
0.54
:::

Interestingly, it provides a first glimpse into the gene content of the assembly (via prodigal). But keep in mind that this is only a preliminary prediction and further annotation steps are necessary to obtain a comprehensive understanding of the gene content.

::: {.callout-caution collapse="true"}
## How many genes were predicted?
5139
:::

<hr>

::: {.callout-note}
## The Snail Plot 

[Snail plots are badges of genome assembly quality](https://academic.oup.com/g3journal/article/16/6/jkag074/8606825).

You can find an example of such interactive snail plot [here](https://juke34.github.io/assembly-stats/).  
You can find a protocol to generate such plot via galaxy [here](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/ERGA-post-assembly-QC/tutorial.html#genome-assembly-overview-with-blobtoolkit). In principle, generating this plot requires just a single command line. However, the Galaxy implementation makes the process less straightforward, as it requires you to perform an additional, computationally intensive contamination analysis before generating the plot.

example of a snail plot is shown below:  

![](/pages/annotation/images/snail_plot.png){width="600"}
:::
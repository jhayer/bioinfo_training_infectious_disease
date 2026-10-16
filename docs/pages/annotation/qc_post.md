# Quality Check of a Genome Annotation

::: {.callout-note}
**Time:** 40 min

**Objectives:** Be able to assess the quality of an annotation, and compare against an existing reference annotation.

**Tools:** [AGAT](https://github.com/NBISweden/AGAT), [BUSCO](https://busco.ezlab.org), [JBrowse2](https://jbrowse.org/jb2/)
:::

## Foreword

Once an annotation is produced, it is essential to assess its reliability and completeness. There is no single “perfect” metric, and each method highlights different aspects of annotation quality. In practice, evaluation usually involves combining several complementary approaches.

## Statistics

General statistics like number of predicted genes, their length, number of intron, etc gives you a sense of congruency with other builds or expectations for the species. However, it’s not a true quality metric: two annotations may have different gene counts without one being objectively “more correct.”

* Search for the `AGAT` toolkit in Galaxy ![](/pages/images/galaxy/tools.png)

* Select the `annotation statistics` tool from the AGAT tool selector.

* Select your annotation file (GFF/GTF format).

* Select your reference genome from history.

* Run the tool

![](/pages/annotation/images/qc-post/AGAT.png){width="600"}

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `AGAT stats file` line in your history on the right side.

::: {.callout-caution collapse="true"}
## Do we have overlapping genes?
Yes
:::
::: {.callout-caution collapse="true"}
## What is the size of the shortest and longest protein coding genes?
Longest: 20778 bp  
Shortest: 27 bp
:::

## BUSCO

[BUSCO](https://busco.ezlab.org) (Benchmarking Universal Single-Copy Orthologs) searches for sets of conserved genes that are expected to be present in nearly all genomes of a given lineage. The results indicate how many of these genes are complete, duplicated, fragmented, or missing (Simão et al. 2015).  

::: {.callout-note}
   The BUSCO results on proteins derived from the annotation should be similar to those obtained from BUSCO on the assembly itself (usually lower). If the annotation shows significantly lower completeness, it indicates that there is room for improvement in your annotation process.
   You need the protein set in fasta format of your annotation in order to run this BUSCO. If the annotation tool did not provide that data, you can extract the proteins from the GTF/GFF annotation using AGAT.
:::


* Search for the `BUSCO` tool in Galaxy ![](/pages/images/galaxy/tools.png)

The full lineage for E. coli is: 
```bash
Lineage (full): cellular organisms; Bacteria; Pseudomonadati; Pseudomonadota; Gammaproteobacteria; Enterobacterales; Enterobacteriaceae; Escherichia; Escherichia coli
```

### BUSCO using Bacteria lineage

In this first run we will use the **Bacteria** lineage to assess the completeness of our annotation assembly. 

* Pay attention that you run BUSCO in genome mode.  
* Select the appropriate lineage for your genome assembly.  
* Run the tool to generate the BUSCO report.

![](/pages/annotation/images/qc-post/busco_a1.png){width="600"}

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `Busco Short summary` line in your history on the right side.

::: {.callout-caution collapse="true"}
## What is the Completeness of the assembly? Do you see any difference with the BUSCO result on assembly?
C:99.1%[S:99.1%,D:0.0%],F:0.0%,M:0.9%,n:116.  
Result on the annotation is slightly better than the assembly one. We have annotated expected genes. We even have a better result, it reflects that Bakta annotation process on this subset of gene is better than the annotation process used by BUSCO on the raw genome assembly.
:::

### BUSCO using Bacteria Enterobacteriaceae lineage

![](/pages/annotation/images/qc-post/busco_a2.png){width="600"}

::: {.callout-caution collapse="true"}
## What is the Completeness of the assembly?
C:99.8%[S:99.5%,D:0.2%],F:0.1%,M:0.1%,n:874
We missed 1 BUSCO gene compared to the same analysis made on raw genome assembly. This is a common pattern. BUSCO use HMMs pattern design for a specific subset of gene, that might work better than approach used by the annotation tool used for annotation. This is not a problem, do not forget that we succedded to annotate much more gene thanks to Bakta.
:::

::: {.callout-caution collapse="true"}
## Based on this result how many coding gene we might expect to be missing in our annotation?
Busco show we missed 1 gene over 874 total BUSCO groups searched.
We annotated 5119 protein coding genes ith Bakta. Based on the busco result, we might have missed 6 genes (1*5119/874).
:::


# Diversity analysis

## Build a phylogenetic tree

Some diversity metrics (Faith's phylogenetic diversity, UniFrac) take into account how related the ASVs are. They need a **phylogenetic tree** of the ASVs.

::: {.callout-tip title="Build a phylogenetic tree"}
1. Open **qiime2 phylogeny align-to-tree-mafft-fasttree** with:
    - *"sequences"*: `rep-seqs.qza`
2. Click **Run Tool**
3. Rename the `rooted_tree` output `rooted-tree.qza`
:::

This tool aligns the ASVs with **MAFFT**, masks highly variable positions, builds a tree with **FastTree** and roots it at its midpoint.

## Choose a rarefaction depth

Samples have different numbers of reads, and the number of ASVs observed in a sample increases with the number of reads. To compare samples fairly, the diversity tools first **rarefy** the table: they randomly subsample each sample to the same number of reads, the **sampling depth**. Samples with fewer reads than this depth are excluded.

Choosing the depth is a trade-off: a high depth keeps more information per sample, a low depth keeps more samples.

::: {.callout-tip title="Alpha rarefaction curves"}
1. Open **qiime2 diversity alpha-rarefaction** with:
    - *"table"*: `table.qza`
    - *"phylogeny"*: `rooted-tree.qza`
    - *"max_depth"*: the number of reads of the sample with the **most** reads (from the **feature-table summarize** visualization)
    - *"metadata"*: `Metadata table`
2. Open the `.qzv` output
:::

:::: {.callout-important title="Question: Sampling depth" collapse="true"}
1. Do the rarefaction curves reach a plateau? What does it mean?
2. Using the **Interactive Sample Detail** tab of the **feature-table summarize** visualization, which sampling depth would you choose to keep all samples?

::: {.callout-tip title="Answer" collapse="true"}
1. Yes, the curves flatten: sequencing more would not reveal many new ASVs, the sampling is deep enough to describe the diversity.
2. The number of reads of the sample with the **fewest** reads. All samples are retained, and the curves show that the diversity is already captured at that depth.
<!-- TODO: add the chosen depth from your own run -->
:::
::::

## Compute the core diversity metrics

::: {.callout-tip title="Core diversity metrics"}
1. Open **qiime2 diversity core-metrics-phylogenetic** with:
    - *"phylogeny"*: `rooted-tree.qza`
    - *"table"*: `table.qza`
    - *"sampling_depth"*: the depth chosen above
    - *"metadata"*: `Metadata table`
2. Click **Run Tool**
:::

This tool rarefies the table and computes several metrics at once:

| Type | Metric | Measures |
|---|---|---|
| Alpha | Observed features | Number of ASVs (richness) |
| Alpha | Shannon | Richness and evenness |
| Alpha | Pielou's evenness | Evenness |
| Alpha | Faith's PD | Phylogenetic diversity |
| Beta | Jaccard | Presence/absence |
| Beta | Bray-Curtis | Abundance |
| Beta | Unweighted UniFrac | Presence/absence + phylogeny |
| Beta | Weighted UniFrac | Abundance + phylogeny |

For each beta diversity metric, it also produces a distance matrix, a PCoA and an **Emperor** plot (`.qzv`).

## Alpha diversity

A **diversity index** is a quantitative measure of the variety of species in a community. The **α diversity** describes the diversity *within* a sample. It considers the number of different species (the **richness**) and how evenly the individuals are distributed among them (the **evenness**).

![Explanation of the α diversity](./images/alphadiversity_metrics.png)

Here we compare the α diversity between early and late samples with 2 measures:

- The **Shannon** index, which calculates the uncertainty in predicting the species identity of an individual selected from a community.
- **Faith's phylogenetic diversity**, the sum of the branch lengths of the tree covered by the ASVs of a sample.

::: {.callout-tip title="Compare alpha diversity between groups"}
1. Open **qiime2 diversity alpha-group-significance** with:
    - *"alpha_diversity"*: `shannon_vector` output of **core-metrics-phylogenetic**
    - *"metadata"*: `Metadata table`
2. Run it again with *"alpha_diversity"*: `faith_pd_vector`
3. Open the `.qzv` outputs and select the column `when`
:::

::: {.callout-note title="Other alpha metrics"}
Other indices, like the **Simpson** index, can be computed with **qiime2 diversity alpha** (*"metric"*: `simpson`) on the rarefied table, then compared with **alpha-group-significance**.
:::

:::: {.callout-important title="Question: Alpha diversity" collapse="true"}
Is there any obvious systematic difference in alpha diversity between early and late samples? Is it significant?

::: {.callout-tip title="Answer" collapse="true"}
No obvious difference. The visualization shows box plots for each group and a **Kruskal-Wallis** test: look at the p-value to conclude whether the difference is significant.
<!-- TODO: add the p-values from your own run -->
:::
::::

## Beta diversity

The **β diversity** measures how different the samples are from each other. Each beta metric produces a distance matrix, which is projected in 2 or 3 dimensions with a **PCoA** (Principal Coordinates Analysis) to visualise it. Samples close to each other in the plot have similar communities.

::: {.callout-tip title="Explore the PCoA with Emperor"}
1. Open the `bray_curtis_emperor` output of **core-metrics-phylogenetic**
2. In the **Color** tab, select `when`
3. Do the same with `unweighted_unifrac_emperor`
:::

:::: {.callout-important title="Question: Ordination" collapse="true"}
Is there a clear separation between the early and late samples?

::: {.callout-tip title="Answer" collapse="true"}
Yes, early and late samples form two separate groups along the first axis.
:::
::::

We can test whether this separation is significant with a **PERMANOVA**:

::: {.callout-tip title="Test differences between groups"}
1. Open **qiime2 diversity beta-group-significance** with:
    - *"distance_matrix"*: `bray_curtis_distance_matrix` output of **core-metrics-phylogenetic**
    - *"metadata"*: `Metadata table`, column `when`
    - *"pairwise"*: `Yes`
2. Open the `.qzv` output
:::

:::: {.callout-important title="Question: PERMANOVA" collapse="true"}
Is the community composition significantly different between early and late samples?

::: {.callout-tip title="Answer" collapse="true"}
Look at the **p-value** of the PERMANOVA test: if it is below 0.05, the composition of the early and late samples is significantly different. The box plots show that distances *between* groups are larger than distances *within* groups.
<!-- TODO: add the pseudo-F and p-value from your own run -->
:::
::::

# Conclusion

We processed raw paired-end 16S amplicon reads with QIIME 2 in Galaxy:

1. Imported the reads as a QIIME 2 artifact and inspected their quality
2. Denoised them with DADA2 into an ASV table
3. Assigned a taxonomy to the ASVs with a naive Bayes classifier
4. Built a phylogenetic tree and compared the alpha and beta diversity of early and late samples

::: {.callout-tip title="Key Points"}
- QIIME 2 stores data in artifacts (`.qza`) and results in visualizations (`.qzv`), both keeping the full provenance of the analysis.
- **qiime2 dada2 denoise-paired** runs the whole DADA2 pipeline (filtering, error learning, denoising, merging, chimera removal) and produces a high-resolution ASV table.
- Truncation lengths are chosen from the quality profiles, while keeping enough overlap to merge the reads.
- Taxonomy is assigned with a classifier trained on the amplified region and matching the QIIME 2 version.
- Diversity analyses require rarefying the samples to a common depth; alpha diversity describes each sample, beta diversity compares samples.
:::

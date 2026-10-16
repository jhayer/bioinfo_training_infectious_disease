---
title: "Hybrid Assembly with Unicycler"
---

::: {.callout-note title="Lesson overview"}
**Questions**

- What is a hybrid assembly, and why is it better than a short-read or a long-read assembly alone?
- How does Unicycler combine short and long reads?
- Which parameters should we use for our data?

**Objectives**

- Understand how Unicycler builds a hybrid assembly.
- Assemble sample 368 with **Unicycler**, using the trimmed short and long reads.
- Interpret the Unicycler outputs: contigs, depth and circularity.
- Compare the result with the genome published by the authors.
:::

::: {.callout-note title="Acknowledgment"}
Some of the material, including schemes, comes from this [Unicycler Galaxy Training](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/assembly-with-preprocessing/slides-plain.html).
:::

# Hybrid Assembly

The best option, when you have long reads and short reads for the same sample, is to use a **hybrid assembly**.
Hybrid sequencing usually means combining two sequencing technologies on the same sample, most often short reads (Illumina: accurate, cheap, ~150 bp) with long reads (PacBio or Oxford Nanopore: 10 kb to 100+ kb, historically more error-prone). The goal is to get the strengths of both:

- **Long reads** are used to resolve the ambiguities (repeats) that break the assemblies made with short reads only.
- **Short reads**, with a low error rate, are used to get an accurate sequence, correcting the errors of the long reads.

![hybrid-seq-overview](images/06-assembly/hybrid-seq-overview.png)

This is the strategy chosen by the authors of the study: the genome of strain 368 deposited at the ENA ([GCA_964200035](https://www.ebi.ac.uk/ena/browser/view/GCA_964200035)) was assembled with **Unicycler** from the Illumina and Nanopore reads, and is made of 7 circular sequences:

| Sequence | Length (bp) |
|---|---|
| Chromosome | 4,894,004 |
| Plasmid p1 | 149,835 |
| Plasmid p2 | 110,246 |
| Plasmid p3 | 34,980 |
| Plasmid p4 | 7,872 |
| Plasmid p5 | 3,646 |
| Plasmid p6 | 2,101 |
| **Total** | **5,202,684** |

We will try to reproduce this assembly.

# How does Unicycler work ?

[Unicycler](https://github.com/rrwick/Unicycler) ([Wick et al. 2017](https://doi.org/10.1371/journal.pcbi.1005595)) is an assembler designed for bacterial genomes. For hybrid data, it uses a **short-read-first** approach:

![unicycler-diagram](images/06-assembly/unicycler-diagram.png)

1. **Short-read assembly graph**: Unicycler assembles the short reads with **SPAdes**, trying several k-mer sizes and keeping the best graph. As in our short read assembly, this graph is accurate, but tangled: the repeats longer than the reads appear as nodes with several possible paths in and out.
2. **Bridging with the long reads**: Unicycler aligns the long reads to the graph. A long read that spans a repeat, with unique sequence on both sides, shows which path is the correct one. Unicycler builds a **bridge** across the repeat and simplifies the graph.
3. **Finalisation**: the graph is cleaned, the circular sequences (chromosome and plasmids) are detected, and they are **rotated** to start at a standard gene (*dnaA* for the chromosome, *repA* for the plasmids), as in reference genomes.

The sequence of the final assembly comes from the short reads (except in the bridges), so it is **accurate**; its structure comes from the long reads, so it is **complete**. This is why a hybrid assembly is better than a short-read assembly (accurate, but fragmented) or a long-read assembly (complete, but with more small errors) alone.

::: {.callout-note title="Short-read-first or long-read-first?"}
Unicycler starts from the short reads and uses the long reads to resolve the repeats. Another strategy is to assemble the long reads first (e.g. with Flye, as in the long read assembly tutorial) and then **polish** the assembly with the short reads (e.g. with Polypolish). With modern, accurate Nanopore reads, the long-read-first approach is now often recommended, but the short-read-first approach of Unicycler remains very robust when the long reads are less accurate or have a low coverage.
:::

# Create a new history

::: {.callout-tip title="Create a new history"}
1. Go to the **History** panel (right side).
2. Click **✏️ → Create new history**.
3. Rename it to **Hybrid Genome Assembly**.
4. Click **Save**.
:::

# Import the trimmed reads

We use the reads cleaned in the [Read Quality Control](assessing-read-quality.md) tutorial:

- the short reads cleaned with **fastp**
- the long reads cleaned with **fastplong**

::: {.callout-tip title="Import input reads"}
1. Open the **History Multiview** (left panel).
2. From the previous history (e.g., "Quality Control and Trimming"), drag into the **Hybrid Genome Assembly** history:
    - **Trimmed Short Read**: the trimmed paired-end short reads
    - **Trimmed Long Read**: the trimmed long reads
:::

# Unicycler in Galaxy

## Choose the parameters

Unicycler was designed to work well with its default parameters, and most of them are appropriate for our data. Let's look at the main ones:

| Parameter | Default | Our choice | Why |
|---|---|---|---|
| *"Select Bridging mode"* | `normal` | `normal` | Controls how bold Unicycler is when building bridges across repeats. `conservative` gives the fewest misassemblies but a more fragmented assembly, `bold` the most complete assembly but more risk of errors. With a high long-read coverage (> 100x), `normal` is a good balance |
| *"Exclude contigs from the FASTA file which are shorter than this length"* | `100` | `100` | Removes tiny contigs. Our smallest plasmid (2.1 kb) is far above this threshold |
| *"Expected number of linear sequences"* | `0` | `0` | The chromosome and plasmids of *E. coli* are all **circular**: we expect no linear sequence |
| k-mer sizes (SPAdes options) | automatic | automatic | Unicycler chooses the k-mer range from the length of the short reads |
| Rotation of the replicons | enabled | enabled | The circular sequences are rotated to start at *dnaA* / *repA*, as in reference genomes |

::: {.callout-note title="Do we have enough data?"}
Unicycler needs a good short-read coverage (ideally > 30x), and works with a long-read coverage as low as ~10x, as the long reads are only used to build bridges. With ~100x of short reads and ~150x of long reads after trimming, we have more than enough.
:::

:::: {.callout-important title="Question: Bridging mode" collapse="true"}
When would you choose the `conservative` or the `bold` bridging mode?

::: {.callout-tip title="Answer" collapse="true"}
- `conservative`: when correctness matters more than completeness, or when the long reads are of poor quality or low coverage: Unicycler only builds bridges it is very confident about.
- `bold`: when completeness matters more, e.g. to close a genome that stays fragmented in `normal` mode, at the cost of a higher risk of misassemblies.
:::
::::

## Run Unicycler

::: {.callout-important title="Exercise: Run Unicycler" collapse="true"}
1. In the Galaxy tools panel, search for **Unicycler** and click on **Create assemblies with Unicycler**.
2. Select the following parameters:
    - *"Paired or Single end data?"*: `Paired Collection`
        - *"Select a paired collection"*: `Trimmed Short Read`
    - *"Select long reads. If there are no long reads, leave this empty"*: `Trimmed Long Read`
    - *"Select Bridging mode"*: `Normal (moderate contig size and misassembly rate)`
    - No change in the other parameters
3. Click **Run Tool**
:::

::: {.callout-note title="Collections as input"}
Our reads are stored in collections with a single sample. If Galaxy does not let you select one of them, extract the dataset from the collection with the **Extract dataset** tool (*"Input List"*: the collection, *"How should a dataset be selected?"*: `The first dataset`) and use its output instead.
:::

The assembly can take **one to several hours**: Unicycler runs SPAdes several times, then aligns all the long reads to the graph.

## Galaxy outputs

Unicycler generates 2 outputs:

- **Unicycler on data X: Final Assembly**: the assembled contigs, in FASTA format
- **Unicycler on data X: Final Assembly Graph**: the assembly graph, in GFA format, to visualise with **Bandage**

The header of each contig in the FASTA file gives its length, its **depth** and whether it is **circular**, for example:

```
>1 length=4894004 depth=1.00x circular=true
```

The depth is **normalised** to the depth of the chromosome (the longest contig), which is set to 1.00x.

# Results

:::: {.callout-important title="Exercise: Explore the hybrid assembly" collapse="true"}
Look at the headers of the **Final Assembly** (use the **Select lines that match an expression** tool with the pattern `^>`, or look at the 👁️ view of the dataset).

1. How many contigs did Unicycler assemble? How many are circular?
2. Compare their lengths with the 7 sequences of the published genome. Did you recover the chromosome and all the plasmids?
3. Look at the depth of the plasmids. What does it tell you?
4. Compare with the long read assembly made with Flye. Which plasmids are recovered by Unicycler but not by Flye? Why?

::: {.callout-tip title="Answer" collapse="true"}
<!-- TODO: add the contigs, lengths, depths and circularity from your own run -->
1. and 2. As the authors used Unicycler on the same reads, we expect a result very close to the published genome: 7 circular contigs, with the chromosome (~4.89 Mb) and the 6 plasmids. Small differences in length are possible, as the reads were trimmed differently and the Unicycler version may differ.
3. As the depth is normalised to the chromosome, the depth of a plasmid estimates its **copy number** per cell: the small plasmids usually have a much higher depth (several copies per cell) than the large ones (~1 copy per cell).
4. The small plasmids (2–8 kb) are often missing from the long read assembly, as their reads are short and were partly removed by the filtering. In the hybrid assembly, they are assembled from the short reads, which cover them well.
:::
::::

::: {.callout-note title="Polishing"}
Older versions of Unicycler (< 0.5) polished the final assembly with **Pilon**, using the short reads. Since version 0.5, this step was removed, as the sequence of a short-read-first assembly is already accurate. If needed, the assembly can be polished afterwards with tools such as **Polypolish**.
:::

# Conclusion

We assembled sample 368 with Unicycler, combining the accurate short reads and the long reads. The short reads gave an accurate sequence, and the long reads resolved the repeats to recover the chromosome and the plasmids as complete, circular sequences. In the next steps, we will compare the quality of the short read, long read and hybrid assemblies with **QUAST**, and visualise their graphs with **Bandage**.

::: {.callout-tip title="Key Points"}
- A hybrid assembly combines the accuracy of short reads with the ability of long reads to span repeats.
- Unicycler builds a short-read assembly graph with SPAdes, and uses the long reads to build bridges across the repeats.
- Unicycler works well with its default parameters; the bridging mode sets the balance between completeness and misassembly risk.
- The depth of each circular contig, normalised to the chromosome, estimates the copy number of the plasmids.
:::

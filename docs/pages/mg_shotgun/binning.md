# MAGs: Assembly and binning shotgun metagenomics data

::: {.callout-note}
**Time:** 120 min

**Objectives:** 

- to be able to perform the *de novo* assembly of shotgun metagenomics dataset

- to use several methods for binning the produced contigs

- to refine the obtained bins to get MAGs (Metagenome Assembled Genomes)

- to assess the taxonomy of the obtained MAGs

**Questions:**
- Which species are present in my sample?

**Tools:** 
[megahit]()
[MetaBat2]()
[MaxBin2]()
[SemiBin]()
[Binette]()
[GTDBTk]()

:::

## Foreword about the metagenomics assembly

Reconstructing the genomes of microorganisms in the sampled communities is critical step in analyzing metagenomic data. To do that, we can use **assembly** and assemblers, *i.e.* computational programs that stich together the small fragments of sequenced DNA produced by sequencing instruments.

Assembling seems intuitively similar to putting together a jigsaw puzzle. Essentially, it looks for reads "that work together" or more precisely, reads that overlap. Tasks like this are **not straightforward**, but rather complex because of the complexity of the genomics (specially the repeats), the missing pieces and the errors introduced during sequencing.

**Metagenomic assembly** is further complicated by

*   the large volume of data produced

*   the quality of the sequence

*   the unequal representation of members of the microbial community

*   the presence of closely related microorganisms with similar genomes

*   the presence of several strains of the same microorganism

*   an insufficient amount of data for minor community members

For metagenomic assembly, several tools exist: [metaSPAdes](https://pmc.ncbi.nlm.nih.gov/articles/PMC5411777/), [MEGAHIT](https://academic.oup.com/bioinformatics/article/31/10/1674/177884), etc. 
The different assemblers have different computational characteristics and their performance varies according to the microbiome as shown in by both rounds of  Critical Assessment of Metagenome Interpretation initiative [CAMI](). The preference of one assembler over another depends on the purpose at hand.

In this tutorial, we will learn how to run metagenomic assembly tool and evaluate the quality of the generated assemblies. To do that, we will use data from the study: [16S rRNA and metagenomic shotgun sequencing data revealed consistent patterns of gut microbiome signature in pediatric ulcerative colitis](https://www.nature.com/articles/s41598-022-07995-7). 

We will use the same 4 datasets that we used previously in the taxonomic profiling.

::: {.callout-handson}

Create a new history for this tutorial. You can call it **MAG** or **binning**

Use the multiview panel, and use the drag and drop function to retrieve the collection of reads that hs been quality controlled and trimmed using fastp when we did the taxonomic profiling exercise.

Once the fastp reads collection in your new MAG history, you are ready to run Megahit.
Search for the **Megahit** in the tool menu.

Provide the fastp collection as input after selecting *Paired end collection*
Keep *Run individually*
Select *Yes* to *Generate log file*
::: 

![](images/megahit.png)

::: {.callout-note}
Please note that this step might take a long time (maybe an hour or two)
:::

Once the assembly done, you can inspect the output files but for getting some metrics from the assembly, we will use Quast.

# Quality control of assembly

Once assembly is done, it is important to check its quality.

Assemblies can be evaluated with **metaQUAST**, the metagenomics mode of **QUAST**.

::: {.callout-handson}

Search for Quast in the tool menu.
- *"Assembly mode?"*: `Individual assembly (1 contig file per samples)`
- *"Use customized names for the input files?"*: `No, use dataset names`
    - {% icon param-collection %} *"Contigs/scaffolds file"*: output **MEGAHIT**
       - *"Reads options"*: `Illumina paired-end reads in paired collection`

 <comment-title>Tip</comment-title>
        To speed up the processing, you may choose `Disabled` here. In this case, raw reads will **not** be mapped back to the assembly to compute metrics such as read coverage.


       - {% icon param-collection %} *"FASTQ/FASTA files"*: `Raw reads`
     - *"Type of assembly"*: `Metagenome`
   - *"Output files"*: `HTML report`, `PDF report`, `Tabular reports`, `Log file`, `Key metric summary (metagenome mode)`, `Krona charts (metagenome mode without reference genomes)`

::: 


--
Megahit
Quast
Mapping for binning
MetaBat
MaxBin
SemiBin

Binette ou MetaWrap ?
GTDB

Write about co-assembly
Map host??

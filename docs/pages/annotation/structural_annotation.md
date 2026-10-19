# Structural annotation

<iframe id="iframepdf" src="../../lectures/annotation/Structural_annotation2026.pdf" frameborder="0" width="1280" height="960" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

<hr>

Structural annotation is the first major stage of genome annotation and represents the transition from a raw DNA sequence to an interpretable genomic landscape. After genome assembly, the sequence itself is only a long succession of nucleotides (A, T, G, and C) without explicit biological meaning. The objective of structural annotation is therefore to identify the functional elements encoded in the genome and determine their precise organization and genomic coordinates.
In other words, structural annotation answers fundamental questions such as:  

* Where are the genes located?
* Where do transcription and translation begin and end?
* How are exons and introns organized?
* Which regions correspond to coding or non-coding elements?
* Which sequences correspond to repetitive DNA,transposable elements, etc.?

The central goal is to reconstruct the architecture of the genome. For protein-coding genes, this includes identifying promoters, transcription start sites, untranslated regions (UTRs), coding sequences (CDS), splice junctions, exons, introns, and transcription termination sites. Structural annotation also seeks to identify non-coding RNA genes such as tRNAs, rRNAs, microRNAs, and long non-coding RNAs, as well as repetitive and mobile elements that can represent a large fraction of many genomes.  

This step is essential because all downstream biological analyses depend on the quality of the structural annotation. Functional annotation, comparative genomics, transcriptomics, evolutionary studies, and variant interpretation all require an accurate description of gene structures. A poorly annotated genome can lead to incorrect biological interpretations, missing genes, fragmented gene models, or erroneous protein predictions.

# Eukaryote annotation

Eukaryote annotation: **big and messy**

# Prokaryote annotation

**compact and efficient**

Prokaryotic annotation is streamlined, can often be done in a few minutes with standard pipelines.

- Compact genomes: usually 1–10 Mb, high gene density.
- Almost no introns: genes are mostly continuous ORFs.
- Few repeats: no need for repeat masking.
- Automatic tools available.
- tRNA/rRNA detection: handled with standard tools (tRNAscan-SE, barrnap).
- Functional annotation: BLAST, HMM profiles, KEGG, eggNOG.

## QC 

You can follow the same approaches than eukaryotes. Completeness done with BUSCO.

## Tools

- Prokka widely used, integrates gene calling + functional annotation. 
- Bakta Newer alternative to Prokka, even better. A gold standard tool!
- PGAP (Prokaryotic Genome Annotation Pipeline, NCBI) Official NCBI pipeline.
- RAST (**Web server**) is a fully-automated service for annotating complete or nearly complete bacterial and archaeal genomes.
- RASTtk (toolkit version) is a fully-automated service for annotating complete or nearly complete bacterial and archaeal genomes. 
- BV-BRC (Bacterial and Viral Bioinformatics Resource Center) A web-based resource that integrates annotation service based on RASTtk, but also comparative genomics, phylogenetics, AMR prediction, metabolic pathway exploration.

# Virus annotation

**tiny but tricky**

Virus annotation is streamlined, can often be done in a few minutes with standard pipelines.

- Compact genomes: usually kb–hundreds of kb.
- Almost no introns.
- Masking generally unnecessary.
- Automatic tools available.

## QC 

Completeness has to be checked with [`checkV`](https://www.nature.com/articles/s41587-020-00774-7).

## Challenges

- Very high diversity across virus families.
- Many ORFs are short and overlapping (gene prediction less straightforward).
- Many proteins are “hypothetical” (unknown function).

## Tools

- Vgas (Viral Genome Annotation System)patr
- VADR: validation and annotation of virus sequence submissions to GenBank. 
- Pharokka: a fast scalable bacteriophage annotation tool - https://doi.org/10.1093/bioinformatics/btac776
- BV-BRC (Bacterial and Viral Bioinformatics Resource Center) is a **Web-based** platform (successor of PATRIC, IRD, ViPR) that integrates annotation service based on RASTtk, but also comparative genomics, phylogenetics, AMR prediction, metabolic pathway exploration.
<iframe id="iframepdf" src="../../lectures/annotation/Structural_annotation2025.pdf" frameborder="0" width="640" height="480" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
 
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
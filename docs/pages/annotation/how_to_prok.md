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
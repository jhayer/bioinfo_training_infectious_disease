# Virus annotation

**tiny but tricky**

Virus annotation is streamlined, can often be done in a few minutes with standard pipelines.

- Compact genomes: usually kb–hundreds of kb.
- Almost no introns.
- Masking generally unnecessary.
- Automatic tools available.

## QC 

Completeness has to be checked with [`checkV`](https://www.nature.com/articles/s41587-020-00774-7).

```bash
checkv --help
checkv end_to_end input.fa myresult  -d /path/to/database/checkv-db-v1.5 
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/checkv:1.0.3--pyhdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/checkv:1.0.3--pyhdfd78af_0 checkv --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && checkv end_to_end input.fa myresult  -d /path/to/database/checkv-db-v1.5"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/checkv:1.0.3--pyhdfd78af_0
# call the help
singularity exec checkv_1.0.3--pyhdfd78af_0.sif checkv --help
# call the check
singularity exec checkv_1.0.3--pyhdfd78af_0.sif checkv end_to_end input.fa myresult -d /path/to/database/checkv-db-v1.5
```
:::

## Challenges

- Very high diversity across virus families.
- Many ORFs are short and overlapping (gene prediction less straightforward).
- Many proteins are “hypothetical” (unknown function).

## Tools

- Vgas (Viral Genome Annotation System)patr
- VADR: validation and annotation of virus sequence submissions to GenBank. 
- Pharokka: a fast scalable bacteriophage annotation tool - https://doi.org/10.1093/bioinformatics/btac776
- BV-BRC (Bacterial and Viral Bioinformatics Resource Center) is a **Web-based** platform (successor of PATRIC, IRD, ViPR) that integrates annotation service based on RASTtk, but also comparative genomics, phylogenetics, AMR prediction, metabolic pathway exploration.
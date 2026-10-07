# Functional annotation

::: {.callout-note}
**Time:** 60 min

**Objectives:** Be able to collect functional information and attach it to a genome annotation.

**Tools:** [AGAT](https://github.com/NBISweden/AGAT), [interproscan](https://interproscan-docs.readthedocs.io/en/v5/), [Blast](https://blast.ncbi.nlm.nih.gov/Blast.cgi)
:::

## Foreword

Answering “what does this gene do?” is challenging. One common strategy is to infer function by similarity: if a protein resembles one with a known role, we assume they share it. Yet this approach can be unreliable. Functional annotation therefore adds multiple layers of information—such as cellular location, pathway involvement, structural features as coiled-coil regions, signal peptides, conserved domains, protein families, etc. Together, these details provide a richer picture than a single function label and form the basis for interpreting new genomes and linking sequences to biological meaning.

## Prerequisites

Setup the folder structure and move in:

```bash
mkdir -p {{< var training_path >}}/function
cd {{< var training_path >}}/function
```

To perform the functional annotation of your protein coding genes you would need the protein fasta file corresponding to your annotation.

```bash
agat_sp_extract_sequences.pl --help
agat_sp_extract_sequences.pl -i annotation.gff --cfs --cis -o annotation.faa 
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_extract_sequences.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_extract_sequences.pl -i annotation.gff -cfs --cis -o annotation.faa "
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_extract_sequences.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_extract_sequences.pl -i annotation.gff -cfs --cis -o annotation.faa 
```
:::

`-cfs` stands for `clean final stop` and `--cis` stands for `clean internal stop`. These parameters are important because InterProScan has difficulty handling stop codons, which are represented as `*` characters in protein FASTA files.  

You are now good to go.

## BLAST 

BLAST searches provide an indication of potential homology to known proteins. A complete BLAST analysis can be time-consuming and may require several gigabytes of RAM. Therefore, when working with large datasets, it is recommended to parallelize the process
This strategy allows you to efficiently assign putative names to genes and predict possible functions for transcripts

You can run BLAST against any protein database of your choice; however, for the purpose of this exercise, it is advisable to use a small database to keep runtimes manageable.

```bash
blastp -h
blastp -db database.fa -query annotation.faa -outfmt 6 -out blast.out -num_threads 8
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/blast:2.17.0--h66d330f_0
# call the help
docker run --rm quay.io/biocontainers/blast:2.17.0--h66d330f_0 blastp --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/blast:2.17.0--h66d330f_0 bash -c "cd data && blastp -db database.fa -query annotation.faa -outfmt 6 -out blast.out -num_threads 8 "
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker:/quay.io/biocontainers/blast:2.17.0--h66d330f_0
# call the help
singularity exec blast_2.17.0--h66d330f_0.sif blastp --help
# call the check
singularity exec blast_2.17.0--h66d330f_0.sif blastp -db database.fa -query annotation.faa -outfmt 6 -out blast.out -num_threads 8
```
:::

Depending the protein length, the database size, the Hardware and the BLAST parameters a single protein blast can takes seconds to minutes.  
As a rough guide:  

- Single short protein (~300 aa) vs. SwissProt: a few seconds.
- Single long protein (~1000 aa) vs. SwissProt: 10–30 seconds.
- Single protein vs. NCBI nr: tens of seconds to several minutes.


### Add your result to the structural annotation.

You can integrate BLASTp functional annotation into your structural GFF/GTF annotation using AGAT. This process adds a name attribute to the gene features and a description attribute (corresponding to the product information) to the mRNA features in your annotation file.

```bash
agat_sp_manage_functional_annotation.pl --help
agat_sp_manage_functional_annotation.pl -f annotation.gff -b blast.out --db database.fa -o annotation_blast.gff  
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_manage_functional_annotation.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_manage_functional_annotation.pl -f annotation.gff -b blast.out --db database.fa -o annotation_blast.gff "
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_manage_functional_annotation.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_manage_functional_annotation.pl -f annotation.gff -b blast.out --db database.fa -o annotation_blast.gff  
```
:::

::: {.callout-question}
How many gene names were assigned?
:::

## Interproscan

InterProScan can be run either through a web interface or from the command line. In this exercise, we will focus on the command-line approach. InterProScan enables the identification of pathways, protein families, domains, sites, structural domains, and other sequence features.

- `-appl`: This option specifies which databases to use. In this case, we will use PfamA, ProDom, and SuperFamily. The list of available databases is available [here](https://interproscan-docs.readthedocs.io/en/v5/HowToRun.html#included-analyses).
- `--goterms` InterProScan maintains an internal database that links public database entries to established Gene Ontology (GO) terms. By including the --goterms option, you can add GO annotations to your dataset.
- `--iprlookup`: This option retrieves the corresponding InterPro identifiers for each motif. For example, the same motif may appear as PF01623 in Pfam and as IPR002568 in InterPro.
- `-pa`: This option provides mappings from matches to pathway information (MetaCyc and Reactome).

Runtime is approximately 2–3 seconds per protein, depending on the number of sequences submitted.  
InterProScan produces three result files with the following extensions: .gff3, .tsv, and .xml.


```bash
interproscan.sh --help
interproscan.sh -i annotation.faa -t p -dp -pa -appl COILS,Pfam --goterms
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/interproscan:5.59-91.0 
# call the help
docker run --rm quay.io/biocontainers/interproscan:5.59-91.0 interproscan.sh --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/interproscan:5.59-91.0 bash -c "cd data && interproscan.sh -i annotation.faa -t p -dp -pa -appl COILS,Pfam --goterms"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker:/quay.io/biocontainers/interproscan:5.59-91.0
# call the help
singularity exec interproscan_5.59-91.0.sif interproscan.sh --help
# call the check
singularity exec interproscan_5.59-91.0.sif interproscan.sh -i annotation.faa -t p -dp -pa -appl COILS,Pfam --goterms
```
:::

::: {.callout-question}
How many database you searched against?
:::

::: {.callout-example title="solution"}
The command used seach against 6 databases. The search is centered on COILS and Pfam database (2),  InterProScan will map the hits to the corresponding InterPro entries (+1=3), then with the `-pa`option activated it will add the pathways linked to the InterPro entries (+2=5), finally with the `--goterms` it will add the GO terms linked to the InterPro entries (+1=6).
:::

### Add your result to the structural annotation.

You can integrate Interproscan functional annotation into your structural GFF/GTF annotation using AGAT. This process adds the matches found within attributes called Dbxref and/or Ontology_term in the gene and transcript feature field (9th column).

```bash
agat_sp_manage_functional_annotation.pl --help
agat_sp_manage_functional_annotation.pl -f annotation_blast.gff -i annotation.faa.tsv -o  annotation_blast_interpro.gff
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_manage_functional_annotation.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_manage_functional_annotation.pl -f annotation_blast.gff -i annotation.faa.tsv -o annotation_blast_interpro.gff "
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_manage_functional_annotation.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_manage_functional_annotation.pl -f annotation_blast.gff -i annotation.faa.tsv -o annotation_blast_interpro.gff  
```
:::

## Remarks

Functional annotation can also serve as a quality check for your annotation. For example, if you are undecided between two versions of an annotation, prefer the one in which you recover a greater number of protein domains.
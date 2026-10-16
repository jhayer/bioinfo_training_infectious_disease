# Quality Check of a Genome Annotation

::: {.callout-note}
**Time:** 30 min

**Objectives:** Be able to assess the quality of an annotation, and compare against an existing reference annotation.

**Tools:** [AGAT](https://github.com/NBISweden/AGAT), [BUSCO](https://busco.ezlab.org), [checkM2](https://github.com/chklovski/CheckM2)
:::

## Foreword

Once an annotation is produced, it is essential to assess its reliability and completeness. There is no single “perfect” metric, and each method highlights different aspects of annotation quality. In practice, evaluation usually involves combining several complementary approaches.

## Statistics

General statistics like number of predicted genes, their length, number of intron, etc gives you a sense of congruency with other builds or expectations for the species. However, it’s not a true quality metric: two annotations may have different gene counts without one being objectively “more correct.”

```bash
agat_sp_statistics.pl --help
agat_sp_statistics.pl -i annotation.gff -o annotation.stat 
```


::: {.callout-question}
How many gene do you have?  
Do you have isoforms?  
What is the percentage of genome covered by genes?  
:::

## BUSCO

[BUSCO](https://busco.ezlab.org) (Benchmarking Universal Single-Copy Orthologs) searches for sets of conserved genes that are expected to be present in nearly all genomes of a given lineage. The results indicate how many of these genes are complete, duplicated, fragmented, or missing.  
In this case BUSCO is run on proteins derived from the annotation. This is useful for benchmarking, but it has an inherent bias: it emphasizes conserved genes across species and may overlook lineage-specific or novel genes.

::: {.callout-note}
:::
   The BUSCO results on proteins derived from the annotation should be similar to those obtained from BUSCO on the assembly itself. If the annotation shows significantly lower completeness, it indicates that there is room for improvement in your annotation process.

You need the protein set in fasta format of your annotation in order to run this BUSCO. If the annotation tool did not provide that data, you can extract the proteins from the GTF/GFF annotation using AGAT:

```bash
agat_sp_extract_sequences.pl --help
agat_sp_extract_sequences.pl -g annotation.gff -f assembly_masked.fa -p -o annotation.fa
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_extract_sequences.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_extract_sequences.pl --g annotation.gff -f assembly_masked.fa -p -o annotation.fa"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_extract_sequences.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_extract_sequences.pl -g annotation.gff -f assembly_masked.fa -p -o annotation.fa
```
:::

Then you can run the BUSCO on your protein set.

```bash
busco -h
busco -m proteins -i annotation.fa -o annotation_busco
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/busco:6.0.0--pyhdfd78af_0
# call the help
docker run --rm  quay.io/biocontainers/busco:6.0.0--pyhdfd78af_0 busco -h
# call the check
docker run --rm -v .:/data quay.io/biocontainers/busco:6.0.0--pyhdfd78af_0 bash -c "cd data && busco -m proteins -i annotation.fa -o annotation_busco"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/busco:6.0.0--pyhdfd78af_0
# call the help
singularity exec  busco_6.0.0--pyhdfd78af_0.sif busco -h
# call the check
singularity exec  busco_6.0.0--pyhdfd78af_0.sif busco -m proteins -i annotation.fa -o annotation_busco
```
:::

::: {.callout-tip}
To speed up the analysis, you can use multiple CPUs with the `-c` option (e.g. `-c 4` for 4 cores).  
By default, BUSCO automatically selects the lineage dataset it considers most appropriate for your assembly. However, you can specify a dataset manually using the `-l` option (e.g. `-l vertebrata_odb12` or `-l arthropoda_odb12`).
To see the list of all available datasets, run `busco --list-datasets`.
:::


## Comparison

A powerful way to gain insight into your annotation is to compare it with alternative builds.

### Against another annotation build

Comparing with a build produced by another pipeline or team can help you identify major structural or methodological differences. This sheds light on where annotations diverge and which regions or gene models may need closer inspection.

```bash
agat_sp_compare_two_annotations.pl --help
agat_sp_compare_two_annotations.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_structural
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_compare_two_annotations.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_compare_two_annotations.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_structural"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_compare_two_annotations.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_compare_two_annotations.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_structural
```
:::
::: {.callout-question}
How many genes are specific to each build?
:::

### Against a trusted reference

This approach is particularly useful for evaluating the behavior of annotation tools. The idea is to take an assembly that already has a well-curated annotation (e.g. a model organism), re-annotate it, and then compare the new results against the trusted reference.

It is also valuable when training new ab initio models: here, the predicted gene models are compared to the “reference” models provided as training input. These trusted models are typically built from strong experimental evidence such as protein or transcript data, making them a reliable benchmark for assessing prediction accuracy.

::: {.callout-note}
if you want to compare with a reference genome, use the reference.gff in the --gff2 parameter
:::

```bash
agat_sp_sensitivity_specificity.pl --help
agat_sp_sensitivity_specificity.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_sesp
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_sensitivity_specificity.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_sensitivity_specificity.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_sesp"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_sensitivity_specificity.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_sensitivity_specificity.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_sesp
```
:::
::: {.callout-question}
What is the specificity and the sensitivity of your annotation?
:::

## Evidence-based congruency

Some tools add congruence score to assess how well predicted models agree with the supporting evidence (transcripts, proteins, etc). It can then be used to filter gene models or compare the overall quality between two gene builds.


- MAKER annotation tool uses Annotation Edit Distance (AED): A gene models with a low AED score are well supported, while a AED score of 1 mean the gene model is full *ab initio* (not supported). 

- GeMoMa provide lot of information, here the interesting one:
| Type of Evidence | Attribute(s) in the GFF | Description |
| -- | -- | -- |
| Protein homology | score, iAA, pAA | Measures how well models align with references |
| RNA-seq (intron support) | tie | Fraction of introns supported by RNA-seq reads |
| RNA-seq (coverage) | tpc | Proportion of transcript covered by RNA-seq |

To filter gene models based on these type of information we can use `agat_sp_filter_feature_by_attribute_value.pl`

```bash
agat_sp_filter_feature_by_attribute_value.pl --help
agat_sp_filter_feature_by_attribute_value.pl --gff1 annotation1.gff --gff2 annotation2.gff -o annotation1_vs_annotation2_sesp
```

::: {.callout-note title="docker"}
```bash
# get the tool
docker pull quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
docker run --rm quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 agat_sp_filter_feature_by_attribute_value.pl --help
# call the check
docker run --rm -v .:/data quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0 bash -c "cd data && agat_sp_filter_feature_by_attribute_value.pl --gff annotation.gff -a iAA --value 50 -o annotation_iAA50filter"
```
:::
::: {.callout-note title="singularity"}
```bash
# get the tool
singularity pull docker://quay.io/biocontainers/agat:1.5.1--pl5321hdfd78af_0
# call the help
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_filter_feature_by_attribute_value.pl --help
# call the check
singularity exec agat_1.5.1--pl5321hdfd78af_0.sif agat_sp_filter_feature_by_attribute_value.pl --gff annotation.gff -a iAA --value 50 -o annotation_iAA50filter
```
:::

::: {.callout-question}
GeMoMA: How many genes are over 50% of identical amino acids between reference protein and prediction (iAA)?  
MAKER: How many genes have a AED score under 0.3?
:::

## Visualising annotation

All information we gather until here are very interessting to assess the quality of the annotation, but they don’t always reflect biological relevance.
Indeed, two gene models of the same locus convey the same information—that a gene exists in a given region, but with tiny differences like (ORF start, different splice site)  may completly differ in their product. Tools apply fixed rules to integrate information without questioning the reliability of the inputs. They do not always recognize systematic errors or inconsistencies. Human pattern recognition remains efficient. This is why visual inspection is crucial.

For this, annotators use annotation viewers such as IGV or JBrowse.

* Load the assembly
* Load annotation
* Load evidence tracks (bam, gff from protein/transcript alignment)
* Laod other tracks (repeats, tRNA, RFAM, etc.)

See dedicated section on how visualise annotation data [here](visualisation).
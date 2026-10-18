# AMR genes and virulence factors

::: {.callout-note}
**Time:** 1h30

**Objectives:** Be able to screen a bacterial genome assembly for antimicrobial resistance (AMR) genes, virulence factors and plasmid replicons with ABRicate in Galaxy, and to interpret the results.

**Questions:**

- Which AMR genes are present in my genome, and which antibiotic classes could they affect?
- Are these genes on the chromosome or on a plasmid?
- Does my genome carry known virulence factors?
- How do results change depending on the database and the thresholds used?

**Tools:** [FASTA Width formatter](https://toolshed.g2.bx.psu.edu/view/devteam/fasta_formatter), [ABRicate](https://github.com/tseemann/abricate) and ABRicate Summary
:::

## Foreword

Antimicrobial resistance (AMR) is one of the major threats to global health. Whole-genome sequencing allows us to predict the resistance potential of a bacterial isolate by looking for genes known to confer resistance (acquired resistance genes), and to look at its virulence potential and its mobile genetic elements such as plasmids.

ABRicate is a mass screening tool of contigs for antimicrobial resistance or virulence genes. It aligns the contigs of an assembly (BLASTn) against curated databases of reference genes and reports the hits that pass identity and coverage thresholds.

ABRicate comes with several databases:

| Database | Content |
| --- | --- |
| `ncbi` | NCBI AMRFinderPlus reference gene catalog (AMR) |
| `card` | Comprehensive Antibiotic Resistance Database (AMR) |
| `resfinder` | ResFinder, acquired AMR genes (AMR) |
| `argannot` | ARG-ANNOT (AMR) |
| `megares` | MEGARes (AMR) |
| `vfdb` | Virulence Factor DataBase (virulence) |
| `ecoli_vf` | *E. coli* virulence factors (virulence) |
| `ecoh` | *E. coli* O and H antigen typing (serotype) |
| `plasmidfinder` | Plasmid replicons (plasmid) |

In this tutorial we will use `ncbi` and `card` for AMR genes, `vfdb` for virulence factors and `plasmidfinder` for plasmid replicons.

::: {.callout-warning}
ABRicate only detects **acquired genes** by nucleotide similarity. It does **not** detect resistance due to point mutations (e.g. *gyrA*/*parC* mutations conferring fluoroquinolone resistance). For this, use tools such as [AMRFinderPlus](https://github.com/ncbi/amr) (`--organism` option) or [PointFinder](https://bitbucket.org/genomicepidemiology/pointfinder). Also, the presence of a gene does not always mean the phenotype is resistant: always confirm with antimicrobial susceptibility testing (AST).
:::

## Prerequisites

You need the genome assembly uploaded in the [Data preparation](get_data.md) step, in your Galaxy Australia history.

::: {.callout-note}
The answers given in this page were obtained with the assembly **GCA_964199995.1** (*E. coli* isolate 364_BP0073). This assembly is complete: one chromosome (`NZ_OZ121069.1`, 4,682,057 bp) and two plasmids, `p1` (`NZ_OZ121070.1`, 150,301 bp) and `p2` (`NZ_OZ121071.1`, 4,145 bp). If you chose another assembly, your results will be different: compare them with your neighbours!

You can look at the complete example history here: [AMR_annotation_test](https://usegalaxy.org.au/u/rahi_praveen/h/amr-annotation-test).
:::

## Prepare the assembly

The FASTA file downloaded from NCBI has its sequences split over many lines. We first reformat it so each sequence is on a single line.

* Search for the `FASTA Width formatter` tool in Galaxy ![](/pages/images/galaxy/tools.png)

* In `Library to re-format`, select your genome assembly.

* Set `New width for nucleotides strings` to `0` (0 means each sequence is written on a single line).

* Click on `Run tool`.

## Running ABRicate

We will run ABRicate five times on the reformatted assembly, with a different database or threshold each time:

| Run | Database | Minimum DNA %identity | Minimum DNA %coverage | Rename output to |
| --- | --- | --- | --- | --- |
| 1 | `ncbi` | 80 | 80 | `NCBI_Ref_report` |
| 2 | `card` | 80 | 80 | `CARD_report` |
| 3 | `card` | 90 | 90 | `CARD90_report` |
| 4 | `vfdb` | 80 | 80 | `VF_report` |
| 5 | `plasmidfinder` | 80 | 80 | `PF_report` |

For each run:

* Search for the `ABRicate` tool in Galaxy.

* In `Input file (Fasta, Genbank or EMBL file)`, select the output of `FASTA Width formatter`.

* Open `Advanced options`, select the database in `Database to use` and set `Minimum DNA %identity` and `Minimum DNA %coverage` as in the table above.

* Click on `Run tool`.

::: {.callout-tip}
Rename each output in your history as in the table (`NCBI_Ref_report`, `CARD_report`, ...) so you can tell them apart. Click on the dataset, then on the pencil icon to edit its name.
:::

=> Look at the result by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `NCBI_Ref_report` line in your history on the right side.

## Understanding the output

The output is a tab-separated table with one line per hit:

| Column | Description |
| --- | --- |
| `SEQUENCE` | Contig where the gene was found |
| `START` / `END` / `STRAND` | Position of the hit on the contig |
| `GENE` | Name of the AMR / virulence gene |
| `COVERAGE` | Range of the reference gene covered by the hit |
| `COVERAGE_MAP` | Visual map of the coverage (`=` aligned, `.` missing, `/` gap) |
| `GAPS` | Number of gaps in the alignment |
| `%COVERAGE` | Proportion of the reference gene covered |
| `%IDENTITY` | Proportion of identical nucleotides |
| `DATABASE` | Database used |
| `ACCESSION` | Accession of the reference gene |
| `PRODUCT` | Gene product description |
| `RESISTANCE` | Predicted antibiotic class(es) affected |

## AMR genes

::: {.callout-caution collapse="true"}
## How many AMR genes were detected with the `ncbi` database?
Count the number of lines in the table (excluding the header).

For GCA_964199995.1: **16 hits**, corresponding to 15 different genes. `blaCTX-M-55` is found twice, at two different positions of plasmid `p1` (two copies of the gene).
:::

::: {.callout-caution collapse="true"}
## Which antibiotic classes is this isolate predicted to be resistant to?
Look at the `GENE`, `PRODUCT` and `RESISTANCE` columns.

For GCA_964199995.1:

| Class | Genes |
| --- | --- |
| Beta-lactams, incl. 3rd-generation cephalosporins | `blaCTX-M-55` (×2, extended-spectrum beta-lactamase, ESBL), `blaTEM-1` |
| Aminoglycosides | `aadA1`, `aph(3'')-Ib`, `aph(6)-Id` (streptomycin), `aph(3')-Ia` (kanamycin), `aac(3)-IIe` (gentamicin) |
| Phenicols | `floR` (chloramphenicol/florfenicol), `catA2` (chloramphenicol) |
| Sulfonamides | `sul2`, `sul3` |
| Trimethoprim | `dfrA14` |
| Tetracyclines | `tet(A)` |
| Macrolides | `estX-3` |

This isolate carries acquired resistance genes for 7 antibiotic classes, including an ESBL. If confirmed by AST, it would be considered multidrug-resistant (MDR).

The last hit, `blaEC-19`, is on the chromosome: `blaEC` genes are the chromosomal AmpC beta-lactamase naturally present in all *E. coli* (intrinsic, not acquired).

No fluoroquinolone resistance gene was found, but remember that ABRicate cannot see *gyrA*/*parC* point mutations!
:::

::: {.callout-caution collapse="true"}
## Are AMR genes located on the chromosome or on plasmids?
Look at the `SEQUENCE` column.

For GCA_964199995.1: 15 of the 16 hits are on `NZ_OZ121070.1`, plasmid `p1`, between positions ~29,000 and ~80,000. Only the intrinsic `blaEC-19` is on the chromosome. All the acquired resistance genes are clustered in a single region of one plasmid, which could be transferred to other bacteria by conjugation.
:::

::: {.callout-caution collapse="true"}
## Are some genes found with `%COVERAGE` or `%IDENTITY` below 100%? What could it mean?
A hit with lower coverage can be a truncated gene (possibly non-functional), a gene split across two contigs because the assembly is fragmented, or a divergent variant of the reference gene. A hit at the edge of a contig should be interpreted with care.

For GCA_964199995.1 with `ncbi`, all hits are almost perfect (≥ 99.75% identity and ≥ 99.88% coverage): these are the reference genes themselves.
:::

## Comparing databases and thresholds

Now open `CARD_report`.

::: {.callout-caution collapse="true"}
## Do `ncbi` and `card` report the same genes? Why?
No. For GCA_964199995.1, `card` reports **59 hits** against 16 for `ncbi`:

- On plasmid `p1`, both databases find 15 hits and mostly the same acquired genes, with some differences: gene names (`blaTEM-1` vs `TEM-1`, `catA2` vs `catII_from_Escherichia_coli_K-12`, ...), `estX-3` is only found with `ncbi` and `qacL` (resistance to quaternary ammonium disinfectants) only with `card`.
- On the chromosome, `card` reports 44 hits against 1 for `ncbi`. They are mostly efflux pumps and their regulators (`acrB`, `TolC`, `mdtEF`, `emrAB`, `marA`, ...), which are present in all *E. coli*. They contribute to the intrinsic resistance of the species, but they are not acquired genes and do not mean the isolate is resistant.

This is why many antibiotic classes appear in the `RESISTANCE` column of `card` (e.g. fluoroquinolone, rifamycin), even though no acquired gene for these classes was found. Databases differ in content, curation and gene naming: always report results together with the database name and version.
:::

Now open `CARD90_report`, obtained with stricter thresholds (90% identity and 90% coverage).

::: {.callout-caution collapse="true"}
## What changes when you increase the thresholds to 90%?
For GCA_964199995.1: 58 hits instead of 59. The missing gene is `OmpA` (outer membrane porin), which was found with only 83.66% identity and 95.82% coverage.

Higher thresholds give fewer but more reliable hits, and lower thresholds can catch divergent variants at the risk of false positives. The default ABRicate thresholds are 80% identity and 80% coverage.
:::

## Virulence factors

Open `VF_report`.

::: {.callout-caution collapse="true"}
## Which virulence factors were found? Are they expected for *E. coli*?
For GCA_964199995.1: **97 hits**, 92 on the chromosome and 5 on plasmid `p1`. Look at the `PRODUCT` column, which gives the virulence factor and its category between brackets. The main systems are:

| Category | Virulence factor | Genes |
| --- | --- | --- |
| Iron uptake | Yersiniabactin (siderophore of the High Pathogenicity Island) | `ybt*`, `irp1`, `irp2`, `fyuA` |
| Iron uptake | Aerobactin (**on plasmid `p1`**) | `iucABCD`, `iutA` |
| Iron uptake | Enterobactin | `ent*`, `fep*`, `fes` |
| Adhesion | Type 1 fimbriae, curli, *E. coli* common pilus, FdeC | `fim*`, `csg*`, `ecp*`, `fdeC` |
| Invasion | Invasion of brain endothelium | `ibeB`, `ibeC` |
| Secretion | Type VI secretion system | `tssA`, `hcp1`, `rhs/PAAR` |
| Motility | Flagella and chemotaxis | `fli*`, `flg*`, `che*`, `mot*` |

Many of these genes (adhesins, enterobactin, flagella) are found in most *E. coli*. Several iron uptake systems (yersiniabactin and aerobactin) are more often associated with extraintestinal pathogenic *E. coli* (ExPEC). None of the hallmark toxin genes of intestinal pathotypes, such as `stx` (Shiga toxin) or `eae` (intimin), were found.
:::

::: {.callout-caution collapse="true"}
## Why do the flagellar genes (`fli`, `flg`, `che`, `mot`) have a low identity (~80–87%)?
Look at the end of the `PRODUCT` column: the reference sequences of these genes in VFDB come from *Enterobacter cloacae*, not from *E. coli*. The *E. coli* genes are homologous but divergent, so they are just above the 80% identity threshold. Always check which organism the reference comes from before interpreting a hit.
:::

## Plasmid replicons

Open `PF_report`.

::: {.callout-caution collapse="true"}
## Which plasmid replicons were found and on which sequences?
For GCA_964199995.1: 3 replicons.

| Sequence | Replicon | Comment |
| --- | --- | --- |
| `NZ_OZ121070.1` (plasmid `p1`, 150 kb) | `IncFIB(AP001918)` and `IncFIC(FII)` | A large IncF plasmid with two replicons |
| `NZ_OZ121071.1` (plasmid `p2`, 4 kb) | `Col(pHAD28)` | A small Col plasmid |

Combined with the previous results: plasmid `p1` is an IncF plasmid carrying 15 acquired AMR genes **and** the aerobactin virulence genes. IncF plasmids are frequently found in *E. coli* and can spread resistance and virulence together. Plasmid `p2` carries no AMR gene.
:::

::: {.callout-caution collapse="true"}
## The `IncFIB(AP001918)` hit has only 88.42% coverage, and starts at position 1. Why?
Look at the `COVERAGE` column: positions 80 to 682 of the reference (682 bp) are aligned, starting at the very first base of the plasmid sequence. Plasmids are circular, but they are stored as linear sequences in a FASTA file. The replicon most likely sits across the "start" of the circular sequence: the missing 79 bp are at the end of the sequence and ABRicate cannot join both parts. This is a good example of a hit with lower coverage that is not a truncated gene.
:::

## Comparing databases with ABRicate Summary

ABRicate Summary combines one or several ABRicate reports into a presence/absence matrix: one line per input report, one column per gene, with the `%COVERAGE` of the hit (or `.` when absent).

* Search for the `ABRicate Summary` tool in Galaxy.

* Select one of your ABRicate reports, e.g. `NCBI_Ref_report`.

* Click on `Run tool`.

* Do the same for the other reports, and rename the outputs `NCBI_Ref_summary`, `CARD_summary`, `CARD90_summary`, `VF_summary` and `PF_summary`.

::: {.callout-caution collapse="true"}
## The `NUM_FOUND` column of `NCBI_Ref_summary` is 15, but `NCBI_Ref_report` has 16 lines. Why?
ABRicate Summary counts genes, not hits. `blaCTX-M-55` is present twice: it has one column, with the value `100.00;100.00` (one coverage per copy).
:::

::: {.callout-tip}
The summary is most useful when you analyse several genomes: select the reports of all your genomes for the same database in a single ABRicate Summary run, to get one line per genome and compare their gene content at a glance.
:::

## Run the whole analysis as a workflow

All the steps above have been bundled into a Galaxy workflow: `FASTA Width formatter` → 5 × `ABRicate` → 5 × `ABRicate Summary`.

* Import the workflow from: [AMR ABRicate](https://usegalaxy.org.au/published/workflow?id=537e1a5d5896e431)

* Click on `Workflows` in the activity bar on the left, then on the run icon next to the imported workflow.

* Select your genome assembly as input and click on `Run workflow`.

::: {.callout-note}
To learn how to build your own workflow from a history, see [Create a Galaxy workflow](/pages/course/create_galaxy_workflow.md).
:::

## Remarks

- ABRicate is fast and easy to use, but limited to acquired genes detected by nucleotide similarity. For a more complete prediction (point mutations, protein-level search), consider AMRFinderPlus, also available in Galaxy.
- Always report the database name and its version/date, as databases are regularly updated.
- Results depend on the database and on the thresholds: report them too.
- Knowing whether a gene is on the chromosome or on a plasmid is important to assess the risk of transmission. This is much easier with a complete assembly like GCA_964199995.1 than with a fragmented draft assembly.
- Genotype-to-phenotype prediction is not perfect: confirm with antimicrobial susceptibility testing.

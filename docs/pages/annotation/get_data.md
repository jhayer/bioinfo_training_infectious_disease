# Get data and information

::: {.callout-note}
**Time:** 15 min

**Objectives:** Be able to upload data needed.

**Prerequisites:** You should have access to the GALAXY platform and be familiar with basic navigation within it.
:::

Connect to your galaxy account. 

## Download Genome Assembly Data

To make it simple we will work on a single genome assembly file, but the same steps can be applied to multiple files if needed.
I offer you the choice to use the assembly of your choice among a list of Escherichia coli assemblies Accession.The accesion number uniquely identifies a genome assembly in public databases.


| Accession  | URL |
| ---   | --- | 	
| GCA_964200045.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/200/045/GCF_964200045.1_366_BP0076U_scaffolds/GCF_964200045.1_366_BP0076U_scaffolds_genomic.fna.gz |
| GCA_964200025.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/200/025/GCF_964200025.1_372_BP0083B_scaffolds/GCF_964200025.1_372_BP0083B_scaffolds_genomic.fna.gz |
| GCA_964200035.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/200/035/GCF_964200035.1_368_BP0078U_scaffolds/GCF_964200035.1_368_BP0078U_scaffolds_genomic.fna.gz |
| GCA_964200015.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/200/015/GCF_964200015.1_323_BP0014P_scaffolds/GCF_964200015.1_323_BP0014P_scaffolds_genomic.fna.gz |
| GCA_964199995.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/199/995/GCF_964199995.1_364_BP0073U-1_scaffolds/GCF_964199995.1_364_BP0073U-1_scaffolds_genomic.fna.gz |
| GCA_964200005.1 | https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/964/200/005/GCF_964200005.1_327_BP0021B_scaffolds/GCF_964200005.1_327_BP0021B_scaffolds_genomic.fna.gz |


* Click on the "Upload Data" button in Galaxy to open the upload interface ![](/pages/images/galaxy/upload.png).

* Then click on "Paste/Fetch Data" to open the interface where you can paste the URLs of the genome assemblies you want to upload.

![](/pages/annotation/images/get_data1.png){width="900"}

* Now 1) provide a file name ; 2) paste the URLs of the genome assemblies you want to upload into the provided text area ; 3) click on start to begin the upload process.

![](/pages/annotation/images/get_data2.png){width="900"}


=> As a result you should see the uploaded genome assembly files listed in your Galaxy history on the right side of the interface:

![](/pages/annotation/images/get_data3.png){width="450"}

## Get taxonomical information

This step is for sure only possible if you know what organism your genome assembly belongs to.

* Go the [NCBI Taxonomy](taxonomy ncbi) website.

* We are currenlty working with Escherichia coli, so you can search for "Escherichia coli".

::: {.callout-caution collapse="true"}
## Question: What is the full lineabe of number?

Lineage (full): cellular organisms; Bacteria; Pseudomonadati; Pseudomonadota; Gammaproteobacteria; Enterobacterales; Enterobacteriaceae; Escherichia; Escherichia coli
:::

::: {.callout-caution collapse="true"}
## What is the Taxonomy ID of Escherichia coli?

A taxonomy ID is a unique identifier assigned to each organism in the NCBI Taxonomy database. The Taxonomy ID of Escherichia coli is 562.
:::






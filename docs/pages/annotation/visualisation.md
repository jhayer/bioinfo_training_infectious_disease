# Genome Annotation Visualisation

::: {.callout-note}
**Time:** 30 min

**Objectives:** Be able to visualize annotation, and all related data (GFF/BAM/FASTA) .

**Tools:** [Jbrowse2](https://jbrowse.org/jb2/) or [IGV](https://igv.org)
:::

## Prerequisites

⚠️ Make sure your data is available locally.  
It will make your life easier to create a dedicated folder where you can collect all the files you plan to visualize:  

```bash
mkdir -p {{< var training_path >}}/visualisation
cd {{< var training_path >}}/visualisation
```
If you have been working on a remote server, you will first need to copy the files to your local machine using tools such as `rsync` or `scp`.
If you have been working locally a `cp` command will be suffucient.

## JBrowse2

JBrowse2 is a pluggable, open-source computational platform for integrating many kinds of biological data from many different places

**Features**

- Improved visualization of structural variants and genome synteny with tightly-integrated linear, circular, dotplot, and synteny views
- In-browser protein 3D structure displays with integrated alignment and phylogenetic tree views. Mouse over a genomic variant to see where it lands on the structure!
- Support for many common data types including BAM, CRAM, tabix indexed VCF, GFF, BED, BigBed, BigWig, and several specialized formats
- Powerful extensibility with a plugin ecosytem which can add additional view types, track types, data adapters, and more!

### Get JBrowse2

Download JBrowse2 [here](https://jbrowse.org/jb2/download/), install it and open it.

![](../images/visualisation/jb_open.png){style="height:400px;"}

### Load a genome

The first step is to add a reference genome. 

- choose `open a sequence file`
- Give an assembly name 
- Select `FastaAdapter` for the `Type` as we will use an assembly fasta file as input
- Use `Choose file` to select on your computer the fasta file you want to load
- click `SUBMIT`  
![](../images/visualisation/jb_reference.png){style="height:600px;"}
- Jbrowse suggest then to select the type of view for your data. Choose `Linear genome view` and click `launch view`
![](../images/visualisation/jb_linear_genome_view.png){style="height:100px;"}
- A last window appears you just have to click open.  
![](../images/visualisation/jb_reference_open.png){style="height:100px;"}

At the top, you can see the sequence you are visualizing along with its size. You can move around and zoom in or out freely. For now, however, the nucleotide sequence itself will not be displayed.  
![](../images/visualisation/jb_reference_raw.png){style="height:112px;"}


**Visulalise your genome**

- open the `track selector`. It is a right panel where you can select the available tracks you can display.  

![](../images/visualisation/jb_track_selector.png){style="height:112px;"}

- Tick the box to display the reference sequence. Now if you zoom enough you will be able to see the data!

![](../images/visualisation/jb_reference_ok.png){style="height:150px;"}

### Add tracks

To add a new track, you can either click `File>Open track` at the top left of the page or go to the `+` button at the bottom right (red arrows).

![](../images/visualisation/jb_track_open.png){style="height:150px;"} 

Then choose the file you want to load. Do not forget to add the index of the file for format that need it (e.g. `bam` files require `.bai` indicies)

![](../images/visualisation/jb_add_annotation_track.png){style="height:400px;"}

Then the track type is automatically detected based on the suffix, you can change this via `adapter type`.  Some other options will also be available.

![](../images/visualisation/jb_add_annotation_track2.png){style="height:600px;"}

Once loaded you should be a able to see the information hold by the selected tracks as below for an annotation track:

![](../images/visualisation/jb_track_ok.png){style="height:150px;"}

## IGV

The Integrative Genomics Viewer (IGV) is a high-performance, easy-to-use, interactive tool for the visual exploration of genomic data. It supports flexible integration of all the common types of genomic data and metadata.

**Features**

- Optimized for BAM/CRAM/VCF and other read-based data.
- Handles very large files efficiently (streaming support).
- Popular in clinical and research labs for variant inspection.

### Get IGV

Download IGV [here](https://igv.org/doc/desktop/#DownloadPage/), install it and open it.  
A human reference genome is already present by default with the refseq annotation.

![](../images/visualisation/igv_open.png){style="height:200px;"}

### Load a genome

The first step is to add a reference genome `Genomes>Load Genome from File`, select the reference assembly fasta file and click `open`.  
You should now see the name of your file in the top right box that indicates you are visualising this data. At the bottom you can see the nucleotides of the sequence if you zoom enough.

![](../images/visualisation/igv_reference.png){style="height:200px;"}

### Add tracks

To add a new track you must use `File>Load From File`, select the file you want to load e.g. a GFF3 file, and click `open`. The new track should appear below the Sequence.

![](../images/visualisation/igv_reference_collapsed.png){style="height:160px;"}

You may change some parameters with a right click on the track:  

- rename the track
- by default the features are collapsed and it can be unreadable, choose the `expanded` parameter.

![](../images/visualisation/igv_reference_expanded.png){style="height:300px;"}
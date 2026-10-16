# Genome Annotation Visualisation

::: {.callout-note}
**Time:** 15 min

**Objectives:** Be able to visualize annotation, and all related data (GFF/BAM/FASTA) .

**Tools:** [Jbrowse2](https://jbrowse.org/jb2/)

Related ressources : https://training.galaxyproject.org/training-material/topics/visualisation/tutorials/jbrowse2/tutorial.html
:::

## Foreword

Visualizing genome annotations is crucial for understanding the structure and function of genomic elements. Tools like [JBrowse2](https://jbrowse.org/jb2/) and [IGV](https://igv.org) allow researchers to explore genomic data interactively, providing insights into gene organization, variant effects, and other genomic features. In this section, we will focus on using JBrowse2 for visualizing bacterial genome annotations.

## JBrowse2

JBrowse2 is a pluggable, open-source computational platform for integrating many kinds of biological data from many different places

**Features**

- Improved visualization of structural variants and genome synteny with tightly-integrated linear, circular, dotplot, and synteny views
- In-browser protein 3D structure displays with integrated alignment and phylogenetic tree views. Mouse over a genomic variant to see where it lands on the structure!
- Support for many common data types including BAM, CRAM, tabix indexed VCF, GFF, BED, BigBed, BigWig, and several specialized formats
- Powerful extensibility with a plugin ecosytem which can add additional view types, track types, data adapters, and more!

* Search for the `Jbrowse2` toolkit in Galaxy ![](/pages/images/galaxy/tools.png)

* Select your reference genome from history in `Reference genome source` section.

* Click on `Insert Track Group` ![](/pages/annotation/images/visualisation/trackgroup.png)

* Click on `Insert Annotation Track` ![](/pages/annotation/images/visualisation/annotationtrack.png)

* Select the `Track Type`  to GFF/GFF3 track.

* Select `Track data from a history file`

* Select the `Bakta Annotation and sequences (GFF3)` file.

* Run the tool

![](/pages/annotation/images/visualisation/jbrowse2_launch.png){width="600"} 


=> On Jbrowse2 Genome browser by clicking on <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" role="img" aria-label="Scan eye"><path d="M3 7V5a2 2 0 0 1 2-2h2"/><path d="M17 3h2a2 2 0 0 1 2 2v2"/><path d="M21 17v2a2 2 0 0 1-2 2h-2"/><path d="M7 21H5a2 2 0 0 1-2-2v-2"/><circle cx="12" cy="12" r="1"/><path d="M5 12s2.5-5 7-5 7 5 7 5-2.5 5-7 5-7-5-7-5"/></svg> on the `Jbrowse2` line in your history on the right side.

We can now oberve all the annotation features on their sequences.


* Now click on ![](/pages/annotation/images/visualisation/jbrowse2_menu.png) to display the `Available tracks` right menu bar.

* Now select `Reference sequence` and the `annotation gff3` track from the available tracks.

![](/pages/annotation/images/visualisation/jbrowse2_view.png)

::: {.callout-caution collapse="true"}
## Play with the browser and the zoom to check start and end positions of a feature.
You should find something like that for a start:
![](/pages/annotation/images/visualisation/jbrowse2_start.png)  
You should find something like that for an end:
![](/pages/annotation/images/visualisation/jbrowse2_end.png)
:::
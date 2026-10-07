# Lecture

<iframe id="iframepdf" src="../../lectures/annotation/Introduction_annotation2025.pdf" frameborder="0" width="1280" height="960" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

<hr>

Genome annotation is a critical step in making sense of raw DNA sequence data. While sequencing reveals the letters of the genetic code, annotation interprets that code—identifying regions of interest (e.g. genes, regulatory elements, repeat) within the genome. Without annotation, genomes remain unreadable and biologically meaningless. Annotation enables researchers to understand how organisms function, how genes are regulated, and how genetic variations lead to phenotypic traits or disease. Broadly, genome annotation involves two main tasks: structural annotation, which defines the location and structure of genes (e.g., exons, introns, start/stop codons), and functional annotation, which assigns biological meaning by linking gene products to functions, pathways, and domains. As sequencing becomes more accessible, high-quality genome annotation is essential for advancing research in medicine, agriculture, biodiversity, and evolutionary biology.

# Prerequisite

To ease your life do

<!--
```bash
mkdir -p ~/bin/
cat << 'EOF' > ~/bin/singu
#!/bin/bash
# Usage: sing <container_name> <command> [args...]
singularity $1 {{< var container_path >}}/$2 "${@:3}"
EOF
chmod +x ~/bin/singu
export PATH=$PATH:~/bin
```

You can now execute commande like that
```bash
sing exec repeatmasker_4.2.1--pl5321hdfd78af_0.sif <commande>
````

instead of 
```bash
singularity exec path/to/repeatmasker_4.2.1--pl5321hdfd78af_0.sif <commande>
````
-->
```bash
DATA={{< var data_path >}}
CONTAINER={{< var container_path >}}
```
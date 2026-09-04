[![CC BY SA 4.0][cc-by-sa-shield]][cc-by-sa]


# bioinfo_training_infectious_disease
---------------------------

Tutorial pages for the training: Bioinformatics for Infectious Disease Ecology Research - based on Galaxy

<img src="docs/pages/images/IRD.png" width="300" height="100" style="height: 100px" />

## Table of Contents

   * [Foreword](#foreword)
   * [Project layout](#project-layout)
   * [For collaborators-teachers and developers](#for-collaborators-teachers-and-developers)     
     * [Modify content](#modify-content)   
     * [Quarto](#quarto)
        * [Welcome to Quarto](#welcome-to-quarto)
        * [Installation](#installation)
          * [Manual](#manual)
        * [Testing and building the website](#testing-and-building-the-website)
   * [Acknowledgement](#acknowledgement)
   * [License](#license)


## Foreword

The University of Science and Technology of Hanoi (USTH) and the French National Research Institute for Sustainable Development (Institut de Recherche pour le Développement – IRD) in Montpellier, France, have established extensive research and training collaborations in the fields of microbiology, infectious disease ecology, and life sciences.
Within the framework of the cooperation between USTH and IRD, and with the aim of supporting training and enhancing research capacity among students, researchers, and early-career lecturers in Southeast Asia, USTH, in collaboration with IRD, is organizing the specialized course “Bioinformatics for Infectious Disease Ecology Research”

The course itself lives [https://jhayer.github.io/bioinfo_training_infectious_disease](https://jhayer.github.io/bioinfo_training_infectious_disease),
where you can find all the relevant information.  

## Project layout

```
    README.md               # General readme 
    conda_env.yml           # Conda env to build and test the site locally
    docs/                   # material that will be publish with the static web site
        _extensions/...     # Quarto extensions used for the site rendering
        _quarto.yml         # The configuration file for the site rendering.
        _variables_.yml     # The variables for the site rendering
        index.md            # The documentation homepage (Website Home page).
        pages/              # Folder dedicated to the course materials use by mkdocs for the website
            images          # Images used in the course materials in general
            xxx/            # Folder containning the course materials for one topic
                xxx.md      # Page taking about all or a part of the topic
                images/     # Images related to the topic
            ...    
        lectures/           # Folder dedicated to lectures (within docs to ease embedding within the course)
            README.md       # readme
            xxx.pdf         # PDF of a lecture
    LICENSE                 # License of the course materials
    .gitignore              # gitignore file
    .gitlab-ci.yml          # Gitlab CI configuration file to automatically build and deploy the website on gitlab pages
    .github/workflows/pages.yml  # GitHub CI configuration file to automatically build and deploy the website on github pages
```

## For collaborators-teachers and developers

This part is for collaborators-teachers and developers.

### Modify content

When working in the repository, add and/or modify your course materials in the docs/ directory.

The structure of the website (navigation, sections, pages) is defined in the _quarto.yml file.

Changes pushed to the repository will trigger the GitHub Actions workflow (`.github/workflows/pages.yml`), which builds the website and deploys it when GitHub Pages is configured for this repository. The updated version should be available online a few minutes after the workflow completes.

> [!NOTE]
> GitHub Pages must be enabled once in **Settings → Pages** with **Source** set to **GitHub Actions** before automated deployments can run.

To see modifications locally before to push them, you can use the `quarto preview` command to start a local server and see the result of your modification.

> [!TIP]
> Keep the folder structure simple and intuitive to make navigation and maintenance easier.


##### Project variables information

`_variables.yml`

This file contains the variables of the website.


```yaml
site_name: Course Name # Displayed at the very top of the page and in the title of the browser tab
repo_name: 'top_right_corner' # Displayed in the top right corner of the page, can be set to false to hide it
repo_url:  # URL of the repository, used to generate the "Edit on GitHub" links
```

**optional** 
  You can define variable here and use then in your markdown files with `{{< var variable_name >}}` syntax.

Example of variable :
```yaml
training_path: "~/genome_annotation"
data_path: "/shared/projects/tp_2539_ird_genome_annotation_181503/GA_ITROP/DATA/"
```

##### Project configuration

`_quarto.yml`

This file contains the main configuration of the website.

It defines:
 * the website title
 * the navigation structure
 * global options and metadata

```yaml
#Example ...
website:
  title: "Course Name"
  description: "Description"
  page-navigation: false

  #navigation structure **menu**
  navbar:
    background: "#01A9F5"
    foreground: "white"

    left:
      - text: Home
        href: docs/index.md
#...
```

#### data

Where to have data if needed for the course. Please avoid large data files in the repository, you can use git lfs if needed.
You can also provide a link to an external data repository if you do not want to provide the data directly in the repository.

#### docs

Where to put the course materials. The content of this folder will be used to generate the static website.
The arborescence of the website menu is to setup in the `_quarto.yml`.

##### lectures

Where to put lectures. You can display the lecture in a specific page of the website via an iframe.
e.g. <iframe id="iframepdf" src="../../lectures/lecture.pdf" frameborder="0" width="640" height="480" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

> [!NOTE]
> You just have to adapt the path (src="") to point to the right file.

##### pages

Where to put the course materials (.md files and images, etc). The arborescence of the website menu is to setup in the `_quarto.yml`. 

It is diveded in subfolders to ease the organization of the course materials. You can add as many subfolders as you want, but try to keep it simple and intuitive.
By default we have:
 - course-information: where to put general information about the course (practical information, schedule, teachers, etc)
 - images: where to put images used in the course materials
 - cheat_sheet: where to put cheat sheets if needed
 - course : where to all the course materials (e.g. tutorial, exercises, etc). You can add

#### index.md / Home page

This is the homepage of the website, it contains a general introduction to the course, the objectives, the target audience, etc.
This page is made from an aggregation of different .md files from the `doc/pages/course-information` folder, so you just have to adapt the content of these files to modify the content of the homepage. !! No need to touch the index.md file itself.

### Quarto

   <details>
      <summary>See details</summary>

#### Welcome to Quarto

For full documentation visit [quarto.org](https://quarto.org).  

#### Installation

As prerequisite you need python >=3.8 and pip.  
First, create a isolated python environment. 

##### via CONDA
```bash
conda env create -f conda_env.yml
conda activate quarto
```

#### Testing and building the website
* `quarto add mcanouil/quarto-external@1.6.0` # add extension for import external content
* `quarto check` - Check the project for errors and warnings.
* `quarto render docs` - Render the project to create the static website in a folder named `public` (see `_quarto.yml` for configuration).
* `quarto preview docs` - Start a local server to preview the website at `http://localhost:XXXX`. This command also watches for changes in the source files and automatically re-renders the site when changes are detected.
   </details>

##  Acknowledgement

We acknowledge [iTrop](https://itrop.pages.ird.fr/website/ ) for the Quarto framework used to deploy this course.

[cc-by-sa]: https://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY%20SA-blue.svg

# Title1

Please visit this website to catch all information about making nice MKDOCS files:  
=> https://www.mkdocs.org
=> https://squidfunk.github.io/mkdocs-material/


!!! info

    **Time:** 45 min

    **Objectives:** Be able to run Augustus an ab initio annotation tool

    **Questions:**

    - Question1?
    - Question1?  

## Title2

Blablabla

!!! question
    - Question1?
    - Question1?    

??? example "Click to show the solution"
        The solution is to force docker to use an X86/amd64 platform miniforge image as base image:  

### Title3

Here an example of a code block:

```bash
# get the tool
docker pull quay.io/biocontainers/augustus:3.5.0--pl5321h91531cf_7
# call the help
docker run --rm quay.io/biocontainers/augustus:3.5.0--pl5321h91531cf_7 augustus --help
```

### Title3

Here an example of a table:

| Column1 | Column2 | Column3 |
|---------|---------|---------|
| Value1  | Value2  | Value3  |
| Value1  | Value2  | Value3  |
| Value1  | Value2  | Value3  |
| ValueA  | ValueB  | ValueC  |


!!! Success "Quick recap"
    Here a quick recap:

    -  we have learn ....
    - we have learn ....
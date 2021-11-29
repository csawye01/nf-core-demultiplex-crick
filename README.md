# nf-core/demultiplex
**Demultiplexing pipeline for Illumina data**
**IN PROGRESS**

[![Build Status](https://travis-ci.org/nf-core/demultiplex.svg?branch=master)](https://travis-ci.org/nf-core/demultiplex)
[![Nextflow](https://img.shields.io/badge/nextflow-%E2%89%A50.32.0-brightgreen.svg)](https://www.nextflow.io/)

[![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg)](http://bioconda.github.io/)
[![Docker](https://img.shields.io/docker/automated/nfcore/demultiplex.svg)](https://hub.docker.com/r/nfcore/demultiplex)
![Singularity Container available](
https://img.shields.io/badge/singularity-available-7E4C74.svg)

### Introduction
**nf-core/demultiplex** is a bioinformatics demultiplexing pipeline used for multiple types of data input from sequencing runs.
The pipeline is built using [Nextflow](https://www.nextflow.io), a workflow tool to run tasks across multiple compute infrastructures in a very portable manner. It comes with docker / singularity containers making installation trivial and results highly reproducible.

### Sample Sheet Format
The sample sheet must fall into the same format as seen below to adhere to the Illumina standards with the additional column of DataAnalysisType and ReferenceGenome to ensure 10X sample will be processed correctly. Order of columns does not matter but the case of column names does.

| Lane        | Sample_ID   | User_Sample_Name | index   | index2 | Sample_Project | ReferenceGenome | DataAnalysisType |
|-------------|-------------|------------------|---------|--------|----------------|-----------------|------------------|
|     1       |   ABC11A2   | U_ABC0_BS_GL_DNA |  CGATGT |        |     PM10000    |  Homo sapiens   |    Whole Exome   |
|     2       |  SAG100A10  |     SAG100A10    | SI-GA-C1|        |     SC18100    |  Mus musculus	 |    10X-3prime    |
|     3       |  CAP200A11  |    UN1800_AE_6   |  iCLIP  |        |     PM18200    |  Homo sapiens   |       Other      |


### Pipeline summary
1. Reformatting the input sample sheet
    * Script looks for `iCLIP` in the index column of the sample sheet and collapses the iCLIP samples into one per lane.
    * Splits 10X single cell samples into 10X, 10X-ATAC and 10X-DNA .csv files by searching in the sample sheet column DataAnalysisType for `10X-3prime`, `10X-ATAC` and `10X-CNV`.
    * Outputs the results of needing to run specific processes in the pipeline (can be only 10X single cell samples, mix of 10X single cell with non single cell samples or all non single cell samples)
2. Checking the sample sheet for downstream error causing samples such as:
    * a mix of short and long indexes on the same lane
    * a mix of single and dual indexes on the same lane
3. Processes that only run if there are issues within the sample sheet found by the sample sheet check process (CONDITIONAL):
      1. Creates a new sample sheet with any samples that would cause an error removed and create a a txt file of a list of the removed problem samples
      2. Run [`bcl2fastq`](http://emea.support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) on the newly created sample sheet and output the Stats.json file
      3. Parsing the Stats.json file for the indexes that were in the problem samples list.
      4. Recheck newly made sample sheet for any errors or problem samples that did not match any indexes in the Stats.json file. If there is still an issue the pipeline will exit at this stage.
4. Single cell 10X sample processes (CONDITIONAL):
      Will run either CellRanger, CellRangerATAC, CellRangerDNA depending on the samplesheet data type
      NOTE: Must create CONFIG to point to CellRanger genome References
      1. Cell Ranger mkfastq runs only when 10X samples exist. This will run the process with [`CellRanger`](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger), [`CellRanger ATAC`](https://support.10xgenomics.com/single-cell-atac/software/pipelines/latest/what-is-cell-ranger-atac), and [`Cell Ranger DNA`](https://support.10xgenomics.com/single-cell-dna/software/pipelines/latest/what-is-cell-ranger-dna) depending on which sample sheet has been created.
      2. Cell Ranger Count runs only when 10X samples exist. This will run the process with [`Cell Ranger Count`](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/count), [`Cell Ranger ATAC Count`](https://support.10xgenomics.com/single-cell-atac/software/pipelines/latest/using/count), and [`Cell Ranger DNA CNV`](https://support.10xgenomics.com/single-cell-dna/software/pipelines/latest/using/cnv)depending on the output from Cell Ranger mkfastq. 10X reference genomes can be downloaded from the 10X site, a new config would have to be created to point to the location of these. Must add config to point Cell Ranger to genome references if used outside the Crick profile.
5. [`bcl2fastq`](http://emea.support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) (CONDITIONAL):
      1. Runs on either the original sample sheet that had no error prone samples or on the newly created sample sheet created from the extra steps. 
      2. This is only run when there are samples left on the sample sheet after removing the single cell samples. 
      3. The arguments passed in bcl2fastq are changeable parameters that can be set on the command line when initiating the pipeline. Takes into account if Index reads will be made into FastQ's as well
6. [`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) runs on the pooled fastq files from all the conditional processes.
7. [`FastQ Screen`](https://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/) runs on the pooled results from all the conditional processes.
8. [`MultiQC`](https://multiqc.info/docs/) runs on each projects FastQC results produced.
9. [`MultiQC_all`](https://multiqc.info/docs/) runs on all FastQC results produced.


### Documentation
The nf-core/demultiplex pipeline comes with documentation about the pipeline, found in the `docs/` directory:

1. [Installation](docs/installation.md)
2. Pipeline configuration
    * [Local installation](docs/configuration/local.md)
    * [Adding your own system](docs/configuration/adding_your_own.md)
    * [Reference genomes](docs/configuration/reference_genomes.md)  
3. [Running the pipeline](docs/usage.md)
4. [Output and how to interpret the results](docs/output.md)
5. [Troubleshooting](docs/troubleshooting.md)

### Credits
Credits
The nf-core/demultiplex pipeline was written by Chelsea Sawyer of the The Bioinformatics & Biostatistics Group for use at The Francis Crick Institute, London.
Many thanks to others who have helped out along the way too, including (but not limited to): [`@ChristopherBarrington`](https://github.com/ChristopherBarrington), [`@drpatelh`](https://github.com/drpatelh), [`@danielecook`](https://github.com/danielecook), [`@escudem`](https://github.com/escudem), [`@crickbabs`](https://github.com/crickbabs)

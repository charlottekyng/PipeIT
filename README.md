# PipeIT
Stand-alone Singularity Container for somatic variant calling on the Ion Torrent platform.

### Introduction
We present PipeIT, an accurate variant call bioinformatics pipeline specific for Ion Torrent sequencing data. The pipeline has been enclosed into a Singularity image to allow easy and high throughput analyses.

### Software requirements
PipeIT only needs a working Singularity installation. Please visit the [official website](https://singularity.lbl.gov/) for more information.

### Installation
PipeIT can be downloaded from our laboratory's website: http://oncogenomicslab.org/software-downloads/. 
No further installations are required, all the dependencies needed to perform the whole analysis are already installed within the container.

### Running the pipeline
PipeIT can be executed by simply running this command: 
```
singularity run PipeIT.img -t path/to/tumor.bam -n path/to/normal.bam -e path/to/region.bed [-u path/to/unmerged.bed]
```
The only mandatory input files can be directly obtained from the Ion Torrent, the tumor and the normal BAM files and the BED files from the sequenced region. The user can specify his own unmerged BED. Wherever this file is not manually specified PipeIT will simply build the unmerged BED on its own.

While specifying input files please note that due to Singularity's nature:
- Paths to input files have to be *Relative*
> ... relative paths will resolve outside the container, and fully qualified paths will resolve inside the container.

Please read [Singularity's FAQ page](http://singularity.lbl.gov/archive/docs/v2-2/faq) for more information about this.
- Singularity automatically mounts some folders inside the container:
> ... Some of the bind paths are automatically derived (e.g. a user’s home directory) and some are statically defined (e.g. bind path in the Singularity configuration file). In the default configuration, the directories $HOME, /tmp, /proc, /sys, and /dev are among the system-defined bind points. 

Input files should be inside these folders in order to make them accessible to Singularity, otherwise PipeIT won't be able to see and use them.
User can also manually mount additional files and folders using the -B flag. For example:
```
singularity run -B /myHPC/home/username/BAMfiles/ PipeIT.img -t ABSOLUTEpath/to/tumor.bam -n ABSOLUTEpath/to/normal.bam -e RELATIVEpath/to/region.bed
```

Unexperienced users should read [Singularity's official documentation](http://singularity.lbl.gov/docs-mount) to better know which folders and files are automatically mounted within the container and how to mount external ones.

### Output files
PipeIT will locally create a folder that will include both the final output, a VCF file, and the all the files produced by the intermediate steps of the pipeline. The latters will be automatically deleted by PipeIT at the end of its execution.

Please note that an empty final VCF file means that PipeIT have found no significant mutation in the input sample.


### Workflow
- Step1: Torrent Variant Caller is performed with lenient parameters on the input submitted bam files.
- Step2: Multiallelic variants are split, left aligned, trimmed and merged once again with the Biallelic variants using BCFtools and GATK.
- Step3: Using GATK once again, all the variants are filtered by removing regions with less than 10 reads, mutations with less than 8 reads and with a tumor-normal allele frequency lower than 10:1.

***

## Docker version
A Docker image has also been built for Docker users and can be found on the Docker Hub page: https://hub.docker.com/r/ckynlab/pipeit/. We suggest to use the Singularity version of PipeIT because the pipeline was defined to work on HPC environments, execution times on local machines will not be optimal.

**Please note that the Docker image may not be up-to-date, for a proper execution please use the Singularity image or contact us**

### Running the pipeline
Just like for the Singularity image, the Docker version of PipeIT can be launched using a simple command but, due to Docker's behaviour with files external to the container itself, the user needs to mount the folder containing the input file within the container itself.
One easy option could be to create a folder called "data", use it to store all the input files and launch the command: 
```
docker run  --mount type=bind,source="$(pwd)"/data,target=/PipeIT/data,consistency=consistent -it pipeit:latest -t nameoftumor.bam -n nameofnormal.bam -e nameofregion.bed [-u nameofunmerged.bed]
```
Please notice that if you are using the "data" folder you must only use the name of the files, not the path.
PipeIT will create the output files within this directory.

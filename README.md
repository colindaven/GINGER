# GINGER: Gradual INtegrated GEne Reconstruction

## Description #################################################################

`GINGER` is a tool that is implemented an integrated method for gene
structure prediction in higher eukaryotes. RNA-Seq-based methods,
ab initio-based methods, homology-based methods are performed, and then 
gene structures are reconstructed via dynamic programming with appropriately
weighted and scored exon/intron/intergenic regions. Different prediction
processes and filtering criteria are applied to multi-exon and single-exon
genes. This pipeline is implemented using `Nextflow`.

## Web site ####################################################################

<https://github.com/i10labtitech/GINGER>
<https://anaconda.org/i10labtitech/ginger>

## Requirements ################################################################

See `Requirements` section in `INSTALL`.

## Installation ################################################################

See `Installation` section in `INSTALL`.

## Synopsis ####################################################################
### Settings ###################################################################

Set `nextflow.config` properly.
See `Settings` section in `INSTALL`

### Inputs #####################################################################

* A file containing genome sequences (`multi FASTA format`)
* A file containing masked genome sequences by RpeatMasker (`multi FASTA format`)
* A file containing repeat information by RpeatMasker (`***.out`)
* A file containing RNA-Seq 1 (`FASTQ format`)
* A file containing RNA-Seq 2 (`FASTQ format`)
* Files containing closely related species

### Commands A (using the Docker image) #########################################

The command to kick the docker image with the prepared Perl script is as follows.
(Docker image: `i10labtitech/tools:GINGER_v1.0.1`)

```
generateSampleData_cel [Directory name for sample input data (e.g. sample_data_cel)]
# Copy and edit nextflow.config
runGINGER.pl nextflow.config.user
```

* Edit the [The path to the directory that contains `sample_data_cel/`] part of 
  the variables named `INPUT_GENOME, INPUT_GENOME, INPUT_MASKEDGENOME, INPUT_REPOUT`.
  `INPUT_RNASEQR1, INPUT_RNASEQR2, PROTEIN`
* Edit the [The path to the output directory (e.g. `/XXX/XXX/sample_data_cel/`] 
  part of the variable named PDIR.
* Edit the variable (path to scrach directory) named `SCRATCH`.
* Edit the variable (number of threads used) named `N_THREAD`.
* Edit the variable (maximum memory size used) named `MAX_MEMORY`.
* Edit the variables named `RNASEQ_OTHER#, HOMOLOGY_OTHER#, ABINITIO_OTHER#, (paths)`
  `RNASEQ_OTHER#_WEIGHT,  HOMOLOGY_OTHER#_WEIGHT, ABINITIO_OTHER#_WEIGHT, (weights)`
  when you want to use your annotation data to predict gene structures in GINGER.
* Note that `#` is the number of annotation data.

```
cd /XXX/XXX/output_cel/
runEvaluatePred.pl /XXX/XXX/sample_data_cel/GCF_000002985.6_WBcel235_genomic.gff ./ginger_phase2.gff RNA CDS RNA CDS
```

### Commands B (using installed GINGER) ##########################################

The command to run GINGER after installing it is as follows.
Put nextflow.config into your working directory.
If you want to run the preparation phase all at once, type like following.
phase2.sh requires an argument specifying minimum CDS length. 50 is an example.

If you have installed the conda package using "conda" or "mamba" command,
you can obatain the automatically configured nextflow.config file for tool paths
in the current directory using the "gingerInitCfg" command.

```(If you have installed the conda package using "conda" or "mamba" command)
mamba install -c i10labtitech ginger
gingerInitCfg
# Then, a nextflow.config file will be generated in the current directory.
# Edit the paths of input files, the output directory, the scratch directory path,
# the number of threads used, and the maximum memory capacity, among other settings,
# in the nextflow.config file.
```

```
nextflow -C nextflow.config run /path/to/pipeline/ginger.nf
/path/to/pipeline/phase0.sh nextflow.config
/path/to/pipeline/phase1.sh nextflow.config > phase1.log
/path/to/pipeline/phase2.sh 50
/path/to/pipeline/summary.sh nextflow.config
```
* [Note] `/path/to/pipeline/` a path to a directory <"pipeline/" in GINGER's source tree>
* [Note] An automatically calculated threshold for the score is written 
       like `score threshold = 1.25` in `phase1.log`.

If you want to run each methods separetely in the preparation phase, type like
following. `abinitio.nf` must be executed after `mapping.nf`. The order in which
`denovo.nf` and `homology.nf` are executed do not depend on them.

```
nextflow -C nextflow.config run /path/to/pipeline/mapping.nf
nextflow -C nextflow.config run /path/to/pipeline/denovo.nf
nextflow -C nextflow.config run /path/to/pipeline/abinitio.nf
nextflow -C nextflow.config run /path/to/pipeline/homology.nf
/path/to/pipeline/phase0.sh nextflow.config
/path/to/pipeline/phase1.sh nextflow.config > phase1.log
/path/to/pipeline/phase2.sh 50
/path/to/pipeline/summary.sh nextflow.config
```

To set a threshold for reconstructed gene structure's scores in the `merge` phase,
type like following (use `phase1_manual.sh` instead of `phase1.sh`). The first
argument is the threshold. 1.0 is an example.

```
/path/to/pipeline/phase1_manual.sh nextflow.config 1.0
```

### Final output ###############################################################

The final outputs:
* `ginger_phase2.gff` : gene structures by GINGER (GFF3) 
  [Note] See http://gmod.org/wiki/GFF3 for details.
* `ginger.pep`        : protein sequences of the gene structurs (FASTA)
* `ginger.cds`        : CDS of the gene structures (FASTA)
* `ginger_stats.tsv`  : statistical information of gene structures

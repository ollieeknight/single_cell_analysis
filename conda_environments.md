# Setting up conda environments for processing genomics files
Once you've logged into the BIH-CUBI cluster through the command line, setting up your conda environments is crucial for processing genetic data.
```shell
mkdir ~/work/bin/ && cd ~/work/bin/

# download, install, and update miniconda 
curl -L https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/work/bin/miniconda3 && rm Miniconda3-latest-Linux-x86_64.sh
source miniconda3/etc/profile.d/conda.sh && conda activate

# modify conda repositories  
nano ~/.condarc

# copy and paste this into nano (CTRL+C here, right click to paste)
channels:
  - conda-forge
  - bioconda
  - defaults
show_channel_urls: true
changeps1: true
channel_priority: flexible
# close by CTRL+X and y and enter

conda upgrade --all -y
conda config --set solver libmamba

```

## Processing raw BCL files
```shell
conda create -y -n bcl_to_fastq -c bih-cubi bcl2fastq2
```

## Genotyping genomic BAM files
```shell
conda create -y -n donor_genotyping python cellsnp-lite  
conda activate donor_genotyping
pip install vireoSNP
```

## Doublet detection from scATAC fragment overlap
```shell
conda create -y -n amulet_overlap numpy=1.19 pandas scipy statsmodels
```

## Genotyping scATAC mitochondrial DNA
```shell
conda create -y -n mitochondrial_genotyping openjdk r-base=4.2.3 r-data.table r-matrix bioconductor-genomicranges bioconductor-summarizedexperiment ruamel.yaml==0.17.35 snakemake==7.32.4 pyyaml==6.0.1 yaml=0.2.5  
conda activate mitochondrial_genotyping
pip install mgatk
```

## Collating antibody capture counts per cell
```shell
conda create -y -n adt_count python kallisto bustools 
conda activate adt_count
pip install bio
```

## Cellbender
```shell
conda create -y -n cellbender -c nvidia python=3.7 cuda-toolkit cuda-nvcc
conda activate cellbender
pip install cellbender
```

## Manipulating genomic files

```shell
conda create -y -n genome_processing bcftools samtools bedtools bwa
```

## Using python packages in `R` through `reticulate`

```shell
conda create -y -n r-reticulate -c vtraag python-igraph pandas umap-learn scanpy macs2 scvi-tools
conda activate r-reticulate
conda install -c vtraag leidenalg
```

Then, in R, start your script with
```R
Sys.setenv(RETICULATE_MINICONDA_PATH = '~/work/bin/miniconda3/')
Sys.setenv(PATH = paste('~/work/bin/miniconda3/envs/r-reticulate/lib/python3.10/site-packages/', Sys.getenv()['PATH'], sep = ':'))
library(reticulate)
use_miniconda('~/work/bin/miniconda3/envs/r-reticulate/')
```

And for MACS2 peaks calling:
```R
peaks <- Signac::CallPeaks(alldata, assay = 'ATAC', macs2.path = '~/bin/miniconda3/envs/r-reticulate/bin/macs2')
```
Where `alldata` is your seurat object.
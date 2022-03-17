[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

# Contigs scaffolding with Hi-C for plant genomes
This repository is made for eBook entitled “Bioinformatics Recipes for Plant Genomics: Data, Code, and Workflows” in [Bio-101](https://cn.bio-protocol.org/bio101/default.aspx)

Hi-C is a chromosome conformation capture method that was originally developed to detect genome-wide chromatin interactions. Nowadays, it is widely applied in scaffolding de novo assembled contigs into chromosome-scale genome sequences. Multiple open-source software has been developed to perform genome scaffolding with Hi-C data. The input data is de novo assembled contigs using long-read sequencing or next-generation sequencing. Then, Hi-C data is mapped to these contigs and the interact matrix is computed by software to scaffold contigs into chromosome-scale sequences. Different tools have their specific algorithm to calculate the interaction matrix, correct misassembly and misjoins, and also may require different dependent packages or running environments. Here, we describe a step-by-step protocol for genome scaffolding using Hi-C data with a comprehensive pipeline: compute interact matrix with Juicer, scaffold contigs with 3D-DNA pipeline, then visualize and modify scaffolding with Juicebox. This pipeline only requires primarily assembled contigs and Hi-C data as inputs, is compatible with multiple enzymes, and also provides visualization and manual correction. With these advantages, this pipeline has been used in mass big eukaryotic genome scaffolding.

## Software dependencies
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)  
[Juicer](https://github.com/aidenlab/juicer/)   
[3D-DNA pipeline](https://github.com/aidenlab/3d-dna)   
[Juicebox](https://github.com/aidenlab/Juicebox)   
[BWA](http://bio-bwa.sourceforge.net/)   
[BUSCO](https://gitlab.com/ezlab/busco)   
[Java 1.8 JDK](https://www.oracle.com/java/technologies/downloads/#java8)   

## Input data
1. _De novo_ assembly contigs file in FASTA format   
2. Raw Hi-C sequencing data in FASTQ format

## Procedure
More details can be found in the paper

### 1. Equipment required:
a. Linux server or cluster with SLURM or SLF scheduler   
b. PC with at least 16GB RAM for handling big genomes (>1GB)
### 2. Install and configure Juicer
```
mkdir hic; cd hic

git clone https://github.com/theaidenlab/juicer.git

ln -s juicer/SLURM/scripts/ scripts

cd scripts; wget https://hicfiles.tc4ga.com/public/juicer/juicer_tools.1.9.9_jcuda.0.8.jar
ln -s juicer_tools.1.9.9_jcuda.0.8.jar juicer_tools.jar; cd ../

mkdir references
mkdir restriction_sites
```
### 3. Prepare input data for Juicer
a. Copy your_contigs.fasta file (or make soft link) into reference path, and index it with bwa index 
```
ln -s your_path/your_contigs.fasta ./references
cd ./references; bwa index your_contig.fasta; cd ..
```
b. Prepare enzyme site file for your_contigs.fasta
```
cd restriction_sites
wget https://raw.githubusercontent.com/aidenlab/juicer/main/misc/generate_site_positions.py
```
Then, Use vi or vim to edit generate_site_positions.py, insert the following line in line 25:
```
'your_contigs': '../references/your_contigs.fasta',
```
After that:
```
python generate_site_positions.py DpnII your_contigs
awk 'BEGIN{OFS="\t"}{print $1, $NF}' your_contigs_DpnII.txt > your_contigs.chrom.sizes
cd ..
```
c. Filter and clean raw Hi-C sequencing data
```
wget https://github.com/usadellab/Trimmomatic/files/5854859/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip
java -jar ./Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 12 -phred33 -trimlog trimmomatic.log your_hic_R1.fastq.gz your_hic_R2.fastq.gz your_hic_pair_R1.fastq.gz your_hic_unpair_R1.fastq.gz your_hic_pair_R2.fastq.gz your_hic_unpair_R2.fastq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

![](graphs/juicebox_1.jpg)


## License
It is a free and open source software, licensed under [GPLv3](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/gpl-3.0.txt)

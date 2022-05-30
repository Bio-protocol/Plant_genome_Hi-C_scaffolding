[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

# Contigs scaffolding with Hi-C for plant genomes
This repository is made for eBook entitled “Bioinformatics Recipes for Plant Genomics: Data, Code, and Workflows” in [Bio-101](https://cn.bio-protocol.org/bio101/default.aspx)

Hi-C is a chromosome conformation capture method that was originally developed to detect genome-wide chromatin interactions. Nowadays, it is widely applied in scaffolding de novo assembled contigs into chromosome-scale genome sequences. Multiple open-source software has been developed to perform genome scaffolding with Hi-C data. The input data is de novo assembled contigs using long-read sequencing or short-read sequencing. Then, Hi-C data is mapped to these contigs and the interact matrix is computed by software to scaffold contigs into chromosome-scale sequences. Different tools have their specific algorithm to calculate the interact matrix, correct misassemblies and misjoins, and may require different dependent packages or running environments. Here, we describe a step-by-step protocol for genome scaffolding using Hi-C data with a comprehensive pipeline: compute interact matrix with Juicer, scaffold contigs with 3D-DNA pipeline, then visualize and modify scaffolding with Juicebox. It is the first detailed protocol that shows how to scaffold using Hi-C data with this pipeline in plants. Compare to many other pipelines, this protocol only requires primarily assembled contigs and raw Hi-C data as inputs. Besides that, it is also compatible with multiple enzymes, and provides visualization and manual correction. Currently, more and more genomes are sequenced combining Hi-C, this step-by-step protocol will be applied widely in mass big eukaryotic genome scaffolding.

## Software dependencies
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)  
[Juicer](https://github.com/aidenlab/juicer/)   
[3D-DNA pipeline](https://github.com/aidenlab/3d-dna)   
[Juicebox (v. 1.11.08)](https://github.com/aidenlab/Juicebox)   
[BWA](http://bio-bwa.sourceforge.net/)   
[Samtools](http://www.htslib.org/)   
[Miniconda](https://docs.conda.io/en/latest/miniconda.html)   
[BUSCO](https://gitlab.com/ezlab/busco)   
[Java 1.8 JDK](https://www.oracle.com/java/technologies/downloads/#java8)   
__Tips:__ we recommend users use the latest version of each software listed above, except for Juicebox (v. 1.11.08).   

## Input data
1. _De novo_ assembly contigs file in FASTA format, contigs used in this study can be downloaded [HERE](https://zenodo.org/record/6383215#.YjzdkufMKCp)   
2. Raw Hi-C sequencing data in FASTQ format, fastq files used in this study can be downloaed [HERE](https://www.ncbi.nlm.nih.gov/sra/?term=SRR18324160)  
More details can be found in the Input folder  
## Procedure

### 1. Equipment required:
a. Linux server or cluster   
b. PC with at least 16GB RAM for handling big genomes (>1GB)
### 2. Install and configure Juicer
Juicer is the software that maps Hi-C paired-end reads to assembled contigs, and generates the Hi-C interact matrix for downstream analysis.
```
mkdir hic; cd hic

git clone https://github.com/theaidenlab/juicer.git

ln -s juicer/SLURM/scripts/ scripts

cd scripts; wget https://hicfiles.tc4ga.com/public/juicer/juicer_tools.1.9.9_jcuda.0.8.jar
ln -s juicer_tools.1.9.9_jcuda.0.8.jar juicer_tools.jar; cd ../

mkdir references
mkdir restriction_sites
```
Make sure samtools and bwa are in your $PATH
```
export PATH=your_samtools/samtools:$PATH
export PATH=your_bwa/bwa:$PATH
```
__Tips:__ Juicer can be run on AWS, LSF, Univa Grid Engine (UGER), SLURM, and a single CPU, users may need to change the command line “ln -s juicer/SLURM/scripts/ scripts” in b. Configure Juicer section to fit their system. For example, use “ln -s juicer/AWS/scripts/ scripts” for AWS scheduler. For macOS users, "curl https://hicfiles.tc4ga.com/public/juicer/juicer_tools.1.9.9_jcuda.0.8.jar --output ./" can be used instead of _wget_. The same thing can also be applied to all the wget in this protocol.   

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
python generate_site_positions.py your_enzyme your_contigs
awk 'BEGIN{OFS="\t"}{print $1, $NF}' your_contigs_your_enzyme.txt > your_contigs.chrom.sizes
cd ..
```
c. Filter and clean raw Hi-C sequencing data
```
wget https://github.com/usadellab/Trimmomatic/files/5854859/Trimmomatic-0.39.zip
unzip Trimmomatic-0.39.zip

java -jar ./Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads your_threads -phred33 -trimlog trimmomatic.log your_hic_R1.fastq.gz your_hic_R2.fastq.gz your_hic_pair_R1.fastq.gz your_hic_unpair_R1.fastq.gz your_hic_pair_R2.fastq.gz your_hic_unpair_R2.fastq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```
__Tips:__ _your_contigs_, _your_enzyme_, and _your_threads_ are variable, users need to name their own contig file, choose the specific enzyme they used, and specify how many threads they would like to use. _your_hic_R1.fastq.gz_, _your_hic_R2.fastq.gz_ are the sequenced raw Hi-C paired-end data    
### 4. Run Juicer to obtain the interaction matrix
```
mkdir your_contigs_hic; cd your_contigs_hic
mkdir fastq
ln -s ../your_hic_pair* ./fastq/

sh ../scripts/juicer.sh -D $PWD/hic -g your_contigs -s your_enzyme -p ../restriction_sites/your_contigs.chrom.sizes -y ../restriction_sites/your_contigs_your_enzyme.txt -z ../references/your_contig.fasta -Q 2-00:00 -L 7-00:00 -q your_queue_name -l your_long_queue_name -t your_threads -A your_account --assembly

cd ..
```
__Tips:__ Check _juicer.sh_, make sure all the Partition, Account, QOS and Threads fit your cluster’s scheduler. To be safe, add these parameters to your command line. Juicer will submit jobs to the cluster through the scheduler automatically. After all the jobs are done, the file named __merged_nodups.txt__ in _your_contigs_hic/aligned_ will be used by 3D-DNA pipeline.   
### 5. Run 3D-DNA pipeline
3D-DNA pipeline is designed to correct misassembles and scaffold contigs based on the Hi-C interact matrix. It will generate the scaffolds fasta file, and .hic and .assembly files for visualization in Juicebox.   
```
wget https://github.com/aidenlab/3d-dna/archive/refs/tags/201008.tar.gz
tar -zxf 201008.tar.gz
chmod 554 ./3d-dna-201008/*.sh  ##add execute permission

cd your_contigs_hic
../3d-dna-201008/run-asm-pipeline.sh ../references/your_contigs.fasta ./aligned/merged_nodups.txt
```
After the job is done. Two files named __your_contigs.rawchrom.assembly__ and __your_contigs.rawchrom.hic__ will be used by Juicebox   
__Tips:__ If the scaffolding results are not ideal, try different _--round_ different edit round and slightly increase _--editor-repeat-coverage_ misjoin editor threshold repeat coverage. In this case study, we use --editor-repeat-coverage 3.

### 6. Visualize and modify the scaffolding with Juicebox
a. Based on the system, the corresponding Juicebox 1.11.08 version can be downloaded at https://github.com/aidenlab/Juicebox/wiki/Download  
b. Download your_contigs.rawchrom.assembly and your_contigs.rawchrom.hic to your PC  
c. Run Juicebox, then load  your_contigs.rawchrom.hic and your_contigs.rawchrom.assembly in turn (Figure 1)  

![](graphs/juicebox.jpg)
__Figure 1. Steps to load .hic and .assembly file to Juicebox.__ A. Load .hic file; B. Load .assembly file.    
   
d. Correct scaffolding manually (Figure 2 and Figure 3)   
1. Shift+left-click to choose the region that needs to be edited.   
2. Right-click to choose to remove or add chr boundaries.   
3. Move the mouse to the upper-right of the selected region until a circle appears, and then left-click to rotate the selected region   
![](graphs/juicebox_2-1.jpg)
__Figure 2. Examples of how to edit misjoin and misorientation.__ A. edit misjoin via remove and add chr boundaries. B. edit misorientation by rotating selected contigs.  
![](graphs/juicebox_2.jpg)
__Figure 3. Manually correct the scaffolding with Juicebox.__ A. the original scaffolding visilization B. manually corrected scaffolding. Ex. 1: misjoin, Ex. 2: mis-oritention   
   
__Tips:__ Here is a [demo video](https://www.youtube.com/watch?v=Nj7RhQZHM18) show how to use Juicebox from the software developer.
### 7. Run 3D-DNA pipeline again to update the manual modification
a. Upload your_contigs.rawchom.edit.assembly to cluster and put it in your_contigs_hic.
b. Run 3D-DNA pipeline to obtain your edited scaffolds fasta file
```
../3d-dna/run-asm-pipeline-post-review.sh -r your_contigs.FINAL.edit.assembly ../references/your_contigs.fasta aligned/merged_nodups.txt
```
### 8. Primarily estimate the scaffolding with BUSCO
a. Install BUSCO, and download database
```
conda install -c bioconda busco

mkdir busco_evalue; cd busco_evalue
wget https://busco-data.ezlab.org/v4/data/lineages/embryophyta_odb10.2020-09-10.tar.gz
tar -zxf embryophyta_odb10.2020-09-10.tar.gz
```
b. Run BUSCO
```
busco -c your_threads -m genome -i ../your_contigs_hic/your_contigs_HiC.fasta -o your_contigs_hic_busco -l ./embryophyta_odb10
```
c. Check the BUSCO value and components (Figure 4)

![](output/busco.jpg)
__Figure 4. BUSCO summary information.__
## License
It is a free and open source software, licensed under [GPLv3](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/gpl-3.0.txt)

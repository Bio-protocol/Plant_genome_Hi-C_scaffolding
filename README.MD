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




## License
It is a free and open source software, licensed under []() (choose a license from the suggested list:  [GPLv3](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/gpl-3.0.txt), [MIT](https://github.com/github/choosealicense.com/blob/gh-pages/LICENSE.md), or [CC BY 4.0](https://github.com/github/choosealicense.com/blob/gh-pages/_licenses/cc-by-4.0.txt)).

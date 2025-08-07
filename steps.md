# RNA Seq Workflow

In this tutorial we are following this excellent tutorial by [Michael Love et al;](https://www.bioconductor.org/help/course-materials/2019/CSAMA/materials/labs/lab-03-rnaseq/rnaseqGene_CSAMA2019.html?utm_source=perplexity#experimental-data). 

Although the tutorial is great, it skips over some of the key steps that are difficult for beginners, including data acquisition.
Please read up to the following part section 3.2 Salmon quantification:

```bash
salmon index -i gencode.v27_salmon_0.8.2 -t gencode.v27.transcripts.fa.gz
```

Before running the code above, we need to set up the environment and acquire the files that are needed.

## Setting Up

First, we need to download the reference genome that we will align the reads to.

```bash
# donwload the required file
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_27/gencode.v27.transcripts.fa.gz
```

We also need to install salmon which can be done through conda. 

Instructions to install via conda according to their [website](https://combine-lab.github.io/salmon/getting_started/#obtaining-salmon)

```bash
$ conda config --add channels conda-forge
$ conda config --add channels bioconda
$ conda create -n salmon salmon
```
This will install the latest salmon in its own conda environment. The environment can then be activated via:

```bash
$ conda activate salmon
```

Next, we need to download the sequence data.

The data used in this tutorial is GEO entry GSE52778. 
We will [acquire the data from ENA](https://www.ebi.ac.uk/ena/browser/view/SRP033351), rather than GEO as it is more convenient.

```bash
touch ena-download-read-run-SRP033351-fastq-ftp.sh
# then edit the file with nano
nano ena-download-read-run-SRP033351-fastq-ftp.sh
```

Once in the file, copy the text below and paste it into the file:
```bash
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/005/SRR1039515/SRR1039515_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/006/SRR1039516/SRR1039516_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/000/SRR1039520/SRR1039520.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/004/SRR1039514/SRR1039514_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/009/SRR1039519/SRR1039519_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/003/SRR1039513/SRR1039513_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/007/SRR1039517/SRR1039517_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/006/SRR1039516/SRR1039516_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/000/SRR1039520/SRR1039520_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/001/SRR1039521/SRR1039521_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/000/SRR1039510/SRR1039510_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/008/SRR1039518/SRR1039518_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/002/SRR1039522/SRR1039522_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/005/SRR1039515/SRR1039515_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/001/SRR1039511/SRR1039511_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/008/SRR1039508/SRR1039508_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/009/SRR1039509/SRR1039509_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/004/SRR1039514/SRR1039514_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/003/SRR1039513/SRR1039513_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/002/SRR1039512/SRR1039512_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/003/SRR1039523/SRR1039523_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/009/SRR1039519/SRR1039519.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/002/SRR1039522/SRR1039522.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/003/SRR1039513/SRR1039513.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/005/SRR1039515/SRR1039515.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/000/SRR1039510/SRR1039510_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/009/SRR1039519/SRR1039519_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/001/SRR1039521/SRR1039521.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/001/SRR1039511/SRR1039511_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/008/SRR1039508/SRR1039508_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/002/SRR1039512/SRR1039512_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/009/SRR1039509/SRR1039509_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/007/SRR1039517/SRR1039517_1.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/003/SRR1039523/SRR1039523_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/000/SRR1039520/SRR1039520_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/006/SRR1039516/SRR1039516.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/001/SRR1039521/SRR1039521_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/002/SRR1039522/SRR1039522_2.fastq.gz
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR103/008/SRR1039518/SRR1039518_1.fastq.gz
```


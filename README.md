# GlobalSewage Project

## This repository is under construction

This repository contains all data analysis for ***Predicting clinical resistance prevalence using sewage metagenomic data*** (DOI:XXX).

This study is based on the sewage metagenome data from ***Global monitoring of antimicrobial resistance based on metagenomics analyses of urban sewage*** by Hendriksen _et al_. 2019 (https://doi.org/10.1038/s41467-019-08853-3). These metagenomes used in this study can be downloaded from ENA under project accession number ERP015409 or from SRA under project accession PRJEB13831.

The clinical resistance and socioeconomical data sources are described in Table 1 in the article and can be found from the `Data` folder in this repository.

## Pre-processing
Raw sequencing data was downloaded from European Nucleotide Archive (ENA) under the project accession [ERP015409](https://www.ebi.ac.uk/ena/browser/view/PRJEB13831) and remaining sequencing adapters were removed using [cutadapt v.2.7.](https://cutadapt.readthedocs.io/en/v2.7/).  

```
cutadapt -m 1 -e 0.2 -O 10 -g AGATCGGAAGAGC -G AGATCGGAAGAGC -o SAMPLE_R1_trim.fq.gz -p SAMPLE_R2_trim.fq.gz SAMPLE_R1.fq.gz SAMPLE_R2.fq.gz
```

After trimming the adapters the reads were converted from FASTQ to FASTA and all R1 and R2 reads combined. The combined R1 and R2 reads were searched for antibiotic resistance genes and intI1 integrase genes with [DIAMOND v.0.9.114](http://www.diamondsearch.org/index.php).  

```
# R1 reads
diamond blastx -d [ARG/MGE DATABASE] -q GlobalSewage_R1.fasta --max-target-seqs 1 -o GlobalSewage_R1_mge.txt -f 6 --id [ARG:90 / MGE:95] --min-orf 20 -p 24 --seg no

# R2 reads
diamond blastx -d [ARG/MGE DATABASE] -q GlobalSewage_R2.fasta --max-target-seqs 1 -o GlobalSewage_R2_mge.txt -f 6 --id [ARG:90 / MGE:95] --min-orf 20 -p 24 --seg no
```



## Data analyis
After pre-processing the metagenomic data was combined with clinical and socioeconomical data on country level in one data frame in R.
The data analysis steps after this and making of the figures in R are desribed in [here.]( https://karkman.github.io/GlobalSewage/)

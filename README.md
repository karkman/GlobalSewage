# GlobalSewage Project

## This repository is under construction

This repository contains all data analysis for ***Predicting clinical resistance prevalence using sewage metagenomic data*** (Karkman _et al._, 2020).

This study is based on the sewage metagenome data from ***Global monitoring of antimicrobial resistance based on metagenomics analyses of urban sewage*** by Hendriksen _et al_. (2019). These metagenomes used in this study can be downloaded from ENA under project accession number ERP015409 or from SRA under project accession PRJEB13831.

The clinical resistance and socioeconomical data sources are described in Table 2 (Karkman _et al._, 2020) and can be found from the `Data` folder in this repository.

## Pre-processing
Raw sequencing data was downloaded from European Nucleotide Archive (ENA) under the project accession [ERP015409](https://www.ebi.ac.uk/ena/browser/view/PRJEB13831) and remaining sequencing adapters were removed using [cutadapt v.2.7](https://cutadapt.readthedocs.io/en/v2.7/) (Martin, 2011).   

```
cutadapt -m 1 -e 0.2 -O 10 -g AGATCGGAAGAGC -G AGATCGGAAGAGC \
          -o SAMPLE_R1_trim.fq.gz -p SAMPLE_R2_trim.fq.gz \
          SAMPLE_R1.fq.gz SAMPLE_R2.fq.gz
```

After trimming the adapters the reads were converted from FASTQ to FASTA and all R1 and R2 reads combined. The combined R1 and R2 reads were searched for antibiotic resistance genes (ARGs) and _intI1_ integrase genes with [DIAMOND v.0.9.114](http://www.diamondsearch.org/index.php) (Buchfink _et al._ 2015).  
The ARG database was ResFinder (Zankari _et al._, 2012) and the _intI1_ integrase gene was from an MGE database (Pärnänen _et al._, 2019).

The _E. coli_ connected ARGs were annotated in similar fashion (see Karkman _et al._, 2020 for details).

```
# R1 reads
diamond blastx -d [ARG/MGE DATABASE] -q GlobalSewage_R1.fasta \
          --max-target-seqs 1 -o GlobalSewage_R1.txt -f 6 \
          --id [ARG:90 / MGE:95] --min-orf 20 -p 24 --seg no

# R2 reads
diamond blastx -d [ARG/MGE DATABASE] -q GlobalSewage_R2.fasta \
          --max-target-seqs 1 -o GlobalSewage_R2.txt -f 6 \
          --id [ARG:90 / MGE:95] --min-orf 20 -p 24 --seg no
```

After searching R1 and R2 reads for ARGs/_intI_ integrase gene, the results were combined to a gene count table using a [paired end version of the DIAMOND parser v.0.1](https://github.com/karkman/parse_diamond).
The parser counts the occurrence of each gene in each sample. The R2 read is counted only if the corresponding R1 read did not have a hit.  

```
./parse_diamondPE.py -1 GlobalSewage_R1.txt -2 GlobalSewage_R2.txt -o GlobalSewage_gene_count.csv
```

After ARG/_intI1_ integrase gene annotation the results were analysed in R.

## Data analyis
After pre-processing the metagenomic data was imported to R.  Metagenomic data was combined with clinical and socioeconomical data on country level. For countries where there were more than one sewage sample, mean count was used.  

The data analysis steps and making of the figures in R are described in [here.]( https://karkman.github.io/GlobalSewage/)

## References

- Karkman, A., Berglund, F. ...
- Hendriksen https://doi.org/10.1038/s41467-019-08853-3)
- Martin 2011
- Zankari
- Pärnänen
- Buchfink

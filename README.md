# GlobalSewage Project

This repository contains all data analysis for ***Predicting clinical resistance prevalence using sewage metagenomic data*** (Karkman _et al._, 2020).

This study is based on the sewage metagenome data from ***Global monitoring of antimicrobial resistance based on metagenomics analyses of urban sewage*** by Hendriksen _et al_. (2019). The data consists of 234 sewage metagenomemes from 62 countries. These metagenomes used in this study can be downloaded from ENA under project accession number ERP015409 or from SRA under project accession PRJEB13831.

The clinical resistance and socioeconomical data sources are described in Table 2 (Karkman _et al._, 2020) and can be found from the `Data` folder in this repository.

## Pre-processing
Raw sequencing data was downloaded from European Nucleotide Archive (ENA) under the project accession [ERP015409](https://www.ebi.ac.uk/ena/browser/view/PRJEB13831) and remaining sequencing adapters were removed using [cutadapt v.2.7](https://cutadapt.readthedocs.io/en/v2.7/) (Martin, 2011).   

```
cutadapt -m 1 -e 0.2 -O 10 -g AGATCGGAAGAGC -G AGATCGGAAGAGC \
          -o SAMPLE_R1_trim.fq.gz -p SAMPLE_R2_trim.fq.gz \
          SAMPLE_R1.fq.gz SAMPLE_R2.fq.gz
```

After trimming the adapters the reads were converted from FASTQ to FASTA and all R1 and R2 reads combined. The combined R1 and R2 reads were searched for antibiotic resistance genes (ARGs) and _intI1_ integrase genes with [DIAMOND v.0.9.114](http://www.diamondsearch.org/index.php) (Buchfink _et al._ 2015).  
The ARG database was ResFinder v.3.1.0 (Zankari _et al._, 2012) and for _intI1_ integrase gene we used the MGE database from (Pärnänen _et al._, 2019).

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

## Data analysis
After pre-processing the data was imported to R.  Sewage metagenomic data was combined with clinical and socioeconomical data on country level. For countries where there were more than one sewage sample, mean values were used.  

The data analysis steps and making of the figures in R can be found [from here.]( https://karkman.github.io/GlobalSewage/)

## References

- Karkman, A., Berglund, F., Flach, C-F, Kristiansson, E., Larsson, DGJ. 2020. Predicting clinical resistance prevalence using sewage metagenomic data. _Accepted in Communications Biology_
- Hendriksen, R.S., Munk, P., Njage, P., van Bunnik, B., McNally, L., Lukjancenko, O., Röder, T., Nieuwenhuijse, D., Pedersen, S.K., Kjeldgaard, J., et al. (2019). Global monitoring of antimicrobial resistance based on metagenomics analyses of urban sewage. Nat Commun 10, 1124. https://doi.org/10.1038/s41467-019-08853-3
- Martin, M. (2011). Cutadapt removes adapter sequences from high-throughput sequencing reads. EMBnet.Journal 17, 10.
- Zankari, E., Hasman, H., Cosentino, S., Vestergaard, M., Rasmussen, S., Lund, O., Aarestrup, F.M., and Larsen, M.V. (2012). Identification of acquired antimicrobial resistance genes. Journal of Antimicrobial Chemotherapy 67, 2640–2644.
- Pärnänen, K.M.M., Narciso-da-Rocha, C., Kneis, D., Berendonk, T.U., Cacace, D., Do, T.T., Elpers, C., Fatta-Kassinos, D., Henriques, I., Jaeger, T., et al. (2019). Antibiotic resistance in European wastewater treatment plants mirrors the pattern of clinical antibiotic resistance prevalence. Sci. Adv. 5, eaau9124.
- Buchfink, B., Xie, C., and Huson, D.H. (2015). Fast and sensitive protein alignment using DIAMOND. Nature Methods 12, 59–60.

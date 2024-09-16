## Hybrid assembly using OPERA-MS
### Installation
In Software directory
```
module load perl/5.38.2
git clone https://github.com/CSB5/OPERA-MS.git
make
perl OPERA-MS.pl check-dependency
```

After installation I created opera-ms DB with GTDB release 220

```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=96:00:00
#SBATCH --mem=25G
#SBATCH --partition=cpu2022
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --cpus-per-task=8

####### Set environment variables ###############
module load python/3.10.4
module load perl/5.38.2

####### Run your script #########################
wget https://data.gtdb.ecogenomic.org/releases/release220/220.0/genomic_files_reps/gtdb_genomes_reps_r220.tar.gz
wget https://data.gtdb.ecogenomic.org/releases/release220/220.0/bac120_taxonomy_r220.tsv.gz
wget https://data.gtdb.ecogenomic.org/releases/release220/220.0/ar53_taxonomy_r220.tsv.gz

tar xzvf gtdb_genomes_reps_r220.tar.gz
rm gtdb_genomes_reps_r220.tar.gz
zcat ar53_taxonomy_r220.tsv.gz bac120_taxonomy_r220.tsv.gz | gzip > all_taxonomy_r214.tsv.gz
find gtdb_genomes_reps_r220 -type f -name '*.fna.gz' | gzip > all_genomes.txt.gz

python3 ~/software/OPERA-MS/src_utils/make_operams_db_from_gtdb.py all_genomes.txt.gz all_taxonomy_r214.tsv.gz --threads 12 | tqdm
mv OPERA-MS-DB/ ~/software/OPERA-MS/
find -L /home/franciscodaniel.davi/software/OPERA-MS/OPERA-MS-DB/ -type f -name '*.fna.gz' > \
/home/franciscodaniel.davi/software/OPERA-MS/OPERA-MS-DB/genomes_list.txt
conda activate mash
mash sketch -o OPERA-MS-DB/genomes.msh -p 4 -l OPERA-MS-DB/genomes_list.txt
```

Finaly, I checked that OPERA-MS works properly
```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=96:00:00
#SBATCH --mem=25G
#SBATCH --partition=cpu2022
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --cpus-per-task=8

####### Set environment variables ###############
module load python/3.10.4
module load perl/5.38.2

####### Run your script #########################
cd test_files
perl ../OPERA-MS.pl \
    --contig-file contigs.fasta \
    --short-read1 R1.fastq.gz \
    --short-read2 R2.fastq.gz \
    --long-read long_read.fastq \
    --no-ref-clustering \
```

## Improvemnet of assembly 
To improve asselmbly, I use kneaddata to remove low quality reads and remove duplicate (create a conda environment for this software) and chopper for remove reads <500 bp  and quality <8 in nanopore reads **NOTE:For some reason dorado does not remove QC <8 which was the command that I requesrted**

***NOTE CONDA*** For running kneaddata and nanoplot, I need to activate conda environment on the terminal before running  the sbatch script
**Kneaddata output:** Because I do not use a DB for remove read contaminats, the final output of kneaddata to use in the assmebly is repeats.removed.fastq

```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=96:00:00
#SBATCH --mem=25G
#SBATCH --partition=cpu2023
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4

####### Set environment variables ###############
####### Run your script #########################
kneaddata --input1 CRD_H_R3_T1_1.fq.gz --input2 CRD_H_R3_T1_2.fq.gz --output kneaddata_output \
-t 8 -p 2 --remove-intermediate-output --run-fastqc-start --run-fastqc-end \
```

chopper for nanopore reads

```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=12:00:00
#SBATCH --mem=15G
#SBATCH --partition=cpu2023
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4

####### Set environment variables ###############

####### Run your script #########################
cat Replicate3_Initial_Long_reads_07072024.fastq chopper -q 8 -l 500 > filtered_Replicate3_Initial_Long_reads_07072024.fastq
```

Run opera_ms with filtered reads for Replicate 3

```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=96:00:00
#SBATCH --mem=25G
#SBATCH --partition=cpu2023
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4

####### Set environment variables ###############
module load perl/5.38.2

####### Run your script #########################
perl ~/software/OPERA-MS/OPERA-MS.pl \
    --short-read1 Rep3_kneaddata_output/CRD_H_R3_T1_1_kneaddata.repeats.removed.1.fastq \
    --short-read2 Rep3_kneaddata_output/CRD_H_R3_T1_1_kneaddata.repeats.removed.1.fastq \
    --long-read filtered_Replicate3_Initial_Long_reads_07072024.fastq \
    --out-dir Rep3_kneaddata_chopper_hybrid_results --num-processors 8 \
    --no-strain-clustering \
```

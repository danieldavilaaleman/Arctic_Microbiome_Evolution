# Installation of medaka
```
module load python/3.10.4
virtualenv medaka --python=python3 --prompt "(medaka) "
. medaka/bin/activate
pip install --upgrade pip
pip install medaka
```
To install samtools whitout root access, I need to set a prefix for installation on ~/usr directory
```
wget https://github.com/samtools/samtools/releases/download/1.18/samtools-1.18.tar.bz2
tar -xvjf samtools-1.18.tar.bz2
cd samtools-1.18
./configure --prefix=$HOME/usr
make
make install
```
Same thing with bcftool and htslib
# Running sbatch script
```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=6
#SBATCH --time=24:00:00
#SBATCH --mem=150G
#SBATCH --partition=cpu2019

####### Set environment variables ###############
module load python/3.10.4

####### Run your script #########################
. medaka/bin/activate
BASECALLS=~/Wild_2023/2_lenght_filt/filt.trim.wild.all.fastq
DRAFT=~/Wild_2023/4_assembly/meta_flye_assembly/assembly.fasta
OUTDIR=medaka_consensus
medaka_consensus -i ${BASECALLS} -d ${DRAFT} -o ${OUTDIR} -t 6 -m r941_min_fast_g507
```

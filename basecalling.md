# Basecalling using GPU
I used bigmem for GPU basecalling to be faster than cpu basecalling
```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=24:00:00
#SBATCH --mem=40G
#SBATCH --partition=bigmem
#SBATCH --gres=gpu:1
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4

####### Set environment variables ###############
module load python/3.10.4
####### Run your script #########################
dorado basecaller --min-qscore 8 hac --trim adapters pod5/ > gpu_initial_evolution.bam
```

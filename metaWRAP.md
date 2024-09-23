## MetaWRAP pipeline
For Binning, bin refinement and bin-reassembly, I used metaWRAP using the following script
```
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --time=96:00:00
#SBATCH --mem=45G
#SBATCH --partition=cpu2023
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4

####### Set environment variables ###############
source ~/software/miniconda3/etc/profile.d/conda.sh
conda activate metawrap-env
####### Run your script #########################
## BINNING ##
metawrap binning -o Binning_Rep1_kneaddata_hybrid -t 8 -a Rep1_kneaddata_hybrid_results/contigs.polished.fasta \
--metabat2 --maxbin2 --concoct --interleaved -m 40 Replicate1_kneaddata_output/CRD_H_R1_T2_CKDN240006505-1A_HCH7FDSXC_L4_1_kneaddata.repeats.removed.1.fastq \
Replicate1_kneaddata_output/CRD_H_R1_T2_CKDN240006505-1A_HCH7FDSXC_L4_1_kneaddata.repeats.removed.2.fastq

## BIN REFINEMENT ##
metawrap bin_refinement -o Refinement_Binning_Rep1_kneaddata_hybrid -t 8 -A Binning_Rep1_kneaddata_hybrid/metabat2_bins/ \
-B Binning_Rep1_kneaddata_hybrid/maxbin2_bins/ -C Binning_Rep1_kneaddata_hybrid/concoct_bins/ -c 50 -x 10 -m 40

## Bin RE-Assembly ##
metawrap reassemble_bins -o BIN_REASSEMBLY_Replicate1_kneaddata_hybrid \
-1 Replicate1_kneaddata_output/CRD_H_R1_T2_CKDN240006505-1A_HCH7FDSXC_L4_1_kneaddata.repeats.removed.1.fastq \
-2 Replicate1_kneaddata_output/CRD_H_R1_T2_CKDN240006505-1A_HCH7FDSXC_L4_1_kneaddata.repeats.removed.2.fastq \
-t 8 -m 40 -c 50 -x 10 -b Refinement_Binning_Rep1_kneaddata_hybrid/metawrap_50_10_bins
```

However, the re-assembly reduces the N50 of the bins without improving the quality of the bins
### Bin Refinement
bin.6   99.87   0.517   0.575   Gammaproteobacteria     3685647 4165923 binsA\
bin.4   98.02   0.479   0.582   Rhodobacteraceae        3207486 3875359 binsA\
bin.2   94.12   2.818   0.576   Gammaproteobacteria     1437279 4268779 binsA\
bin.5   83.11   2.724   0.538   Gammaproteobacteria     25526   3950401 binsB\
bin.1   67.44   2.102   0.388   Gammaproteobacteria     33241   2872914 binsA\
bin.3   54.68   0.693   0.613   Sphingomonadales        15365   2105367 binsAB\

#### Bin Re-Assembly with SPADES
bin.1.permissive        67.76   2.102   0.388   Gammaproteobacteria     18316   2735448\
bin.2.permissive        95.67   1.027   0.577   Gammaproteobacteria     440751  4184364\
bin.3.permissive        55.22   0.750   0.613   Sphingomonadales        11246   2097065\
bin.4.orig      98.02   0.479   0.582   Rhodobacteraceae        3207486 3875359\
bin.5.permissive        83.68   2.083   0.539   Gammaproteobacteria     17400   3745520\
bin.6.orig      99.87   0.517   0.575   Gammaproteobacteria     3685647 4165923\

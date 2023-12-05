# Predmoter models
The listed models are trained Predmoter (https://github.com/weberlab-hhu/Predmoter)
models. Out of three replicates, meaning three training runs with different seeds,
these models had the highest validation "accuracy", meaning Pearson's correlation. The
models were trained on plant ATAC- and/or ChIP-seq (H3K4me3) data.

>IMPORTANT: Models trained on a GPU can be used to generate predictions on the CPU.
> The CPU predictions will be the same as the GPU predictions would be. Predicting
> on the CPU will take longer.
     
The best model for predicting monocot ATAC-seq data is the BiHybrid_04 model
and for dicot ATAC-seq data the Combined model. The best model for predicting
ChIP-seq is the combined model. (see [3.2 F1 results](#32-f1-results))
     
## Table of contents
1. [Model information](#1-model-information)
2. [Predict with Predmoter](#2-predict-with-predmoter)
3. [F1 statistics](#3-f1-statistics)
   1. [Peak calling parameters](#31-peak-calling-parameters)
   2. [F1 results](#32-f1-results)
4. [Additional information](#4-additional-information)
5. [References](#references)
6. [Citation](#citation)
     
## 1. Model information <a id="1-model-information"></a>
A model can only predict the dataset(s) it was trained on. The model architecture is
listed for completeness.    

| Model name    | Dataset(s)                                                 | Recommendation                                                                                                    | Architecture                                           |
|:--------------|:-----------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------|
| U-Net         | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net                                                 |
| Hybrid        | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net + LSTM                                          |
| BiHybrid      | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net + BiLSTM                                        |
| BiHybrid_02   | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net + BiLSTM + batch normalization                  |
| BiHybrid_03.1 | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
| BiHybrid_03.2 | 	ATAC-seq                                                  | /                                                                                                                 | 	U-Net + BiLSTM + batch normalization + dropout of 0.5 |
| BiHybrid_04   | 	ATAC-seq, filtered flagged sequences*                     | use for monocot ATAC-seq data prediction (see  [3.2. F1 results](#32-f1-results))                                 | U-Net + BiLSTM + batch normalization + dropout of 0.3  |
| BiHybrid_05   | 	ChIP-seq (H3K4me3), filtered flagged sequences*           | /                                                                                                                 | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
| Combined      | 	ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences* | use for ChIP-seq data prediction; use for dicot ATAC-seq data prediction (see  [3.2. F1 results](#32-f1-results)) | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
      
*excluded subsequences of unplaced scaffolds and non-nuclear sequences during training
and testing
     
## 2. Predict with Predmoter <a id="2-predict-with-predmoter"></a>
Example of how to predict with Predmoter on a fasta file using one of the models above
(The batch size depends on the capacity of the CPU/GPU. The default batch size is 120.
If the error ``RuntimeError:CUDA out of memory.`` or other memory errors occur, 
try setting a lower batch size.):
```bash
Predmoter.py -f <species>.fasta -o <output_directory> -m predict \
--model <path_to_model>/Combined_predmoter_v0.3.2.ckpt -b <batch_size> \
-of bigwig --species <species>
```
    
## 3. F1 statistics <a id="3-f1-statistics"></a>
### 3.1 Peak calling parameters <a id="31-peak-calling-parameters"></a>
MACS3 (Zhang et al. 2008) was used for calling peaks. The model trained and was tested
on the mean experimental read coverage of all SRA samples, e.g., all 5 ATAC-seq samples
of *A. thaliana*. Peaks were called from bedGraph files. The experimental peaks, i.e.,
the mean experimental read coverage, were called per test/validation species and NGS
dataset. The predicted peaks of 4 different models, BiHybrid_031, BiHybrid_04,
BiHybrid_05 and the combined model, were called per test and validation species.
For ATAC-seq “bdgpeakcall” was used and for ChIP-seq “bdgbroadcall”. The minimum
peak length was set to the median insert size and the maximum gap size to the mean
read length of all experimental samples of one species and one NGS dataset. Unplaced
scaffolds and non-nuclear sequences were excluded for all species except *S. polyrhiza*.
The parameters used for ATAC-seq peak calling were:
     
| Species (scientific)   | --cutoff                 | --min-length (median insert size) | --max-gap (mean read length) |
|:-----------------------|:-------------------------|:----------------------------------|:-----------------------------|
| *Arabidopsis thaliana* | 10                       | 85                                | 35                           |
| *Oryza sativa*         | 10                       | 60                                | 70                           |
| *Medicago truncatula*  | 35                       | 160                               | 115                          |
| *Spirodela polyrhiza*  | 15                       | 160                               | 35                           |

     
The maximum linking gap of “bdgbroadcall” was set to four times the minimum peak
length. The cutoff parameter of “bdgbroadcall” had no influence on the position,
length or number of the called peaks. The parameters used for ChIP-seq (H3K4me3)
peak calling were:
    
| Species (scientific)   | --cutoff-peak | --cutoff-link | --min-length (median insert size) | --lvl1-max-gap (mean read length) | --lvl2-max-gap |
|:-----------------------|:--------------|:--------------|:----------------------------------|:----------------------------------|:---------------|
| *Arabidopsis thaliana* | 120           | 20            | 180                               | 60                                | 720            |
| *Oryza sativa*         | 120           | 15            | 145                               | 95                                | 580            |
| *Medicago truncatula*  | 120           | 10            | 215                               | 115                               | 860            |
| *Spirodela polyrhiza*  | 60            | 15            | 50*                               | 75                                | 200*           |
*Since the ChIP-seq data for *S. polyrhiza* is single-end, the median insert size is
unavailable. So diffrent values for ``--min-length`` and ``--lvl2-max-gap`` (four
times the min-length) were tested. Since ``--lvl2-max-gap`` has to be higher than
``-lvl1-max-gap``, the lowest value combination tested was 25 and 100.
     
To find the optimal cutoff (“bdgpeakcall”) or linking cutoff (“bdgbroadcall”) an area
under the precision-recall curve (AUPRC) was calculated. The optimal thresholds for
these two parameters were defined as the smallest difference between precision and
recall. Peaks called from the bedGraph files of mean experimental read coverage were
compared with peaks called from the individual SRA sample bam files. The peaks of the
individual samples were called with MACS3’s “callpeak” command, a sophisticated
algorithm calling peaks from alignment results. The parameters used were:    
     
| Species (scientific)   | --format | --gsize     | --qvalue (ATAC-seq/ChIP-seq) | --keep-dup | --broad (ATAC-seq/ChIP-seq) |
|:-----------------------|:---------|:------------|:-----------------------------|:-----------|:----------------------------|
| *Arabidopsis thaliana* | BAMPE    | 119,482,990 | 0.01/0.05                    | yes        | no/yes                      |
| *Oryza sativa*         | BAMPE    | 374,305,350 | 0.01/0.05                    | yes        | no/yes                      |
| *Medicago truncatula*  | BAMPE    | 429,433,753 | 0.01/0.05                    | yes        | no/yes                      |
| *Spirodela polyrhiza*  | BAM*     | 138,570,896 | 0.01/0.05                    | yes        | no/yes                      |
*the format BAM was chosen for calling single-end ChIP-seq peaks

### 3.2 F1 results <a id="32-f1-results"></a>
A base-wise F1 was calculated to quantify predicted peaks matching experimental peaks.
A F1 score of 1 indicates that the predicted peaks are at the same position as the
experimental peaks. The lowest score possible is 0. The F1 of the predicted peaks versus
the experimental peaks was calculated per model, species and NGS dataset. The predicted
and experimental peaks of both strands were compared. Flagged sequences (i.e., unplaced
scaffolds and non-nuclear sequences) were excluded from peak calling and F1 calculation.

>**ATAC-seq:** The best model for predicting monocot ATAC-seq data is the BiHybrid_04
> model. The best model for predicting dicot ATAC-seq data is the Combined model.
    
| Species (scientific)   | Domain  | BiHybrid_03.1 | BiHybrid_04 | Combined   |
|:-----------------------|:--------|:--------------|:------------|:-----------|
| *Arabidopsis thaliana* | dicot   | 0.0748        | 0.0786      | **0.1206** |
| *Oryza sativa*         | monocot | 0.2913        | **0.3008**  | 0.1635     |
| *Medicago truncatula*  | dicot   | 0.0551        | 0.054       | **0.0608** |
| *Spirodela polyrhiza*  | monocot | 0.3383        | **0.3407**  | 0.3328     |

> **ChIP-seq:** The best model for predicting ChIP-seq is the Combined model.   
    
| Species (scientific)   | Domain  | BiHybrid_05 | Combined   |
|:-----------------------|:--------|:------------|:-----------|
| *Arabidopsis thaliana* | dicot   | 0.6759      | **0.7033** |
| *Oryza sativa*         | monocot | 0.6920      | **0.7049** |
| *Medicago truncatula*  | dicot   | 0.4607      | **0.4664** |
| *Spirodela polyrhiza*  | monocot | 0.5309      | **0.5533** |
     

## 4. Additional information <a id="3-additional-information"></a>
For additional information about training, validation and test data, species
selection and exact hyperparameter configuration used, please refer to the
supplementary material of this
[preprint](https://doi.org/10.1101/2023.11.03.565452).

## References <a id="references"></a>
Zhang, Y., Liu, T., Meyer, C. A., Eeckhoute, J., Johnson, D. S.,
Bernstein, B. E., Nussbaum, C., Myers, R. M., Brown, M., Li, W., Shirley, X. S. (2008).
Model-Based Analysis of ChIP-Seq (MACS). Genome Biology, 9(9) , 1–9.
https://doi.org/10.1186/GB-2008-9-9-R137
    
## Citation <a id="citation"></a>
Kindel, F., Triesch, S., Schlüter, U., Randarevitch, L. A., Reichel-Deland, V.,
Weber, A. P. M., & Denton, A. K. (2023). Predmoter - Cross-species prediction of 
plant promoter and enhancer regions. BioRxiv, 2023.11.03.565452.
https://doi.org/10.1101/2023.11.03.565452

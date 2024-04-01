# Predmoter models
The listed models are trained Predmoter (https://github.com/weberlab-hhu/Predmoter)
models. Out of three replicates, meaning three training runs with different seeds,
these models had the highest validation "accuracy", meaning Pearson's correlation. The
models were trained on plant ATAC- and/or ChIP-seq (H3K4me3) data.

>IMPORTANT: Models trained on a GPU can be used to generate predictions on the CPU.
> The CPU predictions will be the same as the GPU predictions would be. Predicting
> on the CPU will take a little longer.
     
The best model for predicting ATAC- and H3K4me3 ChIP-seq seq data is the Combined_02
model.
     
## Table of contents
1. [Model information](#1-model-information)
2. [Predict with Predmoter](#2-predict-with-predmoter)
3. [Additional information](#3-additional-information)
4. [References](#references)
5. [Citation](#citation)
     
## 1. Model information <a id="1-model-information"></a>
A model can only predict the dataset(s) it was trained on. The model architecture is
listed for completeness.    

| Model name    | Dataset(s)                                                                                                                                             | Recommendation                             | Architecture                                                                                           |
|:--------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------|:-------------------------------------------------------------------------------------------------------|
| U-Net         | 	ATAC-seq                                                                                                                                              | /                                          | 	3 convolutional layers + 3 transposed convolutional layers                                            |
| Hybrid        | 	ATAC-seq                                                                                                                                              | /                                          | 	U-Net + 2 LSTM layers                                                                                 |
| BiHybrid      | 	ATAC-seq                                                                                                                                              | /                                          | 	U-Net + 2 BiLSTM layers                                                                               |
| BiHybrid_02   | 	ATAC-seq                                                                                                                                              | /                                          | 	U-Net + 2 BiLSTM layers + 6 batch normalization layers                                                |
| BiHybrid_03.1 | 	ATAC-seq                                                                                                                                              | /                                          | 	U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3) |
| BiHybrid_03.2 | 	ATAC-seq                                                                                                                                              | /                                          | 	U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.5) |
| BiHybrid_04   | 	ATAC-seq, filtered flagged sequences*                                                                                                                 | /                                          | U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3)  |
| BiHybrid_05   | 	ChIP-seq (H3K4me3), filtered flagged sequences*                                                                                                       | /                                          | 	U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3) |
| Combined      | 	ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences*                                                                                             | /                                          | 	U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3) |
| Combined_02   | ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences*, additional training data                                                                    | use for ATAC- and ChIP-seq data prediction | U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3)  |
| IS_10         | ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences*, additional training data; intra-species training and validation split (validation set: 10%) | /                                          | U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3)  |
| IS_20         | ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences*, additional training data; intra-species training and validation split (validation set: 20%) | /                                          | U-Net + 2 BiLSTM layers + 6 batch normalization layers + 1 dropout layer (dropout probability of 0.3)  |
      
*excluded subsequences of unplaced scaffolds and non-nuclear sequences during training
and testing
     
## 2. Predict with Predmoter <a id="2-predict-with-predmoter"></a>
Example of how to predict with Predmoter on a fasta file using one of the models above
(The batch size depends on the capacity of the CPU/GPU. The default batch size is 120.
If the error ``RuntimeError:CUDA out of memory.`` or other memory errors occur, 
try setting a lower batch size.):
```bash
Predmoter.py -f <species>.fasta -o <output_directory> -m predict \
--model <path_to_model>/Combined_02_predmoter_v0.3.2.ckpt -b <batch_size> \
-of bigwig --species <species>
```
For more information, please refer to this
[subsection of the Predmoter documentation](https://github.com/weberlab-hhu/Predmoter#45-inference).

## 3. Additional information <a id="3-additional-information"></a>
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

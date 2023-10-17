# Predmoter models
The listed models are trained Predmoter (https://github.com/weberlab-hhu/Predmoter)
models. Out of three replicates, meaning three training runs with different seeds,
these models had the highest validation "accuracy", meaning Pearson's correlation. The
models were trained on plant ATAC- and/or ChIP-seq (H3K4me3) data.
     
## Model information
A model can only predict the dataset(s) it was trained on. The model architecture is
listed for completeness.    

| Model name    | Dataset(s)                                                 | Recommendation               | Architecture                                           |
|:--------------|:-----------------------------------------------------------|:-----------------------------|:-------------------------------------------------------|
| U-Net         | 	ATAC-seq                                                  | /                            | 	U-Net                                                 |
| Hybrid        | 	ATAC-seq                                                  | /                            | 	U-Net + LSTM                                          |
| BiHybrid      | 	ATAC-seq                                                  | /                            | 	U-Net + BiLSTM                                        |
| BiHybrid_02   | 	ATAC-seq                                                  | /                            | 	U-Net + BiLSTM + batch normalization                  |
| BiHybrid_03.1 | 	ATAC-seq                                                  | /                            | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
| BiHybrid_03.2 | 	ATAC-seq                                                  | /                            | 	U-Net + BiLSTM + batch normalization + dropout of 0.5 |
| BiHybrid_04   | 	ATAC-seq, filtered flagged sequences*                     | use for ATAC-seq prediction  | U-Net + BiLSTM + batch normalization + dropout of 0.3  |
| BiHybrid_05   | 	ChIP-seq (H3K4me3), filtered flagged sequences*           | /                            | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
| Combined      | 	ATAC-seq, ChIP-seq (H3K4me3), filtered flagged sequences* | use for ChIP-seq prediction  | 	U-Net + BiLSTM + batch normalization + dropout of 0.3 |
      
*excluded subsequences of unplaced scaffolds and non-nuclear sequences during training and testing
     

## Peak F1 statistics
A F1 score of 1 indicates that the predicted peaks are at the same position as the
experimental peaks. The lowest score possible is 0. The F1 of the predicted peaks versus
the experimental peaks was calculated per model, test species and NGS dataset. The
predicted and experimental peaks of both strands were compared. Flagged sequences
were excluded from peak calling and F1 calculation.

**ATAC-seq:** The best model for predicting ATAC-seq is the BiHybrid_04 model.
Predictions for the monocot *Oryza sativa* were notably better using the BiHybrid_04
model. Predictions for the dicot *Arabidopsis thaliana* were slightly better using
the combined model.
    
| Species (scientific)   | BiHybrid_03.1 | BiHybrid_04 | Combined |
|:-----------------------|:--------------|:------------|:---------|
| *Arabidopsis thaliana* | 0.1535        | 0.1589      | 0.1686   |
| *Oryza sativa*         | 0.3524        | 0.3857      | 0.1582   |

**ChIP-seq:** The best model for predicting ChIP-seq is the combined model.   
    
| Species (scientific)   | BiHybrid_05 | Combined |
|:-----------------------|:------------|:---------|
| *Arabidopsis thaliana* | 0.7039      | 0.7240   |
| *Oryza sativa*         | 0.6924      | 0.7045   |
     

## Additional information
See here for additional information about training, validation and test data, species
selection and exact hyperparameter configuration used:  
*TBA*

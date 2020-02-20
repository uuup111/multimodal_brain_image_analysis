# Multimodal Brain Tumor Segmentation 

## IDEA
- Use techniques of previous research
- Design new loss function (using the idea of previous paper)
- Reduce size of models
- Handle the open problems of previous research
- Design new model: cascaded, because they are overlap


## Documents
- Data: 
    - http://braintumorsegmentation.org/ (all BRatS data)
    - https://github.com/BraTS/Instructions/blob/master/data_formats.md
- Docker from previous implementation:
    - hub.docker.com/u/brats/
- Documents:
    - From https://www.med.upenn.edu/cbica/brats2019/registration.html
        - https://arxiv.org/pdf/1811.02629.pdf
        - https://www.ncbi.nlm.nih.gov/pubmed/25494501
        - https://www.ncbi.nlm.nih.gov/pubmed/28872634
        - https://wiki.cancerimagingarchive.net/display/DOI/Segmentation+Labels+and+Radiomic+Features+for+the+Pre-operative+Scans+of+the+TCGA-GBM+collection
        - https://wiki.cancerimagingarchive.net/display/DOI/Segmentation+Labels+and+Radiomic+Features+for+the+Pre-operative+Scans+of+the+TCGA-LGG+collection
    - From other sites:
        - https://paperswithcode.com/task/brain-tumor-segmentation
        - https://www.frontiersin.org/articles/10.3389/fncom.2019.00083/full
        - BRATS 2018:
            - https://arxiv.org/pdf/1810.11654v3.pdf (top 1)
            - https://link.springer.com/chapter/10.1007%2F978-3-030-11726-9_21 (top 2)
            - https://link.springer.com/content/pdf/10.1007%2F978-3-030-11726-9_40.pdf (top 3)
            - https://link.springer.com/content/pdf/10.1007%2F978-3-030-11726-9_44.pdf (top 3)
            - others: https://link.springer.com/book/10.1007/978-3-030-11726-9

- Github:
    - https://github.com/JooHyun-Lee/BraTs
    - https://github.com/JunMa11/SOTA-MedSeg
    - https://github.com/xf4j/brats18

## Literature Review
Excel file can be found [here](https://docs.google.com/spreadsheets/d/1M98qnhK85EqXm91nIv08I58I3GaLlqv6hfQQyXoKp3U/edit#gid=1016976313-)

|                                          Paper                                          |                                                                                                                                                                                                                                                       What's new?                                                                                                                                                                                                                                                       | Methods                                                                                                                                                                                                                                                                                                                                             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        What have we learned?                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |                                                                                                                                                               Open problems?                                                                                                                                                              |
|:---------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|           3D MRI brain tumor segmentation using <br>autoencoder regularization          |                                                                                                                                           Due to a limited training dataset size, <br>a variational auto-encoder branch is added <br>to reconstruct the input image itself in order to <br>regularize the shared decoder and<br> impose additional constraints on its layers.                                                                                                                                           | Input: 160x192x128<br>Outputs: output all 3 nested tumor subregions directly after the sigmoid<br><br><br>Using UNET based + VAE to segmentation                                                                                                                                                                                                    | - Add new VAE branch: using the auto-encoder branch is to add additional guidance and regularization to the encoder part, since the training dataset size is limited<br>- For normalization, we use Group Normalization (GN), which shows better than BatchNorm performance when batch size is small<br>- Ldice is a soft dice loss [19] applied to the decoder output ppred to match the segmentation mask ptrue<br>- We have also experimented with more sophisticated data augmentation techniques, including random histogram matching, affine image transforms, and random image filtering, which did not demonstrate any additional improvements.<br>- We have tried several data post-processing techniques to fine tune the segmentation predictions with CRF [14], but did not find it beneficial,<br>- Increasing the network depth further did not improve the performance, but increasing the network width (the number of features/filters) consistently improved the results                                                                                                          | - The size of model is too large, require 2 days of V100 32G to train with batchsize 1 (300 epochs)<br>-The additional VAE branch helped to regularize the shared encoder (in presence of limited data), which not only improved the performance, but helped to consistently achieve good training accuracy for any random initialization |
|                                        No New-Net                                       | - focus on the training process arguing that a well trained U-Net is hard to beat.<br>- incorporating additional measures such as region based training, additional training data, a simple postprocessing technique and a combination of loss functions-<br>- optimize the training procedure to maximize its performance<br>- It uses instance normalization [23] and leaky ReLU nonlinearities and reduces the number of feature maps before upsampling.<br>- using a soft Dice loss for the training of our network | UNET<br>- Input: 128x128x128<br>- Output: softmax or sigmoid                                                                                                                                                                                                                                                                                        | - With MRI intensity values being non standardized, normalization is critical to allow for data from different institutes, scanners and acquired with varying pro- tocols to be processed by one single algorithm<br>- We normalize each modality of each patient independently by subtracting the mean and dividing by the stan- dard deviation of the brain region. The region outside the brain is set to 0. As opposed to normalizing the entire image including the background, this strategy will yield comparative intensity values within the brain region irrespective of the size of the background region around it.<br>- Postprocessing: The BraTS challenge awards a Dice score of 1 if a label is absent in both the ground truth and the prediction. Conversely, only a single false positive voxel in a patient where no enhancing tumor is present in the ground truth will result in a Dice score of 0. Therefore we replace all enhancing tumor voxels with necrosis if the total number of predicted enhancing tumor is less than some threshold<br>- Disadvantage of Dice loss |                                                              One of the main challenges with brain tumor segmentation is the class imbalance in the dataset.<br>Small data, need to avoid overfitting<br>Could we use cascade model? Because there is one GT inside other GT                                                              |
| Ensembles of Densely-Connected CNNs with Label-Uncertainty for Brain Tumor Segmentation | - densely connected blocks of dilated convolutions are embedded in a shallow U-net-style structure of down/upsampling and skip connections.<br>- newly designed loss function which models label noise and uncer-tainty: Label-Uncertainty Loss and Focal Loss                                                                                                                                                                                                                                                          | Densenet-based Architecture, multitask for prediction<br>Input: the input tensor to the model has dimensions 2 * 4 * 5 * 192 * 192.                                                                                                                                                                                                                 | - Design new loss function for problem<br>- the nonzero intensities in the training, validation and testing sets were standardized, this being done across individual volumes rather than across the training set                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Use focal loss to handle imbalance data                                                                                                                                                                                                                                                                                                   |
| Learning Contextual and Attentive Information for Brain Tumor Segmentation              | - design multiple deep architectures<br>of varied structures to learning contextual and attentive information,                                                                                                                                                                                                                                                                                                                                                                                                          | decompose the multi-class brain tumor segmentation into three different but related sub-tasks to deal with the class - imbalance problem.<br>- Model Cascade:<br>- (1) Coarse segmenta- tion to detect whole tumor.<br>- (2) Refined segmentation for whole tumor and its intra-tumoral classes.<br>- (3) Precise segmentation for enhancing tumor. | Attention Mechanism using SE block, Cascade Network                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Using this Cascade model with other research                                                                                                                                                                                                                                                                                              |
## Application of Medical Image Overview
I do a summarization of application from MICCAI 2019 papers [here](./research/application_medical_overview.md)

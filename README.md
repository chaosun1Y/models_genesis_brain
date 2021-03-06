# models_genesis_brain
Models Genesis for brain data. Tried for ATLAS dataset and CQ500 dataset. Use the following steps:

1. Download datasets for [ATLAS](http://fcon_1000.projects.nitrc.org/indi/retro/atlas.html) and [CQ500](http://headctstudy.qure.ai/dataset). The CQ500 dataset needs to be rearranged. Use the script in the folder CQ500_data to do the rearrangement. 
2. Divide the subjects into sub-folders as per instructions in the directories ATLAS_data and CQ500_data. 
3. Now comes the computation part. We will first take small cubes from the MRI (and CT volumes) for each subject's scans. There are some caveats to this (such as the minimum number of "relevant" pixels you want in each cube, whether you want to use the cube for segmentation or classification, if you're going to do segmentation then you need to include some part of the lesion in the extracted cube, the size and number of extracted cubes from each scan, et cetera). Use code from the create_cubes folder for this process. 
    - `infinite_generator_3D_sukrit_CT_brain.py` file creates cubes from the CQ500 dataset. It creates a label for each cube based on whether the subject has been declared diseased/healthy from the reads.csv file. Use the reads.csv file (with an added last column that I provide in the CQ500_data folder).
    - `infinite_generator_3D_sukrit_MRI_brain_classification.py` file takes the cubes out of the ATLAS dataset for classification. Although all the scans in ATLAS have lesions, we can figure out from the segmentation masks whether the partcicular cube that we're picking has a lesion in it. We create corresponding *.npy file for the cube labels, such that a cube with a lesion is marked as 1 and without a lesion is marked as 0.
    - `infinite_generator_3D_sukrit_MRI_brain.py` file takes the cubes out of the ATLAS dataset for segmentation. The scans have small areas where the lesions are localized. Therefore, it is important to pick cubes in the areas where the lesions are. We figure this out from the segmentation masks whether the partcicular cube that we're picking has enough volume of lesion in it. We create corresponding *.npy file for the cube segmentation mask.

    All the files have a corresponding .sh file to run for multiple sub-folders in the ATLAS_data and CQ500_data folders.

4. Now comes the pretraining time for models_genesis. The transformation functions described in the Models_Genesis paper are in the `models_genesis_brain/utils.py` file. The cubes generated should be stored in folders inside the models_genesis_brain directory. `The Genesis_Brain_MRI.py` file reads the ATLAS MRI data to pretrain the U-Net weights. The `Genesis_Brain_CT.py` file reads the CQ500 CT data to pretrain the U-Net weights. They have their corresponding config files (`config_MRI.py` and `config_CT.py`, respectively). You can modify the config files to pick the cubes from their corresponding dirs. Please note that for ATLAS MRI you can train the network on either/both the cube sets created (since pretraining doesn't involve labels in any way).

5. After pretraining is done (should take some time, few days on a 16GB P100 GPU for me), time has come for training and testing on the train and test data, respectively. We have 3 tasks:
    - Classification on the CQ500 CT data: Use the `Genesis_Brain_CT_classification.py`	and `config_CT.py` files for this.
    - Classification on the ATLAS MRI data: Use the `Genesis_Brain_MRI_classification.py`	and `config_MRI_classification.py` for this.
    - Segmentation on the ATLAS MRI data: Use the `Genesis_Brain_MRI_segmentation.py` and `config_MRI.py` for this. 
    
    Set the `type_of_model` variable in the corresponding `conf_*.py` file in order to train the U-Net model from scratch (value of `type_of_model` = 'scratch') or use the weights from the Models Genesis pre-training (value of `type_of_model` = 'MG_pretrained'). 

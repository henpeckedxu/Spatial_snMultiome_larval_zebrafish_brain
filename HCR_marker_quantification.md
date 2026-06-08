**Transform MapZbrain (mzb) space to Zbrain (zb) space**</br>
All HCR images were made with MapZbrain template. For MAPmap analysis, these images needs to be transformed to Zbrain space;</br> 
We performed the transformation with two parts:</br>
1). download a template of a same transgenic line Huc-H2BGCAMP from both database and register the mzb template to zb template.</br>
2). the registration generates transformation matrics that is then applied to HCR marker images for transformation with following steps:</br>

Step1. In `HCR_temp/HCR_mzb_orig/`, Download all HCR original images from MapZbrain database using `HCR_download_v1.ipynb` with a json file `markers_catalog.json`;

Step2. In `HCR_temp/HCR_mzb_reform/`, reform the HCR original images by running a fiji macro `Preprocess_mzb_v2.ijm`;

Step3. Transfer reformed MapZbrain (mzb) HCR images from local computer to wynton. Due to the limited space on Wynton, I always transfer a subset for registration
```
scp HCR_temp/HCR_mzb_reform/mzb_[Cc]*.nii henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb_reform/
```
Step4. Register mzb HCR images to zbrain template using the bash loop below. All files under the folder were processed in parallele</br>
```
for file in ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb_reform/*;do name="${file##*/}"; base="${name%.*}"; qsub ~/LD_NeuralNetwork/bin/HCR_mzb2zb_v1.sh ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/ref Zbrain_Elavl3-H2BRFP.nii "$base"; done
```
Step5. Dowload the registered image to local computer folder `HCR_temp/HCR_mzb2zb`
```
cd HCR_temp/HCR_mzb2zb
scp 'henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb2zb/*toZbrain*' .
```
Step6. Remove all the output files from Step3-4 on wynton
```
rm ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb_reform/*
rm ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb2zb/*
```


Step6. Change the registered HCR images from 32bit to 16bit and stored in folder `HCR/HCR_mzb2zb_downsample/input/`

Step7. Process the registered HCR images for MAPMap analysis using a fiji macro `PrepareStacksForMAPMapping.ijm`. The results are stored in `HCR/HCR_mzb2zb_downsample/output/`

**Chunk file construction** <br>
The MAPMap analysis performs pairwise comparisons for all pairs of 293 HCR markers. In total, there are 42,778 pairs and comparison all pairs in one process is not possible. For computational efficiency, these pairs need to be partitioned into chuck files. 

Step1. Make an empty folder for each HCR gene under a parent folder `HCR_temp/MAPmap_input/` in local computer using `EmptyFolder_v1.ipynb`.</br>

Step2. Transfer the entire folder to Wynton
```
scp -r HCR_temp/MAPmap_input/ henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/MAPmap/
```
Step3. Make chunk files on Wynton
```
module load matalb
matlab -batch "make_chunks_v1('/wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/MAPmap_input/','/wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/chunks/',500)"
```
Step4. Download the resulted folder `chunks` to local computer as `HCR_temp/chunks/`


Run MAPmap with chunk file on Wynton</br>
```bash
qsub MAPmap_analysis_v1.sh chunk_001.mat
```

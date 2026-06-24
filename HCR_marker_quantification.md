## Transform MapZbrain (mzb) space to Zbrain (zb) space
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
Step4. Register mzb HCR images to zbrain template using the bash loop below. All files under the folder were processed in parallel</br>
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

Step7. Change the registered HCR images from 32bit to 16bit and stored in folder `HCR/HCR_mzb2zb_downsample/input/`

Step8. Process the registered HCR images for MAPMap analysis using a fiji macro `PrepareStacksForMAPMapping.ijm`. The results are stored in `HCR/HCR_mzb2zb_downsample/output/`

## Chunk file construction
The MAPMap analysis performs pairwise comparisons for all pairs of 293 HCR markers. In total, there are 42,778 pairs and comparison all pairs in one process is not possible. For computational efficiency, these pairs need to be partitioned into chuck files. 

Step1. Make an empty folder for each HCR gene under a parent folder `HCR_temp/MAPmap_input/` in local computer using `EmptyFolder_v1.ipynb`.</br>

Step2. Transfer the entire folder to Wynton
```
scp -r HCR_temp/MAPmap_input/ henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/MAPmap/
```
Step3. Make chunk files on Wynton
```
module load matlab
matlab -batch "make_chunks_v1('/wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/MAPmap_input/','/wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/chunks/',500)"
```
Step4. Download the resulted folder `chunks` to local computer as `HCR_temp/chunks/`</br>

Step5. make a dataset with all pairs from all chunk using `chunk_info_v1.ipynb` and save the file as `HCR_temp/chunks/all_chunk_pairs.pkl`


## MAPmap Analysis

Step1. Run MAPmap with chunk file on Wynton</br>
```
cd ~/LD_NeuralNetwork/bin
```

```bash
qsub MAPmap_analysis_v1.sh chunk_001.mat
```
or 
```
for i in {30..50}; do qsub MAPmap_analysis_v1.sh chunk_$(printf "%03d" "$i").mat; done
```

Step2. In the output folder, check if the number of files with name containing *SignificantDeltaMedians* equal to the double of the pairs in the corresponding chunk using `HCR_temp/chunks/all_chunk_pairs.pkl`</br>
for individual result folder:</br>
```
find /wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/output/chunk_00i/ -maxdepth 1 -type f  -name '*SignificantDeltaMedians*' | wc -l
```
for a set of result folders:</br>
```
for i in {5..10}; do echo chunk_$(printf "%03d" "$i"); find /wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/output/chunk_$(printf "%03d" "$i") -maxdepth 1 -type f  -name '*SignificantDeltaMedians*' | wc -l ; done
```

Step3. Save the folder to Box</br>
```
for i in {96..100}; do n=$(printf '%03d' "$i"); rclone copy "/wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/output/chunk_${n}" "box:UCSF/Research/Project15LarvalZebrafishSource/MAPmap_output/chunk_${n}" --progress; done
```

‼️ Make sure Step2 and Step3 have been completed before moving to next step

Step4. On Wynton, remove all other files in the result folder except files with name containing SignificantDeltaMedians.</br>

for individual result folder:</br> 
```
find /wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/output/chunk_00i/ -maxdepth 1 -type f ! -name '*SignificantDeltaMedians*' -delete
```
for a set of result folders:</br>
```
for i in {5..10}; do find /wynton/home/guolab/henpeckedxu/LD_NeuralNetwork/experiments/MAPmap/output/chunk_$(printf "%03d" "$i") -maxdepth 1 -type f  ! -name '*SignificantDeltaMedians*' -delete ; done
```

## Analysis at the level of anatomical region 
Step1. On Wynton, use the output from step2 to find anatomical regions with differential expression between HCR markers.</br>
```
cd ~/LD_NeuralNetwork/bin/Z-Brain-master/
```

```bash
for i in {12..20}; do qsub ZBrainAnalysisOfMAPMaps_v2.sh "$(printf '%03d' "$i")"; done
```
Step2. Compress the results into a tar.gz file and transfer it to box
```
for i in {51..98}; do     n=$(printf '%03d' "$i");      echo "Processing chunk_${n}";      cd ~/LD_NeuralNetwork/experiments/MAPmap/output/chunk_${n} &&     tar -czf ../chunk_${n}.tar.gz ./*SignalResults.csv &&     rclone copy ../chunk_${n}.tar.gz         box:UCSF/Research/Project15LarvalZebrafishSource/BrainMaskAnalysis/         -P; done
```

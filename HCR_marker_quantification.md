Step1. Download all HCR orignal images from MapZbrain database using `HCR_download_v1.ipynb` with a json file;

Transfer reformed MapZbrain (mzb) HCR images from local computer to wynton</br>
Due to the limited space on Wynton, I always transfer a subset for registration
```
scp HCR_temp/HCR_mzb_reform/mzb_[Cc]*.nii henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb_reform/
```
Register mzb HCR images to zbrain template</br>
Using the bash loop below, all files under this folder were processed in parallele</br>
```
for file in ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/HCR_mzb_reform/*;do name="${file##*/}"; base="${name%.*}"; qsub HCR_mzb2zb_v1.sh ~/LD_NeuralNetwork/experiments/antsRegistration/HCR/ref Zbrain_Elavl3-H2BRFP.nii "$base"; done
```
Dowload the registered image to local computer folder `HCR_temp/HCR_mzb2zb`
```
cd HCR_temp/HCR_mzb2zb
scp 'henpeckedxu@dt2.wynton.ucsf.edu:~/LD_NeuralNetwork/experiments/antsRegistration/HCR/ref/*toZbrain*' .
```


Run MAPmap with chunk file on Wynton</br>
```bash
qsub MAPmap_analysis_v1.sh chunk_001.mat
```

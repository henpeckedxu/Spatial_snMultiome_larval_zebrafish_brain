## This file describes results of Hicat consensus iterative clustering

### Part 1. Parameters tuning for consensus clustering
Parameters for the function scrattch.hicat::run_consensus_clust() were tuned in a total of 46 tests using s08 library after QC. Parameters used for each test is stored in `./parameter_tests/Parameter_setup.xlsx`. Clustering results of each test were stored in `./parameter_tests/test[1-40].rds`</br>
### Part 2. Perform consensus clustering 
1. Apply the tuned parameters to each sample for consensus clustering with 10 replicates for each sample as below </br>
```
for i in {1..10}; do   qsub hicat_consensus_test_v3.sh     ~/multiome_sda_vda/hicat/consensus_cl/s03_qc_control.rds     ~/multiome_sda_vda/hicat/consensus_cl/s03_consensus_cl_test${i}; done
```
2. Assess the results of the 10 replicates for each sample for max_pct (the cell percentage of the largest cluster), DE score and DE num (number of DEs) with bash as below</br>
```
qsub hicat_cs_cl_check_v1.sh s0[1-8]
```
3. Download the assessment results onto local folder `./cs_cl/assess/`. Among the results with the minimum DE num > 3 and minimum DE score > 40, select the one with lowest max_pct as the final clustering results for the sample. In case the max_pct difference between two results were less than 0.05, select the one with higher minimum DE score</br>
4. The selected reasults were saved in the local folder</br>```
5.      

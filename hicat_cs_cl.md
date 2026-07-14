### Part 1. Parameters tuning for consensus clustering

### Part 2. Perform consensus clustering 
1. Apply the tuned parameters to each sample for consensus clustering with 10 replicates for each sample as below </br>
```
for i in {1..10}; do   qsub hicat_consensus_test_v3.sh     ~/multiome_sda_vda/hicat/consensus_cl/s03_qc_control.rds     ~/multiome_sda_vda/hicat/consensus_cl/s03_consensus_cl_test${i}; done
```
2. assess the results of the 10 replicates for each sample for max_pct (the cell percentage of the largest cluster), DE score and DE num (number of DEs)</br>
3. among the results with the minimum DE num > 3 and minimum DE score > 40, select the one with lowest max_pct as the final clustering results for the sample</br>
4.    

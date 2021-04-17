---
layout: page
title: Quality Trimming
permalink: /qualitytrimming/
---

# Quality Trimming
Before sequence alignment to a reference genome, you need to trim your sequence data of bases that you have low confidence in being correct. There are a few programs that do this, but I used Fast P and will go through the settings I used for this program. 


## Helpful Links  
[Fast P Manual](https://github.com/OpenGene/fastp#:~:text=fastp%20supports%20global%20trimming%2C%20which,or%20%2D%2Dtrim_tail1%3D1%20option.)  


## Example Submission Script to Cluster
```
#!/bin/bash
#SBATCH --job-name=Pop9_18101_submitFastP.txt
#SBATCH --mem=2Gb
#SBATCH --mail-user=schaal.s@northeastern.edu
#SBATCH --mail-type=FAIL
#SBATCH --partition=lotterhos
#SBATCH --time=1:00:00
#SBATCH --nodes=1
#SBATCH --tasks-per-node=1
#SBATCH --output=FastP_Out/jobSum/clustOut/Pop9_18101.%j.out
#SBATCH --error=FastP_Out/jobSum/clustOut/Pop9_18101.%j.err

module load lotterhos/2020-08-24
source activate lotterhos-py38

fastp --in1 Stacks_Out/Pop9_18101.1.fq.gz --in2 Stacks_Out/Pop9_18101.2.fq.gz --out1 FastP_Out/Pop9_18101.R1.fq.gz --out2 FastP_Out/Pop9_18101.R2.fq.gz -q 15 -u 50 --trim_front1 1 --cut_front --cut_tail --cut_window_size 5 --cut_mean_quality 15 -j FastP_Out/jobSum/Pop9_18101.fp.json -h FastP_Out/jobSum/Pop9_18101.fp.html &> FastP_Out/jobSum/Pop9_18101.fp.trim.log
```

```--in1``` This is read 1 for a given sample that uses paired end sequencing. For paired end sequencing, you will have two input files that need to be identified by ```--in1``` and ```--in2```. If doing single end sequencing, you only need the ```--in``` flag.  

```--in2``` This is read 2 for a given sample that uses paired end sequencing.  

```--out1``` This is the location and name of the out file for the read 1.  

```--out2``` This is the location and name of the out file for the read 2.  

```-q``` this is used as a filter on the PHRED qualiy score that each base is given from the sequencing facility. PHRED quality scores range from 0-40 with 40 being the highest quality. Here I removed any base that had a quality score of less than or equal to 15.  

```-u``` this is used as a filter on the unqualified percent limit by specifying how many percents of bases are allowed to be unqualified (0-100).  

```--trim_front1```

```--cut_front```

```--cut_tail```

```--cut_window_size```

```--cut_mean_quality```

```-j```

```-h```



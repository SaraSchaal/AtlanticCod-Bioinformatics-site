---
layout: page
title: Stacks
permalink: /stacks/
---

# Stacks
If your samples need to be demultiplexed, running the program Stacks is a necessary first step. Stacks process_radtags function searches sequence reads for barcodes that you supply and demultiplex the reads using your barcode file specifiying which barcode belongs to which sample. This program will also trim off those barcodes. 

My samples were multiplexed using 3 different unique sequences. Every sample had a unique combination of an i5 primer, i7 primer and adapter barcode. Data from the sequencing facility will demultiplex to the level of unique combinations of i5 and i7 primers. Therefore for my samples, I had 32 unique i5 and i7 primer combinations and so the sequencing facility sent me 64 fastq.gz files one for the forward sequence and one for the reverse for all 32 combinations. I will take one file as an example in the following code. 

### Helpful Links
```process_radtags``` manual and specific walkthrough for ```process_radtags```  
[Manual](https://catchenlab.life.illinois.edu/stacks/manual/)  
[Walkthrough](https://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php)  


## Example Fastq File from Sequencing Facility 

### Read 1
```
[schaal.s@login-00 CodGenomes]$ zcat i5-2-i7-9_R1_001.fastq.gz | head -n 10

@GWNJ-1012:218:GW191226406th:1:1101:1090:1000 1:N:0:GATCAG+ATAGAGGC
ACTTGACTGTGCGTTGGCCTGCGGGCTGACTCGGTCCTGAGATGGACTGCTGTGTAGTTTGAACCATAGATTCATTATATAGAACACGGTCTCCTCTGCGCTGCTGGCCAATGGAGCCGAACGTCCGCACTGGCGGGCGGCCATCTTGCC
+
FF:,FF,FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFF,:FFFFFFFFF:FFFFFFF:FFFFFFFFFFFFF:FFFFFFF
@GWNJ-1012:218:GW191226406th:1:1101:1687:1000 1:N:0:GATCAG+ATAGAGGC
ACTTGATCCCTCTCACTCTCCTCTCCGTCTCCTCTTTTGTCCTCGTCTCTCTCCTCTCTCCCTCTCTCCCATCTCCCTCTCTATCAAGTAGATCGGAAGAGCACACGTCTGAACTCCAGTCACGATCAGATCTCGTATGCCGTCTTCTGC
+
FFFFFF,FFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFF:FFFF:::F,F::FFFFF
@GWNJ-1012:218:GW191226406th:1:1101:6840:1000 1:N:0:GATCAG+ATAGAGGC
ACTTGAAAAAAATACATAGCGGCCATGGACAGGATGACCTCTATGACAATGATAGAAACAGAAAGGACGCGGAGACTCTTGAGTCATCAAGTAGATCGGAAGAGCACACGTCTGAACTCCAGTCACGATCAGATCTCGTATGCCGTCTTC
```	
### Read 2
```
[schaal.s@login-00 CodGenomes]$ zcat i5-2-i7-9_R2_001.fastq.gz | head -n 10

@GWNJ-1012:218:GW191226406th:1:1101:1090:1000 2:N:0:GATCAG+ATAGAGGC
ACTTGAAGGCAAGATGGCCGCCCGCCAGTGCGGACGTTCGGCTCCATTGGCCAGCAGCGCAGAGGAGACCGTGTTCTATATAATGAATCTATGGTTCAAACTACACAGCAGTCCATCTCAGGACCGAGTCAGCCCGCAGGCCAACGCACA
+
FFFFFF,FFFF:FFFFFF:FFFFFFFFFFFFFFFFFF:FFFFFFF:,FFFFF:FF,FFFFFFFFFFFFFFF,F,:FFFF:F,FFFFF:FF,FFFF,FFFFFF,FFF,FF:FFFFF:F:FFFF,FFF:F:F:FFFFFFFFFFF,:FFFFFF
@GWNJ-1012:218:GW191226406th:1:1101:1687:1000 2:N:0:GATCAG+ATAGAGGC
ACTTGAAAGAGAGGGAGATGGGAGAGAGGGAGAGAGGAGAGAGACGAGGACAAAAGAGGAGACGGAGAGGAGAGTGAGAGGGATCAAGTAGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTGCCTCTATGTGTAGATCTCGGTGGTCGC
+
FFFFFF,FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF,FFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFF:FF,FF:
@GWNJ-1012:218:GW191226406th:1:1101:6840:1000 2:N:0:GATCAG+ATAGAGGC
ACTTGAAGACTCAAGAGTCTCCGCGTCCTTTCTGTTTCTATCATTGTCATAGAGGTCATCCTGTCCATGGCCGCTATGTATTTTTATCAAGTAGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTGCCTCTATGTGTAGATCTCGGTGGT
```

## Stacks - process_radtags

### Barcode File
For each set of paired end files that need to be demultiplexed, a barcode file needsto be created. For example, for files i5-4-i7-11_R1_001.fastq.gz and i5-4-i7-11_R2_001.fastq.gz its barcode file is as follows:

```
ATCACG	ATCACG	Pop6_18001
CGATGT	CGATGT	Pop7_18173
TTAGGC	TTAGGC	Pop8_18148
TGGCCA	TGGCCA	Pop9_18064
ACAGTG	ACAGTG	Pop3_16228
GCCAAT	GCCAAT	Pop4_17210
CAGATC	CAGATC	Pop6_18035
ACTTGA	ACTTGA	Pop6_18038
GATCAG	GATCAG	Pop8_18130
TAGCTT	TAGCTT	Pop8_18113
GGCTAC	GGCTAC	Pop7_18190
CTTGCA	CTTGCA	Pop1_17327
```

This file contains the 12 barcodes that have the i5_4 and i7_11 index and which sample that combination belongs to. This file is tab delimited and because I had inline barcodes, meaning they were the same on both ends of the read, I have the same barcode sequence in the first and second column. Then the sample ID is on the right. 

Sample naming convention is important for downstream steps. The way my samples are names are first a Pop identifier for what a priori population they belonged to (see Sample Notes file for my population IDs). Then a 5 digit identifier for the specific sample from that population where the first two digits represent the year the sample was collected and the last 3 digits represent the indivdiual sample ID from that sampling year.

I wrote a simple R script to create the barcodefiles for each primer index combination. See file: Stacks_FileCreate.R

### Additional Flags 
 
```-p``` path to your genome files. If you have paired samples put your two fastq files in the same folder and only have those two files in it. I created folders for each index combination to keep everything organized and easy to run stacks on.

```-o``` path to where you want your output files to go. For subsequent steps, you'll want all of these demultiplexed files in the same folder. Create a folder like "Stacks_Out" where you write all your files to. 

```-b``` path to your barcode file 

```--inline_inline ``` The barcode option flag has 6 different options and you have to use the proper flag for you your barcodes or indexes need to be demultiplexed. Because my samples have inline barcodes on either end of the sequence I need to use the --inline_inline flag. 

```-P``` If paired end sequencing was used you need to add the -P flag to indicate this.

```--disable_rad_check``` this flag is needed if you are doing anything other than RAD-seq. Stacks defaults to requiring cutsite sequences that are used in RAD-seq, but for WGS you won't have cut sites and need to disable the rad check.

```-r``` this flag allows Stacks to rescue any barcodes that are off by one bp 


### Example code for running on the cluster - stacks_i54_i711.sh
```
#!/bin/bash
#SBATCH --job-name=stacks_i54_i711                # sets the job name
#SBATCH --mem=1Gb                                 # reserves 10 GB memory
#SBATCH --partition=lotterhos                     # requests that the job is executed in lotterhos partition 
#SBATCH --mail-user=schaal.s@northeastern.edu
#SBATCH --mail-type=FAIL
#SBATCH --time=24:00:00                           # reserves machines/cores for 24 hours.
#SBATCH --output=stacksi54_i711.%j.out                # sets the standard output to be stored in file, where %j is the job id
#SBATCH --error=stacksi54_i711.%j.err                 # sets the standard error to be stored in file
	
module load lotterhos/2019-11-15

srun process_radtags -P -p i54_i711/GenomeFilesi54_i711/ -o ../Stacks_Out/ -b i54_i711/barcodeFilei54_i711.txt --inline_inline --disable_rad_check -r
```

### Results 
Time: 				each file took approx. 7 hrs  
Memory: 			approx. 80 MB RAM


### Read 1 Post Stacks for example sample Pop3_17304 

```
[schaal.s@login-00 Stacks_Out]$ zcat Pop3_17304.1.fq.gz | head -n 10

@218_1_1101_1090_1000/1
CTGTGCGTTGGCCTGCGGGCTGACTCGGTCCTGAGATGGACTGCTGTGTAGTTTGAACCATAGATTCATTATATAGAACACGGTCTCCTCTGCGCTGCTGGCCAATGGAGCCGAACGTCCGCACTGGCGGGCGGCCATCTTGCC
+
,FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFF,:FFFFFFFFF:FFFFFFF:FFFFFFFFFFFFF:FFFFFFF
@218_1_1101_1687_1000/1	TCCCTCTCACTCTCCTCTCCGTCTCCTCTTTTGTCCTCGTCTCTCTCCTCTCTCCCTCTCTCCCATCTCCCTCTCTATCAAGTAGATCGGAAGAGCACACGTCTGAACTCCAGTCACGATCAGATCTCGTATGCCGTCTTCTGC
+
,FFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFF:FFFF:::F,F::FFFFF
@218_1_1101_6840_1000/1
AAAAAATACATAGCGGCCATGGACAGGATGACCTCTATGACAATGATAGAAACAGAAAGGACGCGGAGACTCTTGAGTCATCAAGTAGATCGGAAGAGCACACGTCTGAACTCCAGTCACGATCAGATCTCGTATGCCGTCTTC
```

### Read 2 Post Stacks for example sample Pop3_17304 
```
[schaal.s@login-00 Stacks_Out]$ zcat Pop3_17304.2.fq.gz | head -n 10

@218_1_1101_1090_1000/2
AGGCAAGATGGCCGCCCGCCAGTGCGGACGTTCGGCTCCATTGGCCAGCAGCGCAGAGGAGACCGTGTTCTATATAATGAATCTATGGTTCAAACTACACAGCAGTCCATCTCAGGACCGAGTCAGCCCGCAGGCCAACGCACA
+
,FFFF:FFFFFF:FFFFFFFFFFFFFFFFFF:FFFFFFF:,FFFFF:FF,FFFFFFFFFFFFFFF,F,:FFFF:F,FFFFF:FF,FFFF,FFFFFF,FFF,FF:FFFFF:F:FFFF,FFF:F:F:FFFFFFFFFFF,:FFFFFF
@218_1_1101_1687_1000/2
AAGAGAGGGAGATGGGAGAGAGGGAGAGAGGAGAGAGACGAGGACAAAAGAGGAGACGGAGAGGAGAGTGAGAGGGATCAAGTAGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTGCCTCTATGTGTAGATCTCGGTGGTCGC
+
,FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF,FFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFF:FF,FF:
@218_1_1101_6840_1000/2
AGACTCAAGAGTCTCCGCGTCCTTTCTGTTTCTATCATTGTCATAGAGGTCATCCTGTCCATGGCCGCTATGTATTTTTATCAAGTAGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTGCCTCTATGTGTAGATCTCGGTGGT
```

Day 3 Afternoon
===============
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

Klebsiella pneumoniae comparative genomic analysis 
--------------------------------------------------

To finish up the workshop we are going to go through the process of working up a complete dataset, from start to finish.  This set of genomes originated from a regional outbreak of bla-KPC carrying Klebsiella pneumoniae – one of the most concerning healthcare associated pathogens. 
The goal is to follow up on a previously [published](http://cid.oxfordjournals.org/content/53/6/532.abstract) epidemiologic analysis, and see if genomics supports prior epidemiologic conclusions and can provide additional insights. 

The results of this genomics analysis were published in [this](http://stm.sciencemag.org/content/9/417/eaan0093) paper.

We have our genomes, and we know in which regional facility each isolate originated. 

The goal of this exercise is to:

1) process our genomes (QC, variant calling), 

2) perform a phylogenetic analysis and 

3) overlay our meta-data. 

To make this more difficult, the instructions will be much more vague than in previous sessions, and you will be challenged to use what you have learned, both in the past three days and in the prior workshop, to complete this analysis. 

Hopefully we’ve prepared you to take on the challenge, but remember this is an open book test! 

Feel free to lean on materials from the workshops, manuals of tools and Google (and of course instructors and neighbors). 

Execute the following command to copy files for this afternoon’s exercises to your scratch directory:

```

cd /scratch/micro612w21_class_root/micro612w21_class/username

or 

wd

cp -r /scratch/micro612w21_class_root/micro612w21_class/shared/data/day3pm ./

```

Perform QC on fastq files
-------------------------

On the first morning you ran FastQC to evaluate the quality of a single genome. However, a typical project will include many genomes and you will want to check the quality of all of your samples. From the bash workshop, I hope you can appreciate that you do not want to process 100 genomes by typing 100 commands – rather you want to write a short shell script to do the work for you!


> ***i. Edit the shell script fastqc.sh located in /scratch/micro612w21_class_root/micro612w21_class/your username/day3pm to run FastQC on all fastq files.***

**Important info about this shell script** 
- The shell script includes a for loop that loops over all of the genomes in the target directory
- The tricky part of this shell script is that each fastq command contains two files (forward and reverse reads). So, you need to take advantage of the fact that the forward and reverse read files both have the same prefix, and you can loop over these prefixes. 
- You should be able to get prefixes by piping the following unix commands: ls, cut, sort, uniq
- The prefix should be a part of both forward and reverse reads. For example, the file_prefix for samples Rush_KPC_264_1_sequence.fastq.gz and Rush_KPC_264_2_sequence.fastq.gz should be Rush_KPC_264
- when you are testing your shell script, comment out (using #) the lines below echo so you can see that if the script is 'echo'-ing the correct commands.
- Try running multiqc inside the script by adding the multiqc command with appropriate out directory 
- Don't run multiqc inside for loop and should be run only after the for loop ends.


The fastq files are located in:

```
/scratch/micro612w21_class_root/micro612w21_class/shared/data/day3pm_fastq/
```

Rather than copying these to your directory, analyze the files directly in that directory, so everyone doesn’t have to copy 25G to their home directories. 

Copy and paste commands to run fastqc.sh as slurm script, into a slurm script and submit this slurm script as a job to great lakes.

Your slurm script wil contain the following command after the slurm preamble stuff(Make sure your $SLURM_SUBMIT_DIR is set inside the slurm script):

```bash fastqc.sh /scratch/micro612w21_class_root/micro612w21_class/shared/data/day3pm_fastq/ ```


> ***ii. Examine output of FastQC to verify that all samples are OK***

Check the multiqc report of your fastq files.

Explore ARIBA CARD and MLST typing on day3pm_fastq samples
----------------------------------------------------------

On Day 2 afternoon, you explored ARIBA's MLST results that were performed on a single genome. However, a typical public epidimiology project will include many genomes sampled from different sites and you will want to sequence type each of these samples to study their genetic diversity and determine if these samples contain antibiotic resistance genes. 

Since both MLST typing and CARD resistance gene detection takes a while to run on these samples, We already processed them and have placed the results in day3pm folder.

> i. Explore MLST typing results. 

Try exploring mlst.sh and mlst.sbat scripts that were used to generate these results and try to understand the implementation of for loop in running them sequentially.

You can find the processed MLST results for day3pm_fastq samples in this folder:

/scratch/micro612w21_class_root/micro612w21_class/shared/data/day3pm/MLST_results

Explore and run summarize_mlst.sh and check the dominant sequence type that these samples belong to.

> ii. Summarize ARIBA CARD results

Now go to CARD_results folder under day3pm and explore card.sh and card.sbat scripts that were used to generate these results.

Try to understand the implementation of for loop in running them sequentially.

Use Ariba's summary function that we used in day2pm and summarize ARIBA CARD results that are placed in:

/scratch/micro612w21_class_root/micro612w21_class/shared/data/day3pm/CARD_results

> iii. Drag and drop Phandango associated files onto the Phandango website and check what type of actibiotic resistance genes are present in these genomes?

> iv. Explore full ARIBA matrix in R and plot a heatmap to visualize the presence/absence of various resistancve genes.

how many of these samples contain carbapenam resistant gene - KPC?

<!---
commenting out SPANDx and replacing it with snippy 2021-04-16
Examine results of [SPANDx](http://www.ncbi.nlm.nih.gov/pubmed/25201145) pipeline
---------------------------
[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3pmnoon/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

On the afternoon of day 1 we saw how many steps are involved in calling variants relative to a reference genome. However, the same steps are applied to every sample, which makes this very pipeline friendly!  So, you could write your own shell script to string together these commands, or take advantage of one of several published pipelines. Here, we will use the output of the SPANDx pipeline, which takes as input a directory of fastq files and produces core variant and indel calls.

More information on SPANDx pipeline can be obtained from [this](https://sourceforge.net/projects/spandx/files/SPANDx%20Manual_v3.1.pdf/download) manual.

A snapshot of the pipeline is shown below:

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/spandx.jpg)

Because it takes a while to run, we have pre-run it for you. Your task will be to sort through the outputs of SPANDx. The detailed information about how to interpret the output is in SPANDx manual(section INTERPRETING THE OUTPUTS). 

> ***i. Look at overall statistics for variant calling in excel***

SPANDx produces an overall summary file of its run that includes:

1) numbers of SNPs/indels,

2) numbers of filtered SNPs/indels and 

3) average coverage across the reference genome. 

This summary file is in:  Outputs/Single_sample_summary.txt

Use less to look at this file and then apply unix commands to extract and sort individual columns 

**HINTS**
The following unix commands can be used to get sorted lists of coverage and numbers of SNPs/indels: tail, cut, sort

> ***ii. Look at filtered variants produced by SPANDx in excel***

SPANDx also produces a summary file of the variants/indels it identified in the core genome. 

This summary file is: 
```/scratch/micro612w21_class_root/micro612w21_class/username/day3pm/SPANDx_output/Outputs/All_SNPs_annotated.txt ```

Use cyberduck/scp to download this file and view in excel

- View SPANDx manual for interpretation of different columns which can be found [here](https://sourceforge.net/projects/spandx/files/SPANDx%20Manual_v3.1.pdf/download)
- Back on great lakes, use grep to pull SNPs that have HIGH impact
- What types of mutations are predicted to have “HIGH” impact?
- How many genomes do these HIGH impact mutations tend to be present in? How do you interpret this?

commenting out SPANDx and replacing it with snippy 2021-04-16

-->


Examine results of [Snippy](https://github.com/tseemann/snippy) pipeline
------------------------------------------------------------------------

On the afternoon of day 1 we saw how many steps are involved in calling variants relative to a reference genome. However, the same steps are applied to every sample, which makes this very pipeline friendly!  So, you could write your own shell script to string together these commands, or take advantage of one of several published pipelines. 

Here, we will use Snippy to perform rapid haploid variant calling and generate a core genome alignment. Snippy takes a tab delimited file containing list of paths to your fastq samples and a reference genome in Genbank format to generate a runme.sh shell script that can be run inside a SLURM script.   

We already created a runme.sh under snippy_results using snippy-multi command from snippy. Try exploring this shell script and understand what it means.

Every line of runme.sh contains a snippy command for each of your samples that are placed in - /nfs/esnitkin/Workshop_Backups/micro612w20_class/shared/data/day3pm_fastq/

Using runme.sh inside a slurm script - snippy.sbat, we already ran the pipeline for you since it takes ~2Hrs to run on 44 samples. The results of this run are located in snippy_results folder.

Snippy will create an individual folder Rush_KPC_* for each of the samples and their variant call results. The description of each of the output files are documented on Snippy's Github page - Section Calling SNPs -> Output Files

Since we are running Snippy pipeline on multiple samples, Snippy also produces an alignment of "core SNPs" which can be used to build a phylogeny. 

Snippy generates various other files with a prefix core.* that can be further used to explore variants/alignment statistics.

> ***i. Look at overall statistics for variant calling in excel***

Snippy produces an overall summary file - core.txt of its run that includes tab-separated list of alignment/core-size statistics such as:

1) Number of ALIGNED and UNALIGNED bases for a given sample.

2) Number of filtered VARIANT (variants passing all the filters)

3) Number of bases with low coverage - LOWCOV

Use less to look at this file and then apply unix commands to extract and sort individual columns 

**HINTS**
The following unix commands can be used to get sorted list of number of VARIANT and number of ALIGNED bases: tail, cut, sort

> ***ii. Add in another exercise here to explore either core.vcf or generate a file that looks like SPANDx's All_SNPs_annotated.txt***


Recombination detection and tree generation
-------------------------------------------

> ***i. Plot the distribution of variants across the genome in R***

The positions of variants are embedded in the second column of core.tab, but you have to do some work to isolate them! 

**HINTS**  

- You will need to use “cut” command to extract the second column "POS" from core.tab
- Note that for cut you can specify tab as the delimiter as follows: cut –d$’\t’
- You should redirect the output of your cut command (a list of SNP positions) to a file called ‘snp_positions.txt’. For example, the first line of your snp_positions.txt should be:

```
12695
```

- Finally, download this file, read it into R using ‘read.table’ and use ‘hist’ to plot a histogram of these core positions
- Do you observe clustering of variants that would be indicative of recombination? 


> ***ii. Create a neighboring-joining tree from core SNP alignment in R***

```

Download core.aln to your home computer
Read the fasta file into R, create a distance matrix, and make a neighbor joining tree. Make sure you load in the necessary packages! 

```

Phylogenetic tree annotation and visualization
----------------------------------------------

Follow along with the Day 3 morning exercise where we annotated a MRSA tree with CA vs. HA metadata to overlay facility information on your _Klebsiella_ neighbor-joining tree you just made. 

1. Read in annotation file called Rush_KPC_facility_codes.txt to R. This file is comma delimited. 

2. Drop tip labels from the tree that are not in the annotation file. Hint use ```setdiff()``` and ```drop.tip()```

3. Midpoint root the tree for visualization purposes using the function ```midpoint.root()```

```
my_tree = midpoint.root(my_tree) #where my_tree is what you named your neighbor-joining tree above 

```

4. Use ```sapply()``` to make your isolate_legend in the same order as your ```my_tree$tip.labels```. 

5. We provided color hex codes in Rush_KPC_facility_codes.txt so everyone's facilities are labeled with the same color. Use the following commands to extract the colors from the metadata and create your legend colors. 
```
colors = metadata[,c('Color', 'Facility')] # metadata = whatever your variable from reading in the annotation file from step 1 is called 
colors = colors[!duplicated(metadata[,c('Color', 'Facility')]),]
isolate_colors = structure(colors[,'Color'], names = colors[,'Facility'])
```

6. Use ```plot()```, ```tiplabels()``` and ```legend()``` to plot your tree with metadata overlayed as we did previously. 

To visualize the data on the tree better, you can use the argument ```show.tip.label = FALSE``` in ```plot()``` to remove the tree tip labels from the plot. You can also play around with the tree layout type  (e.g. phylogram, fan, etc) using the argument ```type```. You can change the placement of the colored tip labels by changing the number value of the parameter ```adj``` in ```tiplabels()```.  

After you've overlayed facility metadata on your tree, answer the following questions: 
```

Which facilities appear to have a lot of intra-facility transmission based on grouping of isolates from the same facility? 
Which patient’s infections might have originated from the blue facility?

```


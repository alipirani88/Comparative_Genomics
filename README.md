Microbial Comparative Genomics Workshop - 2021
==============================================

***A 3 day microbial bioinformatics workshop conducted by [Dr. Evan Snitkin](http://thesnitkinlab.com/index.php) at [University of Michigan](https://www.umich.edu/). This module covers the basics of microbial genomic analysis using publicly available tools that are commonly referenced in genomics literature. Students will learn the steps and associated tools that are required to process, annotate and compare microbial genomes.***

***Date: April 21-23 2021***
***
<!---
Link to Software Carpentry Etherpad:
http://pad.software-carpentry.org/micro612_bacterial_genomics_workshop
-->

Prerequisites
-------------

- Prior participation in a [Data Carpentry Workshop](https://umich-brcf-bioinf.github.io/2021-04-19-umich-computationalFoundations/site/Module00_Introduction)
***
<!---
- [Micro612 pre-course hw](https://github.com/alipirani88/Comparative_Genomics/blob/master/Micro612_pre-course_hw/Micro612_w18_pre-course_hw.pdf): A pre-course homework will help setting up Micro612 flux directories and bash profile.
-->

<!---
Link
----

GOTO: http://comparative-genomics.readthedocs.io/en/latest/index.html#
***

-->

***The Workshop directory on Great Lakes that you will be working in is - `/scratch/micro612w21_class_root/micro612w21_class/`***

You can find a folder named after your username/uniqname; for example if my uniqname is `apirani`, i will be working in `/scratch/micro612w21_class_root/micro612w21_class/apirani` This would be the directory that workshop attendees will be working from.

Workshop
--------

[Day 1 Morning](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_morning/README.md)
***
- [Installing and setting up Cyberduck for file transfer](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_morning/README.md#installing-and-setting-up-cyberduck-for-file-transfer)
- [Getting your data onto Flux and setting up Environment variable](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_morning/README.md#getting-your-data-onto-glux-and-setting-up-environment-variable)
- [Unix is your friend](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_morning/README.md#unix-is-your-friend)
- [Submit Variant Calling Job](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_morning/README.md#submit-variant-calling-job)

[Day 1 Afternoon](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#day-1-afternoon)
***
- [Contamination Screening using Kraken](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#contamination-screening-using-kraken)
- [Quality Control using FastQC](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#quality-control-using-fastqc)
- [Quality Trimming using Trimmomatic](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#quality-trimming-using-trimmomatic)
- [Read Mapping](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#read-mapping)
- [Variant Calling](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#variant-calling-and-filteration)
- [Visualize BAM/VCF files in Artemis](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#visualize-bam-and-vcf-files-in-artemis)
- [Exercise: Daptomycin resistance in VRE](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#exercise:-daptomycin-resistance-in-vre)
- [Exercise: Colistin resistance in Acinetobacter](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md#exercise:-colistin-resistance-in-acinetobacter)

[Day 2 Morning](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#day-2-morning)
***
- [Genome Assembly](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#genome-assembly)
- [Assembly evaluation](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#assembly-evaluation-using-quast)
- [Compare assembly to reference genome and Post-assembly genome improvement](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#compare-assembly-to-reference-genome-and-post-assembly-genome-improvement)
- [Genome Annotation](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#genome-annotation)
<!-- 
- [Map reads to the final ordered assembly](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_morning/README.md#map-reads-to-the-final-ordered-assembly)
-->

[Day 2 Afternoon](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_afternoon/README.md#day-2-afternoon)
***
- [Determine which genomes contain KPC genes using BLAST](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_afternoon/README.md#determine-which-genomes-contain-kpc-genes-using-blast)
- [Identify antibiotic resistance genes with ARIBA directly from paired end reads](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_afternoon/README.md#identify-antibiotic-resistance-genes-with-ariba-directly-from-paired-end-reads)
- [Perform pan-genome analysis with Roary](https://github.com/alipirani88/Comparative_Genomics/blob/master/day2_afternoon/README.md#perform-pan-genome-analysis-with-roary)

[Day 3 Morning](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#day-3-morning)
***
- [Perform whole genome alignment with Parsnp](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#perform-whole-genome-alignment-with-Parsnp-and-convert-alignment-to-other-useful-formats)
- [Perform DNA sequence comparisons and phylogenetic analysis in ape](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#perform-some-dna-sequence-comparisons-and-phylogenetic-analysis-in-ape)
- [Perform SNP density analysis to discern evidence of recombination](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#perform-snp-density-analysis-to-discern-evidence-of-recombination)
- [Perform recombination filtering with gubbins](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#perform-recombination-filtering-with-gubbins)
- [Overlay metadata on your tree using R ](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_morning/README.md#overlay-metadata-on-your-tree-using-r)

[Day 3 Afternoon](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#day-3-afternoon)
***
- [Perform QC on fastq files](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#perform-qc-on-fastq-files)
- [Run ARIBA CARD and MLST typing on day3pm_fastq samples](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#run-ariba-card-and-mlst-typing-on-day3pm_fastq-samples)
- [Examine results of Snippy pipeline](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#examine-results-of-snippy-pipeline)
- [Recombination detection and tree generation](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#recombination-detection-and-tree-generation)
- [Phylogenetic tree annotation and visualization](https://github.com/alipirani88/Comparative_Genomics/blob/master/day3_afternoon/README.md#phylogenetic-tree-annotation-and-visualization)



[Helpful resources for microbial genomics](https://github.com/alipirani88/Comparative_Genomics/blob/master/online_resources/README.md#helpful-resources-for-microbial-genomics)
***

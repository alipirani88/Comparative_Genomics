Day 1 Afternoon
===============
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

Earlier this morning, We performed some quality control steps on our sequencing data to make it clean and usable for various downstream analysis. Now we will perform our first sequence analysis, specifically variant calling, and map these reads to a reference genome and try to find out the differences between them.

Read Mapping is one of the most common Bioinformatics operations that needs to be carried out on NGS data. The main goal behind read mapping/aligning is to find the best possible reference genome position to which reads could be aligned. Reads are generally mapped to a reference genome sequence that is sufficiently closely related genome to accurately align reads. There are number of tools that can map reads to a reference genome and they differ from each other in algorithm, speed and accuracy. Most of these tools work by first building an index of reference sequence which works like a dictionary for fast search/lookup and then applying an alignment algorithm that uses these index to align short read sequences against the reference. 

These alignment has a vast number of uses, including: 

1) variant/SNP calling: Finding differences between your sequenced organism genome and the reference genome
2) coverage estimation: If you have sufficient reads to cover each position of reference genome.
3) gene expression analysis: determining the level of expression of each genes in a genome.

In this session, we will be covering the important steps that are part of any Read mapping/Variant calling bioinformatics pipleine.

Read Mapping
------------
[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/1_1.png)

**1. Navigate to your workshop home directory and copy day1_after directory from shared data directory.**

```
wd

cp -r /scratch/micro612w18_fluxod/shared/data/day1_after ./
```

We will be using trimmed clean reads that were obtained after running Trimmomatic on raw reads.

**2. Map your reads against a finished reference genome using [BWA](http://bio-bwa.sourceforge.net/bwa.shtml "BWA manual")**

Choosing the right read mapper is crucial and should be based on the type of analysis and data you are working with. Each aligners are meant to be better used with specific types of data, for example:

For whole genome or whole exome sequencing data: Use BWA for long reads (> 50/100 bp), use Bowtie2 for short reads (< 50/100bp)
For transcriptomic data (RNA-Seq): use Splice-aware Mapper such as Tophat. (Not applicable for microbial data)

Here, we will be using BWA aligner to map the reads against a reference genome, KPNIH1.

BWA is one of the several read mappers that are based on Burrows-Wheeler transform algorithm. If you feel like challenging yourselves, you can read BWA paper [here](http://bioinformatics.oxfordjournals.org/content/25/14/1754.short) 

Read Mapping is a time-consuming step that involves searching the reference and finding the optimal location for the alignment for millions of reads. Creating an index file of a reference sequence for quick lookup/search operations significantly decreases the time required for read alignment. Imagine indexing a genome sequence like the index at the end of a book. If you want to know on which page a word appears or a chapter begins, it is much more efficient to look it up in a pre-built index than going through every page of the book. Similarly, an index of a large DNA sequence allows aligners to rapidly find shorter sequences embedded within it. 

Note: each read mapper has its own unique way of indexing a reference genome and therefore the reference index created by BWA cannot be used for Bowtie. (Most Bioinformatics tools nowadays require some kind of indexing or reference database creation)

>i. To create BWA index of Reference, you need to run following command.

Start a flux interactive session

```
iflux
```


Navigate to day1_after folder that you recently copied and create a new folder Rush_KPC_266_varcall_result for saving this exercise's output.

```
d1a

#or 

cd /scratch/micro612w18_fluxod/username/day1_after/

mkdir Rush_KPC_266_varcall_result

```

Create bwa index for the reference genome. 

```
bwa index KPNIH1.fasta
```
 
Also go ahead and create fai index file using samtools required by GATK in later downstream steps.

```
samtools faidx KPNIH1.fasta
```

>ii. Align reads to reference and redirect the output into SAM file

Quoting BWA:
"BWA consists of three algorithms: BWA-backtrack, BWA-SW and BWA-MEM. The first algorithm is designed for Illumina sequence reads up to 100bp, while the rest two for longer sequences ranged from 70bp to 1Mbp. BWA-MEM and BWA-SW share similar features such as long-read support and split alignment, but BWA-MEM, which is the latest, is generally recommended for high-quality queries as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads."

For other algorithms employed by BWA, you can refer to BWA [manual](http://bio-bwa.sourceforge.net/bwa.shtml "BWA manual")

Now lets align both left and right end reads to our reference using BWA alignment algorithm 'mem'. 

```

bwa mem -M -R "@RG\tID:96\tSM:Rush_KPC_266_1_combine.fastq.gz\tLB:1\tPL:Illumina" -t 8 KPNIH1.fasta forward_paired.fq.gz reverse_paired.fq.gz > Rush_KPC_266_varcall_result/Rush_KPC_266__aln.sam

```

Read group tells aligners/other tools that certain reads were sequenced together on a specific lane. If you have multiplexed samples in a single lane, you will get multiple samples in a single read group. If you sequenced the same sample in several lanes, you will have multiple read groups for the same sample.

This string with -R flag says that all reads belongs to ID:96 and library LB:1; with sample name SM:Rush_KPC_266_1_combine.fastq.gz and was sequenced on illumina platform PL:Illumina.

You can extract this information from fastq read header. (@M02127:96:000000000-AG04W:1:1101:13648:1481 1:N:0:44)

**3. SAM/BAM manipulation and variant calling using [Samtools](http://www.htslib.org/doc/samtools.html "Samtools Manual")**

>i. Change directory to results folder and look for BWA output:

```
cd Rush_KPC_266_varcall_result

ls
```

The output of BWA and most of the short-reads aligners is a SAM file. SAM format is considered as the standard output for most read aligners and stands for Sequence Alignment/Map format. It is a TAB-delimited format that describes how each reads were aligned to the reference sequence. 

Lets explore first few lines of .sam file.

```

head -n4 Rush_KPC_266__aln.sam

```

example:

```

@SQ     SN:gi|661922017|gb|CP008827.1|  LN:5394056        <=== Reference Genome name and its length
@RG     ID:96   SM:Rush_KPC_266_1_combine.fastq.gz      LB:1    PL:Illumina <=== sample read group info
@PG     ID:bwa  PN:bwa  VN:0.7.12-r1039 CL:bwa mem -M -R @RG\tID:96\tSM:Rush_KPC_266_1_combine.fastq.gz\tLB:1\tPL:Illumina -t 8 KPNIH1.fasta forward_paired.fq.gz reverse_paired.fq.gz       <== aligner command 
M02127:96:000000000-AG04W:1:1101:23094:1725     99      gi|661922017|gb|CP008827.1|     4724728 60      250M    =       4724852 295     GCTGCCTGCAGCATCTCAGCGGCTTTATCGGCTCGCAGCAGGTGCGGCTGGTGACCCTCTCCGGCGGCGTCGGCCCGTATATGACCGGTATCGGCCAGCTTGATGCCGCCTGCAGCGTCAGCATTATCCCGGCGCCGCTGCGGGTCTCTTCGGCGGAGGTCTCCGAGATCCTGCGCCGCGAGTCGAGCGTGCGCGACGTGATCCTCGCGGCGACGGCGGCGGACGCGGCGGTAGTCGGCCTTGGCGCCAT      CCCCCGGGGGGGGGGEGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGFGGGGG@FGFGFFGGGGGGGGGGGGCFGBEGGGCFGGGFGDE>*CGEFCCFCEECCCCGGGDGE5E>5EEFEEGD=C=EDCE=EEECCC?C9CCECEDC<DGGGGCDGG:CBC)<DB>@EF??>>@)7<6?6354,4      NM:i:2  MD:Z:161G77A10  AS:i:240        XS:i:0  RG:Z:96

```

The lines starting with "@" is a header section and contains information about reference genome, sample read group and the aligner command that was used for aligning the samples. The header section is followed by an alignment section information for each read. It contains 11 columns and an optional TAG option.

Detailed information about these 11 columns can be obtained from this [pdf](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwizkvfAk9rLAhXrm4MKHVXxC9kQFggdMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FSAMv1.pdf&usg=AFQjCNHFmjxTXKnxYqN0WpIFjZNylwPm0Q) document.

The second column consists of coded bitwise flags where each code flag carries important information about the alignment. Open [this](https://broadinstitute.github.io/picard/explain-flags.html) site and enter the flag "99" to find out what it stands for.

The last section "NM:i:2  MD:Z:161G77A10  AS:i:240 XS:i:0  RG:Z:96" is an optional tag section and varies for different aligners(specifications based on aligners). 

Here, 

NM tag tells number of changes necessary to make it equal to the reference(2 changes)

MD tag tells you what positions in the read alignment are different from reference base and is used by variant callers to call SNP's. For example, The tag "MD:Z:161G77A10" implies that position 162 in the read carries a different base whereas the reference genome carries base "G"

AS is an alignment score and XS:i:0 is an suboptimal alignment score.

>ii. Convert SAM to BAM using SAMTOOLS:

BAM is the compressed binary equivalent of SAM but are usually quite smaller in size than SAM format. Since, parsing through a SAM format is slow, Most of the downstream tools require SAM file to be converted to BAM so that it can be easily sorted and indexed.

The below command will ask samtools to convert SAM format(-S) to BAM format(-b)

```
samtools view -Sb Rush_KPC_266__aln.sam > Rush_KPC_266__aln.bam
```

>iii. Sort BAM file using SAMTOOLS:

Most of the downstream tools such as GATK requires your BAM file to be indexed and sorted by reference genome positions.

Now before indexing this BAM file, we will sort the data by positions(default) using samtools. Some RNA Seq/Gene expression tools require it to be sorted by read name which is achieved by passing -n flag.

```
samtools sort Rush_KPC_266__aln.bam -o Rush_KPC_266__aln_sort
```

**4. Mark duplicates(PCR optical duplicates) and remove them using [PICARD](http://broadinstitute.github.io/picard/command-line-overview.html#MarkDuplicates "Picard MarkDuplicates")**

Illumina sequencing involves PCR amplification of adapter ligated DNA fragments so that we have enough starting material for sequencing. Therefore, some amount of duplicates are inevitable. Ideally, you amplify upto ~65 fold(4% reads) but higher rates of PCR duplicates e.g. 30% arise when people have too little starting material such that greater amplification of the library is needed or some smaller fragments which are easier to PCR amplify, end up over-represented.

For an in-depth explanation about how PCR duplicates arise in sequencing, please refer to this interesting [blog](http://www.cureffi.org/2012/12/11/how-pcr-duplicates-arise-in-next-generation-sequencing/)

Picard identifies duplicates by searching reads that have same start position on reference or in PE reads same start for both ends. It will choose a representative from each group of duplicate reads based on best base quality scores and other criteria and retain it while removing other duplicates. This step plays a significant role in removing false positive variant calls(such as sequencing error) during variant calling that are represented by PCR duplicate reads.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/picard.png)

>i. Create a dictionary for reference fasta file required by PICARD

Make sure you are in Rush_KPC_266_varcall_result directory and are giving proper reference genome path (day1_after directory).

```

java -jar /scratch/micro612w18_fluxod/shared/bin/picard-tools-1.130/picard.jar CreateSequenceDictionary REFERENCE=../KPNIH1.fasta OUTPUT=../KPNIH1.dict

```

>ii. Run PICARD for removing duplicates.

```

java -jar /scratch/micro612w18_fluxod/shared/bin/picard-tools-1.130/picard.jar MarkDuplicates REMOVE_DUPLICATES=true INPUT=Rush_KPC_266__aln_sort.bam OUTPUT=Rush_KPC_266__aln_marked.bam METRICS_FILE=Rush_KPC_266__markduplicates_metrics CREATE_INDEX=true VALIDATION_STRINGENCY=LENIENT

```

The output of Picard remove duplicate step is a new bam file "Rush_KPC_266__aln_marked.bam" without PCR duplicates.

You will need to index this new marked.bam file for further processing.

>iii. Index these marked bam file again using SAMTOOLS(For input in Artemis later)

```
samtools index Rush_KPC_266__aln_marked.bam
```

Open the markduplicates metrics file and glance through the number and percentage of PCR duplicates removed. 
For more details about each metrics in a metrics file, please refer [this](https://broadinstitute.github.io/picard/picard-metric-definitions.html#DuplicationMetrics)

```
nano Rush_KPC_266__markduplicates_metrics

#or 

less Rush_KPC_266__markduplicates_metrics
```

Generate Alignment Statistics
-----------------------------

Often, While analyzing sequencing data, we are required to make sure that our analysis steps are correct. Some statistics about our analysis will help us in making that decision. So Lets try to get some statistics about various outputs that were created using the above steps and check if everything makes sense.

>i. Collect Alignment statistics using Picard

Run the below command on your marked.bam file

```

java -jar /scratch/micro612w18_fluxod/shared/bin/picard-tools-1.130/picard.jar CollectAlignmentSummaryMetrics R=../KPNIH1.fasta I=Rush_KPC_266__aln_marked.bam O=AlignmentSummaryMetrics.txt

```
Open the file AlignmentSummaryMetrics.txt and explore various statistics. It will generate various statistics and the definition for each statistic s can be found [here](http://broadinstitute.github.io/picard/picard-metric-definitions.html#AlignmentSummaryMetrics)

> Question: Extract alignment percentage from AlignmentSummaryMetrics file. (% of reads aligned to reference genome)

<!---
awk -F'\t' '{print $7}' AlignmentSummaryMetrics.txt
-->

```
grep -v '#' AlignmentSummaryMetrics.txt | cut -f7
```

>ii. Estimate read coverage/read depth using Picard

Read coverage/depth describes the average number of reads that align to, or "cover," known reference bases.

```
java -jar /scratch/micro612w18_fluxod/shared/bin/picard-tools-1.130/picard.jar CollectWgsMetrics R=../KPNIH1.fasta I=Rush_KPC_266__aln_marked.bam O=WgsMetrics.txt

```

Open the file WgsMetrics.txt and explore various statistics. It will generate various statistics and the definition for each can be found [here](https://broadinstitute.github.io/picard/picard-metric-definitions.html#CollectWgsMetrics.WgsMetrics)

> Question: Extract mean coverage information from WgsMetrics.txt
<!---
sed -n 7,8p WgsMetrics.txt | awk -F'\t' '{print $2}'
-->

```
grep -v '#' WgsMetrics.txt | cut -f2 | head -n3
```
<!--
## Generate Alignment Statistics report using [Qualimap](http://qualimap.bioinfo.cipf.es/)
Qualimap outputs a very informative report about the alignments and coverage across the entire genome. Lets create one for our sample. The below command calls bamqc utility of qualimap and generates a report in pdf format.
``` 
qualimap bamqc -bam Rush_KPC_266__aln_sort.bam -outdir ./ -outfile Rush_KPC_266__report.pdf -outformat pdf 
```
Lets get this pdf report onto our local system and check the chromosome stats table, mapping quality and coverage across the entire reference genome.
```
scp username@flux-xfer.arc-ts.umich.edu:/scratch/micro612w18_fluxod/username/day1_after/Rush_KPC_266_varcall_result/Rush_KPC_266__report.pdf /path-to-local-directory/
```
-->

Variant Calling and Filteration
-------------------------------
[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

One of the downstream uses of read mapping is finding differences between our sequence data against a reference. This step is achieved by carrying out variants calling using any of the variant callers(samtools, gatk, freebayes etc). Each variant caller uses a different statistical framework to discover SNPs and other types of mutations. For those of you who are interested in finding out more about the statistics involved, please refer to [this]() samtools paper, one of most commonly used variant callers.

This GATK best practices [guide](https://www.broadinstitute.org/gatk/guide/best-practices.php) will provide more details about various steps that you can incorporate in your analysis.

There are many published articles that compares different variant callers but this is a very interesting [blog](https://bcbio.wordpress.com/2013/10/21/updated-comparison-of-variant-detection-methods-ensemble-freebayes-and-minimal-bam-preparation-pipelines/) that compares the performance and accuracy of different variant callers.

Here we will use samtools mpileup to perform this operation on our BAM file and generate VCF file. 

**1. Call variants using [samtools](http://www.htslib.org/doc/samtools.html "samtools manual") mpileup and [bcftools](https://samtools.github.io/bcftools/bcftools.html "bcftools")**

```

samtools mpileup -ug -f ../KPNIH1.fasta Rush_KPC_266__aln_marked.bam | bcftools call -O v -v -c -o Rush_KPC_266__aln_mpileup_raw.vcf


#In the above command, we are using samtools mpileup to generate a pileup formatted file from BAM alignments and genotype likelihoods(-g flag) in BCF format(binary version of vcf). This bcf output is then piped to bcftools, which calls variants and outputs them in vcf format(-c flag for using consensus calling algorithm  and -v for outputting variants positions only)


```

Lets go through an the vcf file and try to understand a few important vcf specifications and criteria that we can use for filtering low confidence snps. 

```
less Rush_KPC_266__aln_mpileup_raw.vcf
```

Press 'q' from keyboard to exit.

VCF format stores a large variety of information and you can find more details about each nomenclature in this [pdf](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwit35bvktzLAhVHkoMKHe3hAhYQFggdMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FVCFv4.2.pdf&usg=AFQjCNGFka33WgRmvOfOfp4nSaCzkV95HA&sig2=tPLD6jW5ALombN3ALRiCZg&cad=rja)

**2. Variant filtering and processed file generation using GATK and vcftools**

>i. Variant filtering using [GATK](https://www.broadinstitute.org/gatk/guide/tooldocs/org_broadinstitute_gatk_tools_walkers_filters_VariantFiltration.php "GATK Variant Filteration"):

There are various tools that can you can try for variant filteration such as vcftools, GATK, vcfutils etc. Here we will use GATK VariantFiltration utility to filter out low confidence variants.

Run this command on raw vcf file Rush_KPC_266__aln_mpileup_raw.vcf.

```

java -jar /scratch/micro612w18_fluxod/shared/bin/GenomeAnalysisTK-3.3-0/GenomeAnalysisTK.jar -T VariantFiltration -R ../KPNIH1.fasta -o Rush_KPC_266__filter_gatk.vcf --variant Rush_KPC_266__aln_mpileup_raw.vcf --filterExpression "FQ < 0.025 && MQ > 50 && QUAL > 100 && DP > 15" --filterName pass_filter

```

This command will add a 'pass_filter' text in the 7th FILTER column for those variant positions that passed our filtered criteria:

1. DP: Depth of reads. More than 15 reads supporting a variant call at these position.
2. MQ: Root Mean Square Mapping Quality. This provides an estimation of the overall mapping quality of reads supporting a variant call. The root mean square is equivalent to the mean of the mapping qualities plus the standard deviation of the mapping qualities.
3. QUAL stands for phred-scaled quality score for the assertion made in ALT. High QUAL scores indicate high confidence calls.
4. FQ stands for consensus quality. A positive value indicates heterozygote and a negative value indicates homozygous. In bacterial analysis, this plays an important role in defining if a gene was duplicated in a particular sample. We will learn more about this later while visualizing our BAM files in Artemis.

Check if the pass_filter was added properly.

```
grep 'pass_filter' Rush_KPC_266__filter_gatk.vcf | head
```

caveat: These filter criteria should be applied carefully after giving some thought to the type of library, coverage, average mapping quality, type of analysis and other such requirements.

>ii. Remove indels and keep only SNPS that passed our filter criteria using [vcftools](http://vcftools.sourceforge.net/man_latest.html vcftools manual):

vcftools is a program package that is especially written to work with vcf file formats. It thus saves your precious time by making available all the common operations that you would like to perform on vcf file using a single command. One such operation is removing INDEL infromation from a vcf file.

Now, Lets remove indels from our final vcf file and keep only variants that passed our filter criteria(positions with pass_filter in their FILTER column).

```

vcftools --vcf Rush_KPC_266__filter_gatk.vcf --keep-filtered pass_filter --remove-indels --recode --recode-INFO-all --out Rush_KPC_266__filter_onlysnp

```

<!--
commenting out consensus generation
>iii. Generate Consensus fasta file from filtered variants using vcftools:
A consensus fasta sequence will contain alleles from reference sequence at positions where no variants were observed and variants that were observed at positions described in vcf file.
Run the commands below to generate a consensus fasta sequence.
```
bgzip Rush_KPC_266__filter_onlysnp.recode.vcf
tabix Rush_KPC_266__filter_onlysnp.recode.vcf.gz
cat ../KPNIH1.fasta | vcf-consensus Rush_KPC_266__filter_onlysnp.recode.vcf.gz > Rush_KPC_266__consensus.fa
```
> Note: Dont forget to put the actual path to the refeerence sequence in place of /path-to-reference/
Check the fasta header and change it using sed.
```
head -n1 Rush_KPC_266__consensus.fa
sed -i 's/>.*/>Rush_KPC_266_/g' Rush_KPC_266__consensus.fa 
```
-->

**3. Variant Annotation using snpEff**

Variant annotation is one of the crucial steps in any variant calling pipeline. Most of the variant annotation tools creates their own database or use an external one to assign function and predict the effect of variants on genes. We will try to touch base on some basic steps of annotating variants in our vcf file using snpEff. 

You can annotate these variants before performing any filtering steps that we did earlier or you can decide to annotate just the final filtered variants. 

snpEff contains database of about 20000 reference genome built from trusted and public sources. Lets check if snpEff contains a database of our reference genome.

>i. Check snpEff internal database for your reference genome:

```     
java -jar /scratch/micro612w18_fluxod/shared/bin/snpEff/snpEff.jar databases | grep 'kpnih1'
```
Note down the genome id for your reference genome KPNIH1. In this case: GCA_000281535.2.29

>ii. Change the chromosome name in vcf file to ‘Chromosome’ for snpEff reference database compatibility. 

```
sed -i 's/gi.*|/Chromosome/g' Rush_KPC_266__filter_gatk.vcf
```
>iii. Run snpEff for variant annotation.

```

java -jar /scratch/micro612w18_fluxod/shared/bin/snpEff/snpEff.jar -onlyProtein -no-upstream -no-downstream  -no-intergenic -v GCA_000281535.2.29 Rush_KPC_266__filter_gatk.vcf > Rush_KPC_266__filter_gatk_ann.vcf -csvStats Rush_KPC_266__filter_gatk_stats

```

The STDOUT  will print out some useful details such as genome name and version being used, no. of genes, protein-coding genes and transcripts, chromosome and plasmid names etc

Lets go through the ANN field added after annotation step.

```
grep 'ANN=' Rush_KPC_266__filter_gatk_ann.vcf | head -n1
```

ANN field will provide information such as the impact of variants (HIGH/LOW/MODERATE/MODIFIER) on genes and transcripts along with other useful annotations.

Detailed information of ANN field and sequence ontology terms that it uses can be found [here](http://snpeff.sourceforge.net/SnpEff_manual.html#input)

Lets see how many SNPs and Indels passed the filter using grep and wc

```

No. of Variants:
grep '^Chromosome' Rush_KPC_266__filter_gatk_ann.vcf | wc -l

No. of Variants that passed the filter:
grep '^Chromosome.*pass_filter' Rush_KPC_266__filter_gatk_ann.vcf | wc -l

No. of SNPs that passed the filter:
grep '^Chromosome.*pass_filter' Rush_KPC_266__filter_gatk_ann.vcf | grep -v 'INDEL' | wc -l

No. of Indels that passed the filter:
grep '^Chromosome.*pass_filter' Rush_KPC_266__filter_gatk_ann.vcf | grep 'INDEL' | wc -l


```

Visualize BAM and VCF files in [Artemis](http://www.sanger.ac.uk/science/tools/artemis)
----------------------------------------
[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

While these various statistical/text analyses are helpful, visualization of all of these various output files can help in making some significant decisions and inferences about your entire analysis. There are a wide variety of visualization tools out there that you can choose from for this purpose.

We will be using [Artemis](http://www.sanger.ac.uk/science/tools/artemis) here, developed by Sanger Institute for viewing BAM and vcf files for manual inspection of some of the variants.


> Required Input files: 
KPNIH1 reference fasta and genbank file, 
Rush_KPC_266__aln_marked.bam and Rush_KPC_266__aln_marked.bam.bai, 
Rush_KPC_266__filter_gatk_ann.vcf.gz and Rush_KPC_266__filter_gatk_ann.vcf.gz.tbi

Lets make a seperate folder(make sure you are in Rush_KPC_266_varcall_result folder) for the files that we need for visualization and copy it to that folder

```

mkdir Artemis_files

cp ../KPNIH1.fasta ../KPNIH.gb Rush_KPC_266__aln_marked.bam Rush_KPC_266__aln_marked.bam.bai Rush_KPC_266__filter_gatk_ann.vcf Artemis_files/

```

We need to replace the genome name that we changed earlier for snpEff. (Make sure you are in Artemis_files folder)

```

cd Artemis_files

sed -i 's/Chromosome/gi|661922017|gb|CP008827.1|/g' Rush_KPC_266__filter_gatk_ann.vcf 

bgzip Rush_KPC_266__filter_gatk_ann.vcf

tabix Rush_KPC_266__filter_gatk_ann.vcf.gz
```

Open a new terminal and run scp commands or cyberduck to get these files to your local system.

```

scp -r username@flux-xfer.arc-ts.umich.edu:/scratch/micro612w18_fluxod/username/day1_after/Rush_KPC_266_varcall_result/Artemis_files/ /path-to-local-directory/

#You can use ~/Desktop/ as your local directory path
```

start Artemis.

Set your working directory to Artemis_files(The Artemis_files folder that you copied to your local system) by clicking at browse button  and click OK.

Now go to the top left File options and select Open File Manager. You should see the folder Artemis_files. Expand it and select KPNIH.gb file. A new window should open displaying your features stored in a genbank file.

Now open BAM file by selecting File(Top left corner) -> Read BAM/VCF file -> Select -> Rush_KPC_266__aln_marked.bam -> OK

Reads aligned to your reference are displayed as stacked at the top panel of Artemis. The reads are colour coded so that paired reads are blue and those with an inversion are red. Reads that do not have a mapped mate are black and are optionally shown in the inferred insert size view. In the stack view, duplicated reads that span the same region are collapsed into one green line.

Now right click on any of the stacked reads and Go to Graph and select Coverage(screenshot below). 

Now right click on any of the stacked reads and Go to Show and select SNP marks to show SNP's in red marks. 

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/select_graph.png)

Follow the same procedure and select SNP graph. Adjust the gene features panel height to show all the graph in a window.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/graphs.png)

Play around by moving the genbank panel cursor to look at coverage and SNP density across the genome. This will let you look at any regions where the coverage or SNP density is unusually high or low.

If you click a read, its mate pair will also be selected. If the cursor hovers over a read for long enough details of that read will appear in a small box. For more details of the read, right-click and select 'Show details of: READ NAME' (last option in list) from the
menu.(screenshot below) This will open up a new window giving you some useful details such as mapping quality, coordinates etc. 

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/read_details.png)

The snps are denoted by red marks as observed inside the reads. Go to one of the SNPs in VCF file(Position: 50195) by directly navigating to the position. For this, select Goto at the top -> select Navigator -> Type the position in Goto Base box

You will Notice a spike in the middle of the SNP graph window. This is one of the SNPs that passed all our filter criteria. (Screenshot)

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/spike_true.png)

Lets try to see an example of HET variant. Variant positions where more than one allele(variants) with suffficiently high read depth are observed are considered as HET type variant. 

For this, click on Goto option at the top and select navigator. Type 321818 in Goto Base box and click Goto.

You will see a thick spike in the SNP graph as well as thick red vertical line in BAM panel. Also notice the sudden spike in the coverage for this particular region compared to its flanking region(Region before and after a selected region). The coverage here is more than 300 which is unusually high compared to the entire genome coverage. This means that more than one allele with high quality and depth were observed at these positions so we cannot decide which one of these is a true variant. We removed these types of variants during our Variant Filteration step using the criteria FQ. (If the FQ is unusually high, it is suggestive of HET variant and negative FQ value is a suggestive of true variant as observed in the mapped reads) 

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/HET_variant.png)

Now select the gene right below this spiked region. Right click on this gene(KPNIH1_RS01560) and select Zoom to Selection.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/HET_variant_gene_selected.png)

Check the details about gene by selecting View -> Selected Features

You can inspect these type of HET variants later for any gene duplication or copy number analysis (by extracting variant positions with high FQ values). Addition of these details will give a better resolution while inferring Phylogenetic trees.

Play around with Artemis to look at what other kind of information you can find from these BAM and vcf files. Also refer to the manual at Artemis [Homepage](http://www.sanger.ac.uk/science/tools/artemis) for full information about its usage. 

[[back to top]](https://github.com/alipirani88/Comparative_Genomics/blob/master/day1_afternoon/README.md)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

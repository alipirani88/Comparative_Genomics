# Day 1 Afternoon
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

Read Mapping is one of the most common Bioinformatics operation that needs to be carried out on NGS data. Reads are generally mapped to a reference genome sequence or closely related genome if reference is not available. There are number of tools that can map reads to a reference genome and differ from each other in algorithm, speed and accuracy. Most of these tools works by first building an index of reference sequence which works like a dictionary for fast search/lookup and then calling an alignment algorithm that uses these index to align short read sequences against the reference. These alignment has a vast number of uses ranging from Variant/SNP calling, Coverage estimation and gene expression analysis.

## Read Mapping
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)
![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/1.png)

**1. Copy day1_after directory from shared data directory into your home directory.**

```
cp -r /scratch/micro612w16_fluxod/shared/data/day1_after ./
```

We will be using the trimmed clean reads that were obtained after running Trimmmatic on raw reads.

**2. Map your reads against a finished reference genome using [BWA](http://bio-bwa.sourceforge.net/bwa.shtml "BWA manual")**

BWA is one of the several and a very good example of read mappers that are based on Burrows-Wheeler transform algorithm. If you feel like challenging yourselves, you can read BWA paper [here](http://bioinformatics.oxfordjournals.org/content/25/14/1754.short) 

Read Mapping is a time-consuming step that involves searching the reference and aligning millions of reads. Creating an index file of reference sequence for quick lookup/search operations significantly decreases the time required for read alignment.

>i. To create BWA index of Reference, you need to run following command.

Our reference genome is located at: /scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta
Copy it to day1_after folder and create Rush_KPC_266_varcall_result folder for saving this exercise's output.

```
cp /scratch/micro612w16_fluxod/shared/bin/reference/KPNIH1/KPNIH1.fasta day1_after/
cd day1_after/
mkdir Rush_KPC_266_varcall_result
```

Create bwa index for the reference genome.
```
bwa index KPNIH1.fasta
```

Also go ahead and create fai index file using samtools required by GATK in downstream steps.

```
samtools faidx KPNIH1.fasta
```

>ii. Align reads to reference and output into SAM file

Now lets align both left and right end reads to our reference using BWA alignment algorithm 'mem' which is one of the three algorithms that is fast and works on mate paired end reads. 
For other algorithms employed by BWA, you can refer to BWA [manual](http://bio-bwa.sourceforge.net/bwa.shtml "BWA manual")

```
bwa mem -M -R "@RG\tID:96\tSM:Rush_KPC_266_1_combine.fastq.gz\tLB:1\tPL:Illumina" -t 8 KPNIH1.fasta forward_paired.fq.gz reverse_paired.fq.gz > Rush_KPC_266__aln.sam
```

Many algorithms need to know that certain reads were sequenced together on a specific lane. This string with -R flag says that all reads belongs to ID 96 sample Rush_KPC_266_1_combine.fastq.gz and was sequenced on illumina platform.

**3. SAM/BAM manipulation and variant calling using [Samtools](http://www.htslib.org/doc/samtools.html "Samtools Manual")**

>i. Change directory to results folder:

```
cd Rush_KPC_266_varcall_result
```

>ii. Convert SAM to BAM using SAMTOOLS:

SAM stands for sequence alignment/map format and is a TAB-delimited text format that describes how each reads were aligned to the reference sequence. For detailed specifications of SAM format fields, Please  read this [pdf](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwizkvfAk9rLAhXrm4MKHVXxC9kQFggdMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FSAMv1.pdf&usg=AFQjCNHFmjxTXKnxYqN0WpIFjZNylwPm0Q) document.

BAM is the compressed binary equivalent of SAM but are usually quite smaller in size than SAM format. Since, parsing through a SAM format is slow, Most of the downstream tools requires SAM to be converted to BAM so that it can be easily sorted and indexed.

The below command will ask samtools to convert SAM format(-S) to BAM format(-b)
```
samtools view -Sb Rush_KPC_266__aln.sam > Rush_KPC_266__aln.bam
```

>iii. Sort BAM file using SAMTOOLS:

Now before indexing this BAM file, we will sort the data by positions(default) using samtools. Some expression tools requires it to be sorted by read name which is achieved by passing -n flag.

```
samtools sort Rush_KPC_266__aln.bam Rush_KPC_266__aln_sort
```

**4. Mark duplicates(PCR optical duplicates) and remove them using [PICARD](http://broadinstitute.github.io/picard/command-line-overview.html#MarkDuplicates "Picard MarkDuplicates")**

Illumina sequencing involves PCR amplification of adapter ligated DNA fragments so that we have enough starting material for sequencing. Therefore, some amount of duplicates are inevitable. Ideally, you amplify upto ~65 fold(4% reads) but higher rates of PCR duplicates e.g. 30% arise when people have too little starting material such that greater amplification of the library is needed or some smaller fragments which are easier to PCR amplify, end up over-represented.

For an in-depth explanation about how PCR duplicates arise in sequencing, please refer to this interesting [blog](http://www.cureffi.org/2012/12/11/how-pcr-duplicates-arise-in-next-generation-sequencing/)

Picard identifies duplicates by search for reads that have same start position on reference or in PE reads same start for both ends. It will choose a representative from the based on base quality scores and other criteria and retain it while removing other duplicates. This step is plays a significant role in removing false positive variant calls represented by PCR duplicate reads.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1_after/picard.png)

>i. Create a dictionary for reference fasta file required by PICARD
 
```
java -jar /scratch/micro612w16_fluxod/shared/bin/picard-tools-1.130/picard.jar CreateSequenceDictionary REFERENCE=/path-to-reference/KPNIH1.fasta OUTPUT=/path-to-reference/KPNIH1.dict
```

> Note: Dont forget to put the actual path to the refeerence sequence in place of /path-to-reference/ and also keep KPNIH1.dict as output filename in above command.

>ii. Run PICARD for removing duplicates.

```
java -jar /scratch/micro612w16_fluxod/shared/bin/picard-tools-1.130/picard.jar MarkDuplicates REMOVE_DUPLICATES=true INPUT=Rush_KPC_266__aln_sort.bam OUTPUT= Rush_KPC_266__aln_marked.bam METRICS_FILE=Rush_KPC_266__markduplicates_metrics
CREATE_INDEX=true VALIDATION_STRINGENCY=LENIENT
```
Open the markduplicates metrics file and glance through the number and percentage of PCR duplicates removed. For more details about each metrics in a metrics file, please refer [this](https://broadinstitute.github.io/picard/picard-metric-definitions.html#DuplicationMetrics)

```
nano Rush_KPC_266__markduplicates_metrics
```
>iii. Sort these marked BAM file again (Just to make sure, it doesn't throw error in downstream steps)

```
samtools sort Rush_KPC_266__aln_marked.bam Rush_KPC_266__aln_sort
```

>iv. Index these marked bam file again using SAMTOOLS (Just to make sure, it doesn't throw error in downstream steps)

```
samtools index Rush_KPC_266__aln_marked.bam
```

## Variant Calling and Filteration
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)
One of the downstream uses of read mapping is finding differences between our sequence data against a reference. This step is achieve by carrying out calling variants using any of the variant callers(samtools, gatk, freebayes etc). Each variant callers use different statistical framework to discover SNP and other types of mutations. For those of you who are interested in finding out more about the statistics involved, please refere to [this]() samtools paper, one of most commonly used variant caller.

This GATK best practices [guide](https://www.broadinstitute.org/gatk/guide/best-practices.php) will provide more details about various steps that you can incorporate in your analysis.

There are many published articles that compares different variant callers but this is a very interesting [blog](https://bcbio.wordpress.com/2013/10/21/updated-comparison-of-variant-detection-methods-ensemble-freebayes-and-minimal-bam-preparation-pipelines/) that compares the performance and accuracy of different variant callers.

Here we will use samtools mpileup to perform this operation on our BAM file and generate VCF file. 

**1. Call variants using [samtools](http://www.htslib.org/doc/samtools.html "samtools manual") mpileup and [bcftools](https://samtools.github.io/bcftools/bcftools.html "bcftools")**

samtools mpileup generate pileup format from alignments, computes genotype likelihood(-ug flag) and outputs it in bcf format(binary version of vcf). This bcf output is then piped to bcftools that calls variants and outputs it in vcf format(-c flag for consensus calling and -v for outputting variants positions only)

```
samtools mpileup -ug -f /path-to-reference/KPNIH1.fasta Rush_KPC_266__aln_marked.bam | bcftools call -O v -v -c -o Rush_KPC_266__aln_mpileup_raw.vcf
```

> Note: Dont forget to put the actual path to the reference sequence in place of /path-to-reference/

Lets go through our vcf file and try to understand a few vcf specifications and criteria that we can use for filtering low confidence snps. 

```
less Rush_KPC_266__aln_mpileup_raw.vcf
```

VCF format stores a large variety of information and you can find more details about each nomenclature in this [pdf](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwit35bvktzLAhVHkoMKHe3hAhYQFggdMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FVCFv4.2.pdf&usg=AFQjCNGFka33WgRmvOfOfp4nSaCzkV95HA&sig2=tPLD6jW5ALombN3ALRiCZg&cad=rja)

**2. Variant filtering and processed file generation using GATK and vcftools**

>i. Variant filtering using [GATK](https://www.broadinstitute.org/gatk/guide/tooldocs/org_broadinstitute_gatk_tools_walkers_filters_VariantFiltration.php "GATK Variant Filteration"):

There are various tools that can you can try for variant filteration such as vcftools, GATK, vcfutils. Here we will use GATK VariantFiltration utility to filter out low confidence variants.

Here, the below command will add a 'pass_filter' text in the 7th FILTER column for those variant positions that passed our filtered criteria:
1. DP: Depth of reads. More than 15 reads supporting a variant call at these position.
2. MQ: Root Mean Square Mapping Quality. This provides an estimation of the overall mapping quality of reads supporting a variant call. The root mean square is equivalent to the mean of the mapping qualities plus the standard deviation of the mapping qualities.
3. QUAL stands for phred-scaled quality score for the assertion made in ALT. High QUAL scores indicate high confidence calls.
4. FQ stands for consensus quality. A positive value indicates heterozygote and a negative value indicates homozygous. In bacterial analysis, this plays an important role in defining if a genes was duplicated in a particular sample. We will more about later while visualizing our BAM files in IGV.

Run this command on raw vcf file Rush_KPC_266__aln_mpileup_raw.vcf.

```
java -jar /scratch/micro612w16_fluxod/shared/bin/GenomeAnalysisTK-3.3-0/GenomeAnalysisTK.jar -T VariantFiltration -R
/path-to-reference/KPNIH1.fasta -o Rush_KPC_266__filter_gatk.vcf --variant Rush_KPC_266__aln_mpileup_raw.vcf --filterExpression "FQ < 0.025 && MQ > 50 && QUAL > 100 && DP > 15" --filterName pass_filter
```

> Note: Dont forget to put the actual path to the refeerence sequence in place of /path-to-reference/

Lets look at some of the filtered positions.

```
grep 'pass_filter' Rush_KPC_266__filter_gatk.vcf | head
```
caveat: These filter criterias should be applied carefully after giving some thoughts based on the type of library, coverage, average mapping quality, type of analysis and other such requirements.

More Info on VCF format and parameter specifications can be found [here](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjzkcSP4MfLAhUDyYMKHU3yDwMQFggjMAA&url=https%3A%2F%2Fsamtools.github.io%2Fhts-specs%2FVCFv4.2.pdf&usg=AFQjCNGFka33WgRmvOfOfp4nSaCzkV95HA&sig2=6Xb3XDaZfghadZfcnnPQxw&cad=rja "VCF format Specs.")

>ii. Remove indels and keep only SNPS that passed our filter criteria using [vcftools](http://vcftools.sourceforge.net/man_latest.html vcftools manual):

In most of the phylogenetic analysis, we are trying to find how these samples differ and evolve from the reference/index genome. Many tools that carry out such type of phylogenetic analysis requires a consensus sequences containing only variant calls(SNPs). Though there are few tools that take into consideration both SNPs and Indels and give a greater resolution.
Now we will try to construct a consensus sequence using only SNP calls.

vcftools is a program package that is especially written to work with vcf file formats. It thus saves your precious time by making available all the common operations that you would like to perfome on vcf file using a single command.

Now, Lets remove indels from our final vcf file and keep only variants that passed our filter criteria(positions with pass_filter in their FILTER column).

```
vcftools --vcf Rush_KPC_266__filter_gatk.vcf --keep-filtered pass_filter --remove-indels --recode --recode-INFO-all --out
Rush_KPC_266__filter_onlysnp 
```

Notice the details that were printed out in STDOUT.(How many sites were retained out of total site?)

>iii. Generate Consensus fasta file from filtered variants using vcftools:

A consensus fasta sequence will contain alleles from reference sequence at positions where no variants were observed and variants that were observed at positions described in vcf file.

Run the commands below generate a consensus fasta sequence.

```
bgzip Rush_KPC_266__filter_onlysnp.recode.vcf
tabix Rush_KPC_266__filter_onlysnp.recode.vcf.gz
cat /path-to-reference/KPNIH1.fasta | vcf-consensus Rush_KPC_266__filter_onlysnp.recode.vcf.gz > Rush_KPC_266__consensus.fa
```

> Note: Dont forget to put the actual path to the refeerence sequence in place of /path-to-reference/

Check the fasta header and change it using sed.

```
head -n1 Rush_KPC_266__consensus.fa
sed -i 's/>.*/>Rush_KPC_266_/g' Rush_KPC_266__consensus.fa 
```

**3. Variant Annotation using snpEff**

Variant annotation is one of the crucial steps in any Variant Calling Pipeline. Most of the variant annotation tools creates their own database or an external one to assign function and predicts the effect of variants on genes. We will try to touch base on some basic steps of annotating variants in our vcf file using snpEff. 
You can annoate these variants before performing any filtering steps that we did earlier or you can decide to annotate just the final filtered variants. 

snpEff contains database of about 20000 reference genome built from trusted and public sources. Lets check if snpEff contains a database of our reference genome.

>i. Check snpEff internal database for your reference genome:

```     
java -jar /scratch/micro612w16_fluxod/shared/bin/snpEff/snpEff.jar databases | grep 'kpnih1'
```
Note down the genome id for your reference genome KPNIH1. In this case: GCA_000281535.2.29

>ii. Change the Chromosome name in vcf file to ‘Chromosome’ for snpEff reference database compatibility. 

```
sed -i 's/gi.*|/Chromosome/g' Rush_KPC_266__filter_gatk.vcf
```
>iii. Run snpEff for variant annotation.

```
java -jar /scratch/micro612w16_fluxod/shared/bin/snpEff/snpEff.jar -v GCA_000281535.2.29 Rush_KPC_266__filter_gatk.vcf > Rush_KPC_266__filter_gatk_ann.vcf -htmlStats Rush_KPC_266__filter_gatk_stats
```

The STDOUT  will print out some useful details such as genome name and version being used, no. of genes, protein-coding genes and transcripts, chromosome and plasmid names etc

Lets go through the ANN field added after annotation step.

```
grep '^Chromosome' Rush_KPC_266__filter_gatk_ann.vcf | head -n1
```

ANN field will provide information such as the impact of variants (HIGH/LOW/MODERATE/MODIFIER) on genes and transcripts along with other useful annotations.

Detailed information of ANN field and sequence onlogy terms that it uses can be found [here](http://snpeff.sourceforge.net/SnpEff_manual.html#input)


**4. Generate Statistics report using samtools, vcftools and qualimap**

Lets try to get some statistics about various outputs that were created using the above steps and check if everything makes sense.

>i. Reads Alignment statistics:
 
```
samtools flagstat Rush_KPC_266__aln.bam > Rush_KPC_266__alignment_stats
```

These statistics will give you an idea about how well your reads aligned to the reference genome in terms of what percentage of reads that you supplied actually mapped to the genome. 

ii. VCF statistics:  

```
bgzip Rush_KPC_266__aln_mpileup_raw.vcf   
tabix Rush_KPC_266__aln_mpileup_raw.vcf.gz  
vcf-stats Rush_KPC_266__aln_mpileup_raw.vcf.gz > Rush_KPC_266__raw_vcf_stats
```

Open Rush_KPC_266__raw_vcf_stats and check the number of snps and indels called for this sample.  

iii. Qualimap report of BAM coverage:

Qualimap outputs a very imformative reports about the alignments and coverage across the entire genome. Let create one for our samples. The below command call bamqc utility of qualimap and generates a report in pdf format.

``` 
qualimap bamqc -bam Rush_KPC_266__aln_sort.bam -outdir ./ -outfile Rush_KPC_266__report.pdf -outformat pdf 
```

Lets get this pdf report onto our local system and check the chromosome stats table, mapping quality and coverage across the entire reference genome.

```
scp username@flux-xfer.engin.umich.edu:/scratch/micro612w16_fluxod/username/day1_after/Rush_KPC_266_varcall_result/Rush_KPC_266__report.pdf /path-to-local-directory/
```

## Visualize BAM and VCF files in IGV or ACT
[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

A visual visualization of all these various output helps in making some significant decisions and inferences about your entire analysis. There are wide a wide variety of visualization tools out there that you can choose for this purpose.
Lets go ahead and use IGV- Integrative Genomics Viewer(developed by Broad Institute) for viewing BAM and vcf files.


> Required Input files: KPNIH1 reference fasta and genbank file, Rush_KPC_266__aln_marked.bam and Rush_KPC_266__aln_marked.bai, Rush_KPC_266__aln_mpileup_raw.vcf/Rush_KPC_266__filter_onlysnp.recode.vcf/Rush_KPC_266__filter_gatk_ann.vcf

Lets make a seperate folder for the files that we need for visualization and copy it to that folder

```
mkdir IGV_files
bgzip -d Rush_KPC_266__aln_mpileup_raw.vcf.gz
cp /path-to-reference/KPNIH1.fasta Rush_KPC_266__aln_marked.bam Rush_KPC_266__aln_marked.bai Rush_KPC_266__aln_mpileup_raw.vcf Rush_KPC_266__filter_onlysnp.recode.vcf Rush_KPC_266__filter_gatk_ann.vcf IGV_files/
```

We need to replace the genome name we changed earlier for snpEff compatibility.

```
cd IGV_files
sed -i 's/Chromosome/gi|661922017|gb|CP008827.1|/g' *.vcf
```

Get these file to your local system and start IGV.

```
scp -r username@flux-xfer.engin.umich.edu:/scratch/micro612w16_fluxod/username/day1_after/Rush_KPC_266_varcall_result/IGV_files/ /path-to-local-directory/
```

Lets first load our reference genome by selecting Genome -> Load Genome from File
Now load raw vcf, annotated vcf and BAM file by selecting File -> Load From File seperately for each file.






Lets type these genomic positions in the position box: 321,818-322,060 to see an example of HET variants.The coverage for this region 321881 - 321980 is unusually high compare to its flanking region(upto 300). Let zoom in and hover at the coverage for one of the variants observed in these region which are called HET variants. This means that more than one allele with high quality and depth was observed at these positions so we cannot decide which one of these is a true variant.

We removed these type of variants during our Variant Filteration step using the criteria FQ. (If the FQ is unusually high, it is suggestive of HET variant and negative FQ value is a suggestive of true variant as observed in the mapped reads) 
You can inspect these HET variants later for any gene duplication and addition of these details will give a better resolution while inferring Phylogenetic trees.


```
screenshots explanation here
```

Play around with IGV to look at what other kind of information you can find from these BAM and vcf files. Also refer to [this](https://www.broadinstitute.org/igv/UserGuide) user guide provided by Broad Institute or watch [this](https://www.youtube.com/watch?v=IILfC3Uc6Vo) video for different kind files that you can visualize in IGV.

[[back to top]](https://github.com/alipirani88/Comparative_Genomics#bacterial-comparative-genomics-workshop)
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

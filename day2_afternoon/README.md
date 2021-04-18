Day 2 PM
========
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

Goal
----

This morning we learned how to perform basic genome annotation and comparison using Prokka and ACT. Now we will up the ante and do some more sophisticated comparative genomics analyses! 
- First, we will create custom BLAST databases to identify specific antibiotic resistance genes of interest in a set of genomes. 
- Second, we will use the tool [ARIBA](https://github.com/sanger-pathogens/ariba/wiki) to identify the complete antibiotic resistome in our genomes. 
- Third, we will move beyond antibiotic resistance, and look at the complete set of protein coding genes in our input genomes. 
- Finally, we will go back to ACT to understand the sorts of genomic rearrangements underlying observed variation in gene content.

![roadmap](comp_genomics.png)


For BLAST and ARIBA, we will be looking at 8 *Klebsiella pneumoniae* genomes from human and environmental sources. Six of these genomes are from [this paper](https://www.pnas.org/content/112/27/E3574), and the other two are sequences from our lab. We are interested in learning more about potential differences in the resistomes of human and environmental isolates. 

For the pan-genome analysis, we will be looking at four closely related *Acinetobacter baumannii* strains. However, despite being closely related, these genomes have major differences in gene content, as *A. baumannii* has a notoriously flexible genome! In fact, in large part due to its genomic flexibility, *A. baumannii* has transitioned from a harmless environmental contaminant to a pan-resistant super-bug in a matter of a few decades. If you are interested in learning more, check out this nature [review](http://www.nature.com/nrmicro/journal/v5/n12/abs/nrmicro1789.html) or [this](http://www.pnas.org/content/108/33/13758.abstract) paper I published a few years back analyzing the very same genomes you will be working with.

Navigate to the `day2pm` directory and load the conda environment you created before break for this session:


```  

cd /scratch/micro612w21_class_root/micro612w21_class/username

or

wd

cp -r /scratch/micro612w21_class_root/micro612w21_class/shared/data/day2pm/ ./


```

Load your environment and use the tools

```
conda activate day2pm
```

Determine which genomes contain KPC genes using [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi)
----------------------------------------------------

![blast](comp_genomics_details_blast.png)
Before comparing full genomic content, lets start by looking for the presence of particular genes of interest. Some *K. pneumoniae* harbor a KPC gene that confers resistance to carbapenems, a class of antibiotics of last resort (more information [here](https://www.sciencedirect.com/science/article/pii/S1473309913701907?via%3Dihub) and [here](https://academic.oup.com/jid/article/215/suppl_1/S28/3092084)). We will see if any of our samples have a KPC gene, by comparing the genes in our genomes to KPC genes extracted from the antibiotic resistance database ([ARDB](http://ardb.cbcb.umd.edu/)). These extracted genes can be found in the file `blast/data/blast_kleb/ardb_KPC_genes.pfasta`, which we will use to generate a BLAST database.

First, change directories to the blast directory:

```
#change directory to day2pm
d2a

cd blast
```

> ***i. Run makeblastdb on the file of KPC genes to create a BLAST database.***

makeblastdb takes as input: 

1) an input fasta file of protein or nucleotide sequences (`data/blast_kleb/ardb_KPC_genes.pfasta`) and 

2) a flag indicating whether to construct a protein or nucleotide database (in this case protein: `-dbtype prot`).

```
makeblastdb -in data/blast_kleb/ardb_KPC_genes.pfasta -dbtype prot

```

> ***ii. BLAST K. pneumoniae protein sequences against our custom KPC database.***

Run BLAST! 

The input parameters are: 

1) query sequences (`-query data/blast_kleb/kpneumo_all.pfasta`), 

2) the database to search against (`-db data/blast_kleb/ardb_KPC_genes.pfasta`), 

3) the name of a file to store your results (`-out KPC_blastp_results.tsv`), 

4) output format (`-outfmt 6`), 

5) e-value cutoff (`-evalue 1e-100`), 

6) number of database sequences to return (`-max_target_seqs 1`) (Note that when using large databases, this might not give you the best hit. See [here](https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/bty833/5106166) for more details.)


```
blastp -query data/blast_kleb/kpneumo_all.pfasta -db data/blast_kleb/ardb_KPC_genes.pfasta -out KPC_blastp_results.tsv -outfmt 6 -evalue 1e-100 -max_target_seqs 1
```

Use `less` to look at `KPC_blastp_results.tsv`. Which genomes have a KPC gene?

```
less KPC_blastp_results.tsv
```

[Here](http://www.metagenomics.wiki/tools/blast/blastn-output-format-6) is more information about the content for each of the output file columns.

- **Exercise:** In this exercise you will try a different type of blasting – blastx. Blastx compares a nucleotide sequence to a protein database by translating the nucleotide sequence in all six frames and running blastp. Your task is to determine which Enterococcus genomes are vancomycin resistant (VRE, vs. VSE) by blasting against a database of van genes. The required files are located in `blast/data/blast_ent` folder in the `day2pm` directory.

Your steps should be:

1) Concatenate the `data/blast_ent/*.fasta` files (VRE/VSE genomes) into a single file (your blast query file) using the `cat` command.
2) Create a blastp database from `data/blast_ent/ardb_van.pfasta`
3) Run blastx
4) Verify that only the VRE genomes hit the database
5) For extra credit, determine which van genes were hit by using grep to search for the hit gene ID in `data/blast_ent/ardb_van.pfasta`

<details>
  <summary>Solution</summary>
  
```
d2a

cd blast/data/blast_ent

# Make sure you are in blast_ent folder
cat *.fasta > VRE_VSE_genomes.fasta

makeblastdb -in ardb_van.pfasta -dbtype prot

blastx -query VRE_VSE_genomes.fasta -db ardb_van.pfasta -out van_blastp_results.tsv -outfmt 6 -evalue 1e-100 -max_target_seqs 1

```
</details>

- **Exercise:** Experiment with the `–outfmt` parameter, which controls different output formats that BLAST can produce. You can use `blastp -help | less` to get more information about the different output formats. You can search for the `-outfmt` flag by typing `/outfmt` and then typing `n` to get to the next one.

Identify antibiotic resistance genes with [ARIBA](https://github.com/sanger-pathogens/ariba) directly from paired end reads
----------------------------------------------------------

Now let's look at the full spectrum of antibiotic resistance genes in our *Klebsiella* genomes!

[ARIBA](https://github.com/sanger-pathogens/ariba/wiki) (Antimicrobial Resistance Identification By Assembly) is a tool that identifies antibiotic resistance genes by running local assemblies. The input is a FASTA file of reference sequences (can be a mix of genes and noncoding sequences) and paired sequencing reads. ARIBA reports which of the reference sequences were found, plus detailed information on the quality of the assemblies and any variants between the sequencing reads and the reference sequences.

ARIBA is compatible with various databases and also contains a utility to download different databases such as: argannot, card, megares, plasmidfinder, resfinder, srst2_argannot, vfdb_core. Today, we will be working with the [card](https://card.mcmaster.ca/) database (`ariba/data/CARD/` in your `day2pm` directory).

> ***i. Run ARIBA on input paired-end fastq reads for resistance gene identification.***

The fastq reads are in the `ariba/data/kpneumo_fastq/` directory. Since ARIBA is a memory intensive, we started this at the beginning of the afternoon. We should have our results by now, but first let's look at the ARIBA command.

```
# navigate to ariba directory
cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm

# or

d2a

cd ariba

# load conda environment if not already loaded
conda activate day2pm

# look at ariba commands
less ariba.sbat
```


> ***ii. Run ARIBA summary function to generate a summary report.***

ARIBA has a summary function that summarises the results from one or more sample runs of ARIBA and generates an output report with various level of information determined by the `-preset` parameter. The parameter `-preset minimal` will generate a minimal report showing only the presence/absence of resistance genes whereas `-preset all` will output all the extra information related to each database hit such as reads and reference sequence coverage, variants and their associated annotations (if the variant confers resistance to an antibiotic) etc.

```
# look at ariba output
ls results
ls results/card
ls results/card/*

# make directory for ariba results

ariba summary --preset minimal results/kpneumo_card_minimal_results results/card/*/report.tsv

ariba summary --preset all results/kpneumo_card_all_results results/card/*/report.tsv
```

The ARIBA summary generates three output:

1. `kpneumo_card*.csv` file that can be viewed in your favorite spreadsheet program (e.x. Microsoft Excel).

2. `kpneumo_card*.phandango.{csv,tre}` that allow you to view the results in [Phandango](http://jameshadfield.github.io/phandango/#/). You can drag-and-drop these files straight into Phandango.

Lets copy these  files, along with a metadata file, to the local system using cyberduck or scp.

```
mkdir ~/Desktop/micro612
mkdir ~/Desktop/micro612/day2pm

scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/results/kpneumo_card* ~/Desktop/micro612/day2pm
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/data/kpneumo_source.tsv ~/Desktop/micro612/day2pm
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/ariba/data/mlst_typing/kpneumo_mlst.tsv ~/Desktop/micro612/day2pm

```

Drag and drop these two files onto the [Phandango](http://jameshadfield.github.io/phandango/#/) website. What types of resistance genes do you see in these *Klebsiella* genomes? 

> ***iii. Explore full ARIBA matrix in R***

Now, fire up RStudio and read in the ARIBA full report `kpneumo_ariba_all_results.csv` so we can take a look at the results.

```
# Read in data
ariba_full  = read.csv(file = '~/Desktop/micro612/day2pm/kpneumo_card_all_results.csv', row.names = 1)
rownames(ariba_full) = gsub('_1|_R1|/report.tsv|card/|results/','',rownames(ariba_full))

# Subset to get description for each gene
ariba_full_match = ariba_full[, grep('match',colnames(ariba_full))]

# Make binary for plotting purposes
ariba_full_match[,] = as.numeric(ariba_full_match != 'no')

# Make a heatmap!

# install pheatmap if you don't already have it installed 
#install.packages('pheatmap')

# load pheatmap
library(pheatmap)

# load metadata about sample source
annots = read.table('~/Desktop/micro612/day2pm/kpneumo_source.tsv',row.names=1)
colnames(annots) = 'Source'

# plot heatmap
pheatmap(ariba_full_match,annotation_row = annots)
```

Bacteria of the same species can be classified into different sequence types (STs) based on the sequence identity of certain housekeeping genes using a technique called [multilocus sequence typing (MLST)](https://en.wikipedia.org/wiki/Multilocus_sequence_typing). The different combination of these house keeping sequences present within a bacterial species are assigned as distinct alleles and, for each isolate, the alleles at each of the seven genes define the allelic profile or sequence type (ST). Sometimes, different sequence types are associated with different environments or different antibiotic resistance genes. We want to know what sequence type(s) our genomes come from, and if there are certain ones that are associated with certain sources or certain antibiotic resistance genes. 

We already pre-ran Ariba MLST on all 8 of our *K. pneumonia* genomes. Use the MLST results kpneumo_mlst.tsv that we previously downloaded to add a second annotation column to the heatmap we created above to visualize the results. 

Go to your R studio and overlay MLST metadata as an additional row annotation to your previous heatmap

```
#read in MLST data
annots_mlst = read.table('~/Desktop/micro612/day2pm/kpneumo_mlst.tsv',row.names=1)

#make sure order of genomes is the same as source annotation
annots_mlst$ST = annots_mlst[row.names(annots),]

#change from numeric to character, so that heatmap doesn't treat ST as continuous variable
Row_annotations$ST = as.character(Row_annotations$ST)

#paste together annotations
Row_annotations <- cbind(annots, annots_mlst) 

#create new heatmap with source and mlst
pheatmap(ariba_full_match,annotation_row = Row_annotations, annotation_colors = annoCol)
```


The commands and steps that were used for MLST typing are given below:

<details>
  <summary>Solution</summary>

**Dont run this exercise**

Steps:
1. Check if you have an MLST database for your species of interest using `ariba pubmlstspecies`.
2. Download your species MLST database. You can look at the manual or run the command `ariba pubmlstget -h` to help figure out how to download the correct MLST database. I would suggest downloading it to the `data` directory. 
3. Copy the `ariba.sbatch` file to a new file called `mlst.sbatch`.
4. Modify the `mlst.sbatch` script in the following ways:
    1. Change the database directory to the _K. pneumoniae_ MLST database you just downloaded.
    1. Change the `mkdir` line to make a `results/mlst` directory.
    1. Modify the output directory `outdir` line: Change `card` to `mlst`.
5. Submit the `mlst.sbatch` script. It should take about 7 minutes to run.
6. Once the run completes, run `scripts/summarize_mlst.sh results/mlst` to look at the MLST results. If you want, you can save it to its own file. What sequence types are present? 


```
# Make sure you are in ariba directory under day2pm folder and running the below commands from ariba directory.
d2a
cd ariba

# Check if you have an mlst database for your species of interest
ariba pubmlstspecies

# Download your species mlst database
ariba pubmlstget "Klebsiella pneumoniae" data/MLST_db

# Set ARIBA database directory to the get_mlst database that we just downloaded.
db_dir=data/MLST_db/ref_db/

# Run ariba mlst with this database
samples=$(ls data/kpneumo_fastq/*1.fastq.gz) #forward reads

# Generate mlst folder under results folder to save ariba MLST results
mkdir results/mlst_typing

# Run for loop, where it generates ARIBA command for each of the forward end files.
for samp in $samples; do   
samp2=${samp//1.fastq/2.fastq} #reverse reads   
outdir=results/mlst_typing /$(echo $samp | cut -d/ -f3 | sed 's/_.*1.fastq.gz//g')
echo "Results will be saved in $outdir"
echo "Running: ariba run --force $db_dir $samp $samp2 $outdir  #ariba command "
ariba run --force $db_dir $samp $samp2 $outdir  #ariba command 
done

# Once the run completes, run summarize_mlst.sh script to print out mlst reports that are generated in current directory
bash scripts/summarize_mlst.sh results/mlst

```
</details>


Perform pan-genome analysis with [Roary](https://sanger-pathogens.github.io/Roary/)
----------------------------------------
![roary](comp_genomics_details_roary.png)

Roary is a pan genome pipeline, which takes annotated assemblies in GFF3 format and calculates the pan-genome. The pan-genome is just a fancy term for the full complement of genes in a set of genomes. 

The way Roary does this is by: 
1) Roary gets all the coding sequences from GFF files, converts them into protein, and creates pre-clusters of all the genes.
2) Then, using BLASTP and MCL, Roary will create gene clusters, and check for paralogs. 
3) Finally, Roary will take every isolate and order them by presence/absence of genes.

> ***i. Generate pan-genome matrix using Roary and GFF files***

Change your directory to `day2pm`

```

# Make sure to change username with your uniqname

cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm/roary

# or 

d2a
cd roary

```

Let's look at the Roary command:

```
less roary.sbat
```

The roary command runs pan-genome pipeline on gff files placed in `Abau_genomes_gff` (`-v`) using 4 threads (`-p`), save the results in an output directory `Abau_genomes_roary_output` (`-f`), generate R plots using .Rtab output files and align core genes(`-n`)

Change directory to `Abau_genomes_roary_output` to explore the results.

```
cd Abau_genomes_roary_output

ls
```

Output files:

1. `summary_statistics.txt`: This file is an overview of your pan genome analysis showing the number of core genes(present in all isolates) and accessory genes(genes absent from one or more isolates or unique to a given isolate). 

2. `gene_presence_absence.csv`: This file contain detailed information about each gene including their annotations which can be opened in any spreadsheet software to manually explore the results. It contains plethora of information such as gene name and their functional annotation, whether a gene is present in a genome or not, minimum/maximum/Average sequence length etc.

3. `gene_presence_absence.Rtab`: This file is similar to the gene_presence_absence.csv file, however it just contains a simple tab delimited binary matrix with the presence and absence of each gene in each sample. It can be easily loaded into R using the read.table function for further analysis and plotting. The first row is the header containing the name of each sample, and the first column contains the gene name. A 1 indicates the gene is present in the sample, a 0 indicates it is absent.

4. `core_gene_alignment.aln`: a multi-FASTA alignment of all of the core genes that can be used to generate a phylogenetic tree.

> ***ii. Explore pan-genome matrix gene_presence_absence.csv and gene_presence_absence.Rtab using R***

<!---
Note:plots generated by roary_plots.py doesn't seem to be very useful and is completely a waste of time. query_pan_genome script provided by roary doesn't work and generates an empty result which seems like a bug and the link to the issue raised for this bug can be found [here](https://github.com/sanger-pathogens/Roary/issues/298) 
-->

We're going to use information from `gene_presence_absence.csv` and `gene_presence_absence.Rtab`. Let's take a look at these from the Terminal using `less`:

```
# annotation information is here
less -S gene_presence_absence.csv
# presence/absence information
less -S gene_presence_absence.Rtab
```

**Read matrices into R, generate exploratory plots and query pan-genome**

Use scp or cyberduck to get `gene_presence_absence.csv` and `gene_presence_absence.Rtab` onto your laptop desktop folder.

```
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/roary/Abau_genomes_roary_output/gene_presence_absence.csv ~/Desktop/micro612/day2pm
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/username/day2pm/roary/Abau_genomes_roary_output/gene_presence_absence.Rtab ~/Desktop/micro612/day2pm
```

> ***i. Prepare and clean data***

- Fire up RStudio and read both files in.

```
# read in annotations (only need 3rd column)
annots = read.csv('~/Desktop/micro612/day2pm/gene_presence_absence.csv')[,3]
# read in presence-absence heatmap
pg_matrix = read.delim('~/Desktop/micro612/day2pm/gene_presence_absence.Rtab', row.names = 1, header=TRUE)
```

- Use head, str, dim, etc. to explore the matrix.

> ***ii. Generate exploratory heatmaps.***

- Make a heatmap for the full matrix

```
# load pheatmap
library(pheatmap)

# make heatmap
pheatmap(pg_matrix,col= c('black', 'red'), show_rownames = F)
```

- Make a heatmap for variable genes (present in at least one, but not all of the genomes)

```
# subset matrix to include only variable genes
pg_matrix_subset = pg_matrix[rowSums(pg_matrix > 0) > 0 & rowSums(pg_matrix > 0) < 4 ,] 
# make heatmap of variable genes
pheatmap(pg_matrix_subset,col= c('black', 'red'), show_rownames = F)
```

> ***iii. Query pan-genome***

-  Which genomes are most closely related based upon shared gene content?

We will use the apply function to determine the number of genes shared by each pair of genomes. 

```
apply(pg_matrix_subset, 2, function(x){
  colSums(pg_matrix_subset == x & pg_matrix_subset == 1)
})
```

- What is the size of the core genome?

Lets first get an overview of how many genes are present in different numbers of genomes (0, 1, 2, 3 or 4) by plotting a histogram. Here, we combine hist with rowSums to accomplish this.

```
hist(rowSums(pg_matrix > 0), col="red")
```

Next, lets figure out how big the core genome is (e.g. how many genes are common to all of our genomes)?

```
sum(rowSums(pg_matrix > 0) == 4)
```

- What is the size of the accessory genome?

Lets use a similar approach to determine the size of the accessory genome (e.g. those genes present in only a subset of our genomes).

```
sum(rowSums(pg_matrix > 0) < 4 & rowSums(pg_matrix > 0) > 0)
```

- What types of genes are unique to a given genome?

So far we have quantified the core and accessory genome, now lets see if we can get an idea of what types of genes are core vs. accessory. Lets start by looking at those genes present in only a single genome. 

```
# genes present only in a single genome
unique_annots = annots[rowSums(pg_matrix > 0) == 1]
unique_annots
```

What do you notice about these genes?

- What is the number of hypothetical genes in core vs. accessory genome?

Looking at unique genes we see that many are annotated as “hypothetical”, indicating that the sequence looks like a gene, but has no detectable homology with a functionally characterized gene. 

Determine the fraction of “hypothetical” genes in unique vs. core. 

```
# genes present in all genomes
core_annots = annots[rowSums(pg_matrix > 0) == 4]

# fraction of unique and core hypothetical genes
# unique hypothetical
sum(grepl("hypothetical" , unique_annots)) / sum(rowSums(pg_matrix > 0) == 1)
# core hypothetical
sum(grepl("hypothetical" , core_annots)) / sum(rowSums(pg_matrix > 0) == 4)
```

Why does this make sense?

Perform genome comparisons with [ACT](http://www.sanger.ac.uk/science/tools/artemis-comparison-tool-act)
-------------------------------------

![act](comp_genomics_details_ACT.png)

In the previous exercises we were focusing on gene content, but losing the context of the structural variation underlying gene content variation (e.g. large insertions and deletions). 
Here we will use ACT to compare two of our genomes (note that you can use ACT to compare more than two genomes if desired). 

> ***i. Create ACT alignment file with BLAST***

As we saw this morning, to compare genomes in ACT we need to use BLAST to create the alignments. We will do this on great lakes.

```
# navigate to act directory
cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm

# or

d2a
cd act
```

Run blast on two _A. baumanii_ genomes:
```
blastn -query data/Abau_genomes/AbauA_genome.fasta -db data/Abau_BLAST_DB/ACICU_genome.fasta -outfmt 6 -evalue 1e-20 > AbauA_vs_ACICU.blast
```

> ***ii. Read in genomes, alignments and annotation files***

Use scp or cyberduck to transfer Abau_ACT_files folder onto your laptop


1. `data/Abau_genomes/AbauA_genome.fasta` 
2. `data/Abau_genomes/ACICU_genome.fasta` 
3. `AbauA_vs_ACICU.blast` 
4. `data/Abau_ACT_files/AbauA_genome_gene.gff` 
5. `data/Abau_ACT_files/ACICU_genome_gene.gff`


> ***iii. Explore genome comparison and features of ACT***

Read in genomes and alignment into ACT

```

Go to File -> open 
Sequence file 1  = ACICU_genome.fasta 
Comparison file 1  = AbauA_vs_ACICU.blast
Sequence file 2  = AbauA_genome.fasta

```

Before we use annotation present in genbank files. Here we will use ACT specific annotation files so we get some prettier display (resistance genes = red, transposable elements = bright green)  

```

Go to File -> ACICU_genome.fasta -> Read an entry file = ACICU_genome_gene.gff

Go to File -> AbauA_genome.fasta -> Read an entry file = AbauA_genome_gene.gff

```

Play around in ACT to gain some insight into the sorts of genes present in large insertion/deletion regions. 
See if you can find:

1) differences in phage content, 
2) membrane biosynthetic gene cluster variation and 
3) antibiotic resistance island variation.


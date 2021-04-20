Day 1 Morning
=============
[[HOME]](https://github.com/alipirani88/Comparative_Genomics/blob/master/README.md)

Goal
-----

- This morning we will learn how to set up our unix and conda environment which is a necessity when it comes to working on command line. 
- Setting up an environment will make our life easier and running commands more enjoyable. 
- We will brush up on few unix programs that some of you learned in Data Carpentry workshop and see how they can be employed for accessing and parsing omics datasets. 
- We will also learn how for loops and awk can be employed to parse and extract complex information from common bioinformatics file formats. 
- At the end of the session, an R exercise will give you an overview as to how you can parse and visualize omics datasets. 


Installing and setting up Cyberduck for file transfer
-----------------------------------------------------

During workshop, we will transfer different output files from great lakes to your local system. Cyberduck makes it easier to drag and drop any remote file onto your local system and vice versa. Of course, you can use "scp" to transfer files but Cyberduck provides a graphical interface to manage file transfer and helps avoid typing long file paths and commands.

> ***1. Go to [this](https://cyberduck.io/) cyberduck website and download the executable for your respective operating system.***

> ***2. Double-click on the downloaded zip file to unzip it and double click cyberduck icon.***

> ***3. Type sftp://greatlakes-xfer.arc-ts.umich.edu in quickconnect bar(or click on Open Connection button on top left corner), press enter and enter your great lakes username and password.***

> ***4. This will take you to your great lakes home directory /home/username. Select "Go" from tool bar at the top then select "Go to folder" and enter workshop home directory path: /scratch/micro612w21_class_root/micro612w21_class/***


To transfer or upload a file, you can drag and drop it into the location you want. 


Getting your data onto great lakes and setting up environment variable
---------------------------------------------------------------

**Log in to great lakes**


```
ssh username@greatlakes.arc-ts.umich.edu
```

<!-- **Set up your .bashrc file so your environment is all set for genomic analysis!** -->

**Setting up environment variables in .bashrc file so your environment is all set for genomic analysis!**

Environment variables are the variables/values that describe the environment in which programs run in. All the programs and scripts on your unix system use these variables for extracting information such as: 

- What is my current working directory?, 
- Where are temporary files stored?, 
- Where are perl/python libraries?, 
- Where is Blast installed? etc. 

In addition to environment variables that are set up by system administators, each user can set their own environment variables to customize their experience. This may sound like something super advanced that isn't relevant to beginners, but that's not true! 

Some examples of ways that we will use environment variables in the class are: 

1) create shortcuts for directories that you frequently go to,

2) Setup a conda environment to install all the required tools and have them availble in your environment

3) setup a shortcut for getting on a cluster node, so that you don't have to write out the full command each time.

One way to set your environment variables would be to manually set up these variables everytime you log in, but this would be extremely tedious and inefficient. So, Unix has setup a way around this, which is to put your environment variable assignments in special files called .bashrc or .bash_profile. Every user has one or both of these files in their home directory, and what's special about them is that the commands in them are executed every time you login. So, if you simply set your environmental variable assignments in one of these files, your environment will be setup just the way you want it each time you login!

All the softwares/tools that we need in this workshop are installed in a directory "/scratch/micro612w21_class_root/micro612w21_class/shared/bin/" and we want the shell to look for these installed tools in this directory. For this, We will save the full path to these tools in an environment variable PATH.

> ***i. Make a backup copy of bashrc file in case something goes wrong.***
	
```

cp ~/.bashrc ~/bashrc_backup

#Note: "~/" represents your home directory. On great lakes, these means /home/username

```
	
> ***ii. Open ~/.bashrc file using any text editor and add the following lines to your .bashrc file.***



```
##Micro612 Workshop ENV

#Aliases
alias islurm='srun --account=micro612w21_class --nodes=1 --ntasks-per-node=1 --mem-per-cpu=5GB --cpus-per-task=1 --time=12:00:00 --pty /bin/bash'
alias wd='cd /scratch/micro612w21_class_root/micro612w21_class/username/'
alias d1m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day1am'
alias d1a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day1pm'
alias d2m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day2am'
alias d2a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day2pm'
alias d3m='cd /scratch/micro612w21_class_root/micro612w21_class/username/day3am'
alias d3a='cd /scratch/micro612w21_class_root/micro612w21_class/username/day3pm'


#Great Lakes Modules
#module load Bioinformatics
#module load perl-modules



#Bioinformatics Tools
export PATH=$PATH:/scratch/micro612w21_class_root/micro612w21_class/shared/bin/quast/

```



Note: Replace "username" under alias shortcuts with your own umich "uniqname". In the text editor, nano, you can do this by 

- typing Ctrl + \ and You will then be prompted to type in your search string (here, username). 
- Press return. Then you will be prompted to enter what you want to replace "username" with (here, your uniqname). 
- Press return. Then press a to replace all incidences or y to accept each incidence one by one. 

You can also customize the alias name such as wd, d1am etc. catering to your own need and convenience.

The above environment settings will set various shortcuts such as "islurm" for entering interactive great lakes session, "wd" to navigate to your workshop directory, call necessary great lakes modules and perl libraries required by certain tools and finally sets the path for bioinformatics programs that we will run during the workshop.

> ***iii. Save the file and Source .bashrc file to make these changes permanent.***

```

source ~/.bashrc

```

> ***iv. Check if the $PATH environment variable is updated***

```

echo $PATH

#You will see a long list of paths that has been added to your $PATH variable

wd

```

You should be in your workshop working directory that is /scratch/micro612w21_class_root/micro612w21_class/username 

> ***v. Set up a conda environment using a YML file***

The YML file - micro612.yml required for generating the conda environment is located here:

```
/scratch/micro612w21_class_root/micro612w21_class/shared/conda_envs/
```

Load great lakes anaconda package and set up a conda environment in the following way - 

```
# Load anaconda package from great lakes 
module load python3.7-anaconda/2019.07

# Set channel_priority to false so that it can install packages as per the YML file and not from loaded channels.
conda config --set channel_priority false

# Create a new conda environment - micro612 from a YML file
conda env create -f /scratch/micro612w21_class_root/micro612w21_class/shared/conda_envs/micro612.yml -n micro612

# Load your environment and use the tools
conda activate micro612

# Update one of the databases that you would need in one of the Kraken exercises 
ktUpdateTaxonomy.sh
```



Unix is your friend
-------------------

Up until now you’ve probably accessed sequence data from NCBI by going to the website, laboriously clicking around and finally finding and downloading the data you want. 

There are a lot of reasons that is not ideal:

- It’s frustrating and slow to deal with the web interface
- It can be hard to keep track of where the data came from and exactly which version of a sequence you downloaded
- Its not conducive to downloading lots of sequence data

To download sequence data in Unix you can use a variety of commands (e.g. sftp, wget, curl). Here, we will use the curl command to download some genome assemblies from NCBI ftp location:

- Go to your class home directory (use your wd shortcut!)

- Execute the following commands to copy files for this morning’s exercises to your home directory: 

```
cp -r /scratch/micro612w21_class_root/micro612w21_class/shared/data/day1am/ ./

cd day1am/

#or 

d1m

ls

```

- Now get three genome sequences with the following commands:

```
curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/241/685/GCF_000241685.1_ASM24168v2/GCF_000241685.1_ASM24168v2_genomic.fna.gz >Acinetobacter_baumannii.fna.gz

curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/409/005/GCF_000409005.1_gkp33v01/GCF_000409005.1_gkp33v01_genomic.fna.gz > Kleb_pneu.fna.gz

curl ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/165/655/GCF_000165655.1_ASM16565v1/GCF_000165655.1_ASM16565v1_genomic.fna.gz > E_coli.fna.gz

```

- Decompress the gzip compressed fasta file using gzip command

```
gzip -d Acinetobacter_baumannii.fna.gz
gzip -d Kleb_pneu.fna.gz
gzip -d E_coli.fna.gz
```

These files are genome assemblies in fasta format. Fasta files are a common sequence data format that is composed of alternating sequence headers (sequence names and comments) and their corresponding sequences. Of great importance, the sequence header lines must start with “>”. These genome assemblies have one header line for each contig in the assembly, and our goal will be to count the number of contigs/sequences. To do this we will string together two Unix commands: “grep” and “wc”. “grep” (stands for global regular expression print), is an extremely powerful pattern matching command, which we will use to identify all the lines that start with a “>”. “wc” (stand for word count) is a command for counting words, characters and lines in a file. To count the number of contigs in one of your fasta files enter:


```
grep ">" E_coli.fna | wc -l
```

Try this command on other assemblies to see how many contigs they contain. 

Your first sequence analysis program!!!
---------------------------------------

OK, so now that we have a useful command, wouldn’t it be great to turn it into a program that you can easily apply to a large number of genome assemblies? Of course it would! So, now we are going to take out cool contig counting command, and put it in a shell script that applies it to all files in the desired directory.

<!--- Copy “/scratch/micro612w21_class_root/micro612w21_class/shared/fasta_counter.sh” to your current directory (Hint – use the “cp” command)-->

There will be times when you have multiple sets of files in a folder in which case it becomes cumbersome to run individual commands on each file. To simplify this task, most programming language have a concept of loops that can be employed to repeat a task/command on a bunch of files repeatedly. Here we have three fasta files for which we want to know the number of contigs in each file. We can either run the above mentioned grep command seperately on each file or use it in a "for" loop that iterates through a set of values/files until that list is exhausted. 

Try the below example of for loop, that loops over a bunch of numbers and prints out each value until the list is exhausted.

```
for i in 1 2 3 4 5; do echo "Looping ... number $i"; done
```

A simple for loop statement consists of three sections: 

1. for statement that loops through values and files
2. a do statement that can be any type of command that you want to run on a file or a tool that uses the current loop value  as an input
3. done statement that indicates completion of a do statement.

Note that the list values - (1 2 3 4 5) in the above for loop can be anything at all. It can be a bunch of files in a folder with a specific extension (\*.gz, \*.fasta, \*.fna) or a list of values generated through a seperate command that we will see later.

We will incorporate a similar type of for loop in fasta_counter.sh script that will loop over all the \*.fna files in the folder. We will provide the name of the folder through a command line argument and count the number of contigs in each file. A command line argument is a sort of input that can be provided to a script which can then be used in different ways inside the script. fasta_counter.sh requires to know which directory to look for for \*.fna files. For this purpose, we will use positional parameters that are a series of special variables ($0 through $9) that contain the contents of the command line. 

Lets take an example to understand what those parameters stands for:


```
./some_program.sh Argument1 Argument2 Argument3
```

In the above command, we provide three command line Arguments that acts like an input to some_program.sh 
These command line argument inputs can then be used to inside the scripts in the form of $0, $1, $2 and so on in different ways to run a command or a tool.

Try running the above command and see how it prints out each positional parameters. $0 will be always be the name of the script. $1 would contain "Argument1" , $2 would contain "Argument2" and so on...

Lets try to incorporate a for loop inside the fasta_counter.sh script that uses the first command line argument - i.e directory name and search for \*.fna files in that directory and runs contig counting command on each of them.

- Open “fasta_counter.sh” in nano or your favourite text editor and follow instructions for making edits so it will do what we want it to do

- Run this script in day1am directory and verify that you get the correct results. Basic usage of the script will be:

./fasta_counter.sh <directory containing files>

```
./fasta_counter.sh .
```

The "." sign tells the script to use current directory as its first command line argument($1) 

Power of Unix commands
----------------------

In software carpentry, you learned working with shell and automating simple tasks using basic unix commands. Lets see how some of these commands can be employed in genomics analysis while exploring various file formats that we use in day to day analysis. For this session, we will try to explore two different types of bioinformatics file formats: 

gff: used for describing genes and other features of DNA, RNA and protein sequences

fastq: used for storing biological sequence / sequencing reads (usually nucleotide sequence) and its corresponding quality scores

<!--

fasta: used for representing either nucleotide or peptide sequences

- Question: Previously, you downloaded genome assembly fasta files and ran a shell script to count contigs. Now, lets say you want to find out the combined length of genome in each of these files. This can be achieved by running a short unix command piping together two unix programs: grep and wc. The key to crafting the command is understanding the  features of fasta files,

> ***1) each sequence in fasta file is preceded by a fasta header that starts with ">",***

> ***2) the types of bases that a nucleotide sequence represents (A,T,G,C,N)***


To determine the total length of our genome assemblies, we will use grep to match only those lines that doesn't start with ">" (remember grep -v option is used to ignore lines). Then use wc command (stands for word count) to count the characters. We can use unix pipe "|" to pass the output of one command to another for further processing. Lets start by counting the number of bases in Acinetobacter_baumannii.fna file

<details>
  <summary>Solution</summary>
-->

<!--
grep -v '^>' Acinetobacter_baumannii.fna | sed 's/[N,n]//g' | awk -F '\n' '{sum += length} END {print sum}'
for i in *.fna; do grep -v '^>' $i | sed 's/[N,n]//g' | awk -F '\n' '{sum += length} END {print sum}'; done
grep -v '^#' sample.gff | awk -F '\t' '{print $3}' | grep 'rRNA' | wc -l
grep -v '^#' sample.gff | awk -F '\t' '{print $3}' | grep 'CDS' | wc -l
grep -v '^#' sample.gff | awk -F '\t' '{print $3}' | grep 'tRNA' | wc -l


```

grep -v '^>' Acinetobacter_baumannii.fna | wc -m

#Note:

#- The sign "^" inside the grep pattern represents any pattern that starts with ">" and -v asks grep to ignore those lines.
#- Use "|" to pass the output of one command to another.
#- -m parameter will show the character counts. Check wc help menu by typing "wc --help" on terminal to explore other parameters

```
</details>



Now run the same command on other fasta files in day1am directory. Try using a for loop.


<details>
  <summary>Solution</summary>

```

for i in *.fna; do grep -v '^>' $i | wc -m; done

```
</details>



**Unix one-liners**

As soon as you receive your sample data from sequencing centre, the first thing you do is check its quality using a quality control tool such as FastQC and make sure that it contain sequences from organism that you are working on (Free from any contamination). But before carrying out extensive QC, you can run a bash "one-liner" to get some basic statistics about the raw reads. These one-liners are great examples for how a set of simple (relatively) Unix commands can be piped together to do really useful things.

One such unix utility is Awk commands. Awk is a powerful unix utility that can be used to write tiny but effective programs in the form of statements that define text patterns that are to be searched for in each line of a file and the action that is to be taken when a match is found within a line. It is mostly used for pattern scanning and processing. It searches one or more files to see if they contain lines that matches with the specified patterns and then performs the associated actions.

A simple awk syntax is divided into three parts:

```
awk ' condition { action }'
```

If the line that awk reads satisfies the condition, it runs the action on it.

```

for i in Rush_KPC_266_*.gz; do zcat $i | awk 'BEGIN{OFS="\t"};((NR-2)%4==0){read=$1;total++;len+=length(read)};END{print total,len/total}'; done

#The above awk command reads every fourth record and calculates some basic fastq statistics.

```

The above awk command reads every fourth record and calculates the number of reads and average read length.


Now try running above command using fastq_screen.fastq.gz as input.

To see the true power of Awk unix proggraming and understand how you can employ it to extract complex information, take a look at below command. 

The following command will print total number of reads in each file, total number of unique reads, percentage of unique reads, most abundant sequence(useful to find adapter sequences or contamination), its frequency, and frequency of that sequence as a proportion of the total reads, average read length.

```

for i in Rush_KPC_266_*.gz; do zcat $i | awk 'BEGIN{OFS="\t"};((NR-2)%4==0){read=$1;total++;count[read]++;len+=length(read)}END{for(read in count){if(!max||count[read]>max) {max=count[read];maxRead=read};if(count[read]==1){unique++}};print total,unique,unique*100/total,maxRead,count[maxRead],count[maxRead]*100/total,len/total}'; done

```

This command will parse a fastq file and calculate different statistics on the fly in a time efficient manner. Awk lets you perform and explore complex data files without the need for using a programming language or an individual tool.

You can find more of such super useful bash one-liners at Stephen Turner's github [page](https://github.com/stephenturner/oneliners). You can also use some pre-written unix utilities and tools such as [seqtk](https://github.com/lh3/seqtk), [bioawk](https://github.com/lh3/bioawk) and [fastx](http://hannonlab.cshl.edu/fastx_toolkit/) which comes in handy while extracting complex information from fasta/fastq/sam/bam files and are optimized to be insanely fast.

-->

<!--
Pairing fastq Files with for loop
---------------------------------

Oftentimes, bioinformatics analyses involves pairing a bunch of files that can then be used as an input for a command or a tool. The most common type of file that are used as pairs is fastq forward and reverse reads. In this exercise, we will go through a simple shell script that searches fastq files having a particular extension and generate a filename string for reverse paired end file.

The script fastq_pair.sh takes a path to directory as a command line argument and searches files with extension \*_1_combine.fastq.gz and generates a filename with an extension \*_2_combine.fastq.gz in it. 

You can change the parameter suffix inside the script to look for a different entension.

```
less fastq_pair.sh 
```

Try running the script in the following fashion where . represents current directory:

```
./fastq_pair.sh .
```

How about running the awk command that we recently used inside a shell script and ask awk to print some statistics for both forward and reverse reads? Follow instructions in the script and Insert Awk command  in such a way that you use fwd_fastq_file and rev_fastq_file string accordingly.
-->

Exploring GFF files
-------------------

The GFF (General Feature Format) format is a tab-seperated file and consists of one line per feature, each containing 9 columns of data.

column 1: seqname - name of the genome or contig or scaffold

column 2: source - name of the program that generated this feature, or the data source (database or project name)

column 3: feature - feature type name, e.g. Gene, exon, CDS, rRNA, tRNA, CRISPR, etc.

column 4: start - Start position of the feature, with sequence numbering starting at 1.

column 5: end - End position of the feature, with sequence numbering starting at 1.

column 6: score - A floating point value.

column 7: strand - defined as + (forward) or - (reverse).

column 8: frame - One of '0', '1' or '2'. '0' indicates that the first base of the feature is the first base of a codon, '1' that the second base is the first base of a codon, and so on..

column 9: attribute - A semicolon-separated list of tag-value pairs, providing additional information about each feature such as gene name, product name etc.

- Use less to explore first few lines of a gff file sample.gff

```

less sample.gff

```
Note: lines starting with pound sign "#" represent comments and are used to document extra information about the features.

You will notice that the GFF format follows version 3 specifications("##gff-version 3"), followed by genome name("#Genome: 1087440.3|Klebsiella pneumoniae subsp. pneumoniae KPNIH1"), date("#Date:02/09/2017") when it was generated, contig name("##sequence-region") and finally tab-seperated lines describing features.

You can press space bar on keyboard to read more lines and "q" key to exit less command.

- Question: Suppose, you want to find out the number of annotated features in a gff file. how will you achieve this using grep and wc?

<details>
  <summary>Solution</summary>
  
```
grep -v '^#' sample.gff | wc -l
```
</details>

- Question: How about counting the number of rRNA features in a gff(third column) file using grep, cut and wc? You can check the usage for cut by typing "cut --help"

<details>
  <summary>Solution</summary>
  
```

cut -f 3 sample.gff | grep 'rRNA' | wc -l

#Or number of CDS or tRNA features?

cut -f 3 sample.gff | grep 'CDS' | wc -l
cut -f 3 sample.gff | grep 'tRNA' | wc -l

#Note: In the above command, we are trying to extract feature information from third column.

```
</details>

- Question: Try counting the number of features on a "+" or "-" strand (column 7).

Now, let's use what we learned about for loops and creating shell scripts above to create a script called "feature_counter.sh" This script will take as input a directory and will search for all gff files in the directory. It will calculate and output the number of tRNA features in the gff.   

- Open “feature_counter.sh” in nano or your favourite text editor and follow instructions for making edits so it will do what we want it to do

- Run this script in day1am directory and verify that you get the correct results. Basic usage of the script will be:

./feature_counter.sh <directory containing gff files>

```
./feature_counter.sh .
```


Some more useful one-line unix commands for GFF files: [here](https://github.com/stephenturner/oneliners#gff3-annotations)

Now we're going to play around with a GFF in R. Specifically, we're interested in looking at the distribution of gene length for all of the genes in the gff file.

Copy the sample.gff file to your computer using scp or cyberduck:

```
scp username@greatlakes-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/shared/data/day1am/sample.gff ~/Desktop/

Note: You can use your choice of folder/path to copy the file instead of  “~/Desktop/”
```

Open a text file in RStudio and run the following commands:
```
# Plot histogram of gene lengths

# Read in gff file
gff = read.delim('~/Desktop/sample.gff',
                 comment.char = '#', # ignore lines that start with '#'
                 header=F,  #  no header
                 stringsAsFactors = FALSE) # do not read in as data type "factor"

# Rename columns
colnames(gff) = c('seqname','source','feature','start','end','score','strand','frame','attribute')

# Look at the head of the gff file
head(gff)

# Get the gene lengths
gene_lengths = gff$end - gff$start

# Plot a histogram of the gene lengths
hist(gene_lengths,
     breaks = 100, # 100 cells
     xlab = 'Gene Length (bp)', # change x label
     main = '') # no title
```

What information do you learn about gene lengths in this genome?

<!---
Plotting genomic coverage in R
------------------------------

Data visualization plays an important role in organizing, analyzing and interpreting large amount of omics data. R is one of the most basic and powerful tool for manipulating and visualizing these types of data. The following task will brush up some basic R plotting commands and help you visualize some complex omics data for interpretation.

One of the most common types of genomic analysis involves comparing the newly sequenced read data of an organism to your choice of reference organism genome. Mapping millions of reads generated in a sequencing experiment to the reference genome fasta file and interpreting various parameters can achieve this analysis. 
One such parameter is validating how well your sequencing experiment performed and assessing the “uniformity” of coverage from whole-genome sequencing. Visualizing Sequencing coverage across the reference genome help us answer this question. Sequencing coverage describes the average number of reads that align to, or "cover," known reference bases.

The input for this task is a comma-separated file, which contains average sequencing coverage information i.e average number of reads mapped to each 1000 base pairs in reference genome. You can find this input file in your day1am directory by the name, Ecoli_coverage_average_bed.csv


Let’s copy Ecoli_coverage_average_bed.csv file from flux shared directory to your desktop using ‘
’. ‘
’ stands for secure copy and is used for securely transferring files between remote host/server(flux) and your local computer system. (Both directions)
scp username@flux-xfer.arc-ts.umich.edu:/scratch/micro612w21_class_root/micro612w21_class/shared/Ecoli_coverage_average_bed.csv ~/Desktop/
Note: You can use your choice of folder/path to copy the file instead of  “~/Desktop/”


Drag and drop this Ecoli_coverage_average_bed.csv to your local system using cyberduck.

Now, Fire up R console or studio and import the file (Ecoli_coverage_average_bed.csv) using any type of data import functions in R (read.table, read.csv etc.) 

Hint: The file is comma-separated and contains header line (“bin,Average_coverage”) so use appropriate parameters while importing the file

Once the data in file is imported into R object, you can plot the column Average_coverage as a time series plot to assess the coverage of your mapped reads across genome.

Note: A time series plot is a graph that you can use to evaluate patterns and behavior in data over time. Here, we can employ the same plot to see the pattern i.e read depth/coverage at each 1000 bases (represented by bins columns where each bin represents Average number of reads mapped to each 1000 bases in reference genome) using the simplest R function for time series such as [plot.ts]( http://stat.ethz.ch/R-manual/R-devel/library/stats/html/plot.ts.html )

An example plot.ts plot for Ecoli_coverage_average_bed.csv is shown below for your reference.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1aming/plot_1.png)

For advance and more beautiful visualization, ggplot2 can be employed to display the same plot. An example ggplot2 plot for Ecoli_coverage_average_bed.csv is shown below for your reference.

![alt tag](https://github.com/alipirani88/Comparative_Genomics/blob/master/_img/day1aming/plot_2.png)

-->

- Challenge: Now that you know the gene lengths, get the length of the smallest and largest gene. Then get the attribute of the smallest and largest gene. 

<details>
  <summary>Solution</summary>
  
```
# Length of smallest and largest gene
min(gene_lengths) 
max(gene_lengths) 

# Attribute of smallest and largest gene
gff$attribute[gene_lengths == min(gene_lengths)]
gff$attribute[gene_lengths == max(gene_lengths)]

# Or you could use which.min() and which.max()
gff$attribute[which.min(gene_lengths)]
gff$attribute[which.max(gene_lengths)]
```
</details>

What information do you learn about gene lengths in this genome?

Submit Variant Calling Job
--------------------------

Before we go on a break, we will run a variant calling job that will run all the standard variant calling commands on a sample that we will explore in today's afternoon session. The script will run all necessary commands associated with variant calling in an automated fashion. This will let us give ample time to explore the commands that are involved in each of the steps and explore the results that the script generates. 

We will come back later to the script to understand some of the basics of shell scripting and how different commands can be tied together to run a standard process on a bunch of samples.


- Go to your class home directory (use your wd shortcut!)
- Execute the following commands to copy files for this afternoon’s exercises to your home directory:

```

cp -r /scratch/micro612w21_class_root/micro612w21_class/shared/data/day1pm/ ./

```

We will be using sequencing reads from an Illumina-sequenced *Klebsiella pneumoniae* genome (sample PCMP_H326) as an input for these exercises. This sample, isolated from a hospitalized patient, is resistant to colistin, an antibiotic of last resort. We are interested in seeing if we can identify any mutations in the PCMP_H326 genome that could explain why this sample is resistant to colistin. Colistin resistance can arise through various mutations (see [this review](https://www.frontiersin.org/articles/10.3389/fmicb.2014.00643/full)). To narrow our initial search, we will specifically look for mutations that inactivate the [mgrB gene](https://aac.asm.org/content/58/10/5696), a negative regulator of the PhoPQ two-component signalling system. 
 
Change directory to the variant calling folder inside day1pm directory and list all the files to search variant_call.sh script.

```
d1a

cd variant_calling/

#or

cd /scratch/micro612w21_class_root/micro612w21_class/username/day1pm/variant_calling/

ls variant_call.sh
```

Try running the script with help menu and check all the inputs that is required by the script to run variant calling.

```

./variant_call.sh -h

```

USAGE:
variant_call.sh forward_paired reverse_paired reference_genome output_directory basename [-h] -- A simple shell script to run Variant Calling steps on a pair of fastq reads.

The script requires following positional arguments as input to call variants:
1. Forward Paired end reads
2. Reverse Paired end reads
3. Path to Reference Genome Fasta file
4. Output Directory Path
5. Analysis Base name to store result files with this prefix.

The day1pm directory also contains a slurm script called variant_call.sbat. We will run  variant_call.sh on the Great Lakes Cluster using the following command (you will see this command in the variant_call.sbat). 
```

./variant_call.sh PCMP_H326_R1.fastq.gz PCMP_H326_R2.fastq.gz KPNIH1.fasta ./ PCMP_H326_ 1>variant_call.log 2> variant_call.err

```

Change the EMAIL_ADDRESS section (#SBATCH --mail-user=) of the slurm script to your email address using your favorite text editor (we learned nano in the pre-workshop).

```

nano variant_call.sbat

```

Once you are done editing the slurm script, you can go ahead and submit the job. Make sure you are submitting the job from variant_calling folder and you have activated the conda environment. We will go through each of the variant calling result steps folder and explore the results in afternoon session. 

```
conda activate micro612

sbatch variant_call.sbat 

```


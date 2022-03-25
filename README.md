# RNAfastqtoBigWig
Converts fastq.gz files into bigwig tracks for visualization on UCSC browser or IGV.

Author: Mrutyunjaya Parida, David Price Lab, UIOWA

## Usage:
RNAfastqtoBigWig runs on Python 2.7+. It is a linux based, multi-thread capable, Paired-end Next Generation Sequencing (NGS) data analysis program with a command line interface.
It checks for 12 parameters in the user input. If the number of parameters are less than 12 the program displays the following usage example and parameter description prior to exiting the run. This program is written to accomodate the fastq sample file structures of PriceLab. We receive 2 fastq files for each sample R1 (forward), and R2(reverse).To download RNAfastqtoBigWig use this link: https://drive.google.com/file/d/1i7r7ai-4SJgQ13L2_lFDWRiisKDtasR7/view?usp=sharing. The program folder contains installations of samtools, bedtools, dedup, trim_galore, and bedGraphToBigWig. Please run RNAfastqtoBigWig from the program folder. This program was published in https://pubmed.ncbi.nlm.nih.gov/35048979/.

```
python RNAfastqtoBigWig.py <URL> \
                 <fastq folder> \
                 <sample #'s> \
                 <UMI length> \
                 <min insert> \
                 <max insert> \
                 <# of threads> \
                 <bowtie index> \
                 <chr size file> \
                 <genome assemblies> \
                 <Spike-In> \
                 <sample key> \
                 <dwnld>
                 
Example run: python  RNAfastqtoBigWig www.PRO-Seqdata.com /home/xyz-user/xyz-fastq-folder 1-10 4 26 608 8 \
/home/xyz-user/genome-hg38-JQCY02.1 /home/xyz-user/genome-chrom.sizes hg38,JQCY02.1 JQCY02.1 samplekey.csv yes
```
Note: Here, I am using the moth genome(JQCY02.1) as my Spike-In. The genome-hg38-JQCY02.1 index consists of both the human (hg38) and the Spike-In (JQCY02.1) genomes. 
To create a bowtie index like this I combined the human and moth genomes into one fasta file and ran bowtie-build on this combined.fasta file.
```
bowtie-build reference_sequence.fasta index_name
Example run: bowtie-build /home/xyz-user/genome-hg38-JQCY02.1.fa /home/xyz-user/genome-hg38-JQCY02.1
Parallelized version for faster run: bowtie-build --threads 20 /home/xyz-user/genome-hg38-JQCY02.1.fa /home/xyz-user/genome-hg38-JQCY02.1 (If you have version 1.2.1 or higher and a multithreading library installed).

```
I used a minimum insert length of 26bp and maximum insert length of 608bp because the UMIs at both ends of a read is 4bp in length. After trimming the UMIs my goal is retain all fragments between 18-600bp for further analysis. Please consider the strategy suitable for your sequencing library.

### Parameter description:
```
URL: provide a link to download the data.
fastq folder: provide a path/foldername to download all the fastq files.
sample #'s: <int> provide the sample numbers such as 1-10 or 1-5,7-8 or for single sample 6-6.
UMI length: <int> provide the length of UMI sequence such as 8 or 4.
min insert: <int> provide the minimum length of an insert.
max insert: <int> provide the maximum length of an insert.
threads: <int> provide the number of threads. For example, if there are 80 threads available and 10 samples to process then you may assign atmost 8 threads for each sample.
bowtie index: provide the path to the combined index of only the genomes used in the sequencing. If you are trying to build a new index use bowtie-build.
chr size file: provide the path to the combined chromsome size fasta file of only the genomes used in the sequencing.
               If you are trying to generate a chromsome size file then do following commands:
               samtools faidx combined.genome.fa
               cut -f1,2 combined.genome.fa.fai > combined.genome.chrom.sizes .
genome assembly: provide a comma separated list of genome assemblies used in the sequencing such as hg38,JQCY02.1.
Spike-In: provide the name of the genome assembly used as Spike-In in the sequencing such as JQCY02.1 or simply mention no. 
sample_key: provide sample key in a .csv format where sample#'s and sample names are separated by a comma as follows:
            Sample1,Control1
            Sample2,Control2
            Sample3,Exp1
            Sample4,Exp2
            or simply mention no.
 dwnld: <yes/no> first a user must select this option as "yes" for the program to download the data. If the program downloads the data but the sample names are not according to         the program requirements then program will show errors and exit. Please fix the sample names based on the instructions provided below. Please write "no" for this option         when trying to re-run this program.
```
## Requirements:
Python libraries: ``` joblib, and glob. ```

Software requirements: ``` samtools(v1.7), bedtools(v2.26.0), bowtie(1.2.2), trim_galore(0.6.0), and bedGraphToBigWig(v4). ```

This program is designed to run on paired end data only. Sample file names must follow the following format and order:

Format:
```
Sample#_lane#_yearmonthdate000_Sequencing#_Lane#_Read#_001.fastq.gz
```
Order:

Sample1_lane1_20200324000_S1_L001_R1_001.fastq.gz (forward)

Sample1_lane1_20200324000_S1_L001_R2_001.fastq.gz (reverse)

We use the --small_rna option in trim_galore program to remove the adapter sequences. But they can be changed as per the users preference in the TRIM function.

Samples sequenced in 2 or more lanes are automatically merged into one alignment file as long the lane #'s are mentioned in the sample file name in the above format.

If filenames are named such as "Sample_#" instead of "Sample#" the program will produce errors and exit. If the lane#'s are not present after the Sample# separated by an underscore _ the program may produce errors and exit. If the programs exits, a user can rerun the program after meeting the requirements of the program. If required a "#" symbol can be added to the beginning of the following lines (L) to avoid repeating the following commands during the data workup process: 
```
L82 --> trim the data.
L121 --> align the data.
L145,L149,L153,L157 --> dedup the data.
```
After deduping the program normalizes data using Spike-In if mentioned or not and makes bigwig format tracks for visualization on the UCSC genome browser.

### Output:
A BIGWIG folder is created where bigwig files for each sample can be found. Bigwig files can be loaded onto Integrative Genomics Viewer (IGV) to visualize the number of fragments aligned to any genomic position. The bigwig files can also be used to make a UCSC genome browser session after uploading the tracks from the tracks.txt file in the BIGWIG folder. The bigDataUrl=http://xyz-webserver-address.edu/ is a made-up web server address which can be replaced with a real one.

Based on the example run the BIGWIG folder should be under /home/xyz-user/fastq-folder/MAPPED/BIGWIG.

# RNAfastqtoBigWig
Converts fastq.gz files into bigwig tracks for visualization on UCSC browser or IGV.

Author: Mrutyunjaya Parida, David Price Lab, UIOWA

## Usage:
RNAfastqtoBigWig runs on Python 2.7+. It is a linux based, multi-thread capable, Paired-end Next Generation Sequencing (NGS) data analysis program with a command line interface.
It checks for 12 parameters in the user input. If the number of parameters are less than 12 the program displays the following usage example and parameter description prior to exiting the run.
```
python RNAfastqtoBigWig <URL> \
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
                 <sample key>
                 
Example run: python  RNAfastqtoBigWig www.PRO-Seqdata.com /home/xyz-user/xyz-fastq-folder 1-10 4 18 600 10 \
/home/xyz-user/genome-bowtie-index /home/xyz-user/genome-chrom.sizes hg38,JQCY02.1 JQCY02.1 samplekey.csv
```
Here, I am using the moth genome(JQCY02.1) as my Spike-In. The genome-bowtie-index consists of both the human (hg38) and the Spike-In (JQCY02.1) genomes. 
To create a bowtie index like this I combined the human and moth genomes into one fasta file and ran bowtie-build on this combined.fasta file.

### Parameter description:
```
URL: provide a link to download the data.
fastq folder: provide a path/foldername to download all the fastq files.
sample #'s: <int> provide the sample numbers such as 1-10 or 1-5,7-8 or for single sample 6-6.
UMI length: <int> provide the length of UMI sequence such as 8 or 4.
min insert: <int> provide the minimum length of an insert.
max insert: <int> provide the maximum length of an insert.
threads: <int> provide the number of threads. For example, if there 80 threads available and 10 samples to process then you may assign atmost 8 threads for each sample.
bowtie index: provide the path to the combined index of only the genomes used in the sequencing. If you are trying to build a new index use bowtie-build.
chr size file: provide the path to the combined chromsome size fasta file of only the genomes used in the sequencing.
               If you are trying to generate a chromsome size file use samtools faidx combined.genome.fa
               and cut -f1,2 combined.genome.fa.fai > combined.genome.sizes.
genome assembly: provide a comma separated list of genome assemblies used in the sequencing such as hg38,JQCY02.1.
SpikeIN: provide the genome assembly name used as SpikeIN in the sequencing such as JQCY02.1 or simply mention no. 
sample_key: provide sample key in a .csv format where sample#'s and sample names are separated by a comma or simply mention no.
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

Users must provide a path to their own trim_galore,samtools, bedtools, bowtie, and bedGraphtoBigWig installations inside this program.

Adapter sequences are hard coded but can be changed as per the users preference in the TRIM function.

Samples sequenced in 2 lanes are automatically merged into one alignment file as long the lane #'s are mentioned in the sample file name following the above format.

If sample numbers are mentioned such as Sample_# the program will produce errors and exit.

### Output:
A BIGWIG folder is created where bigwig files for each sample can be found. Bigwig files can be loaded onto Integrative Genomics Viewer (IGV) to visualize the number of fragments aligned to any genomic position.

Based on the example run the BIGWIG folder should be under /home/xyz-user/fastq-folder/MAPPED/BIGWIG.

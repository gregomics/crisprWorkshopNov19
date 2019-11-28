# CrispR cas9 validation Hands on Workshop University of Otago

## Set up

Processing for the edition validation will be done on the cloud.
To do so, we will need to use several tools that needs to be available on your computer.

The standard protocol for logging into a modern UNIX server is through using a Secure SHell (SSH) client.

Here are the different flavors of the computer:

  * Linux
The default Unix Shell for Linux operating systems is usually Bash. On most versions of Linux, it is accessible by running the (Gnome) Terminal or (KDE) Konsole or xterm, which can be found via the applications menu or the search bar. On Linux machine, ssh is installed by default.

  * macOS
For a Mac computer running macOS Mojave or earlier releases, the default Unix Shell is Bash. For a Mac computer, ssh is installed by default.

  * Windows
A free SSH client available for MS Windows computers is [PuTTY](http://www.putty.org/)


##  How to connect to the cloud

For this workshop we will use Cloud Catalyst which is kiwi's own cloud computing company.

## Data description

Subset of sequencing data from (Kindly provided by) Bicknel's lab (University of Otago).
Amplicon sequencing of ORC1 gene:

  * Forward PCR primer: TTACATGTTTTGTCCTCATTTGC
  * Reverse PCR primer: GCAAGCATCCCATCTCAATATC (revcomp: GATATTGAGATGGGATGCTTGC)
  * sgRNA: GGGCGGGAGCCTCATCAAGAGG


Fastq sequencing data can be found /data/ : 

  * Injected_R1.fastq.gz
  * Injected_R2.fastq.gz
  * UnInjected_R1.fastq.gz (negative control)
  * UnInjected_R2.fastq.gz (negative control)


## Edition workflow

### Quality control of sequencing data


create directory to store qc:

```
mkdir qc
```
run quality control:

```
fastqc -o qc/ /data/Injected_R1.fastq.gz

```

### Trimming bad quality or adaptor content

To trim adaptor sequences as well as bad quality data, we will use cutadapt.

  * -q quality score (Phred)
  * -B -b fasta file containing adaptors we want to remove
  * -o R1 trimmed file
  * -p R2 trimmed file (paired)

```
mkdir trimmed

```

```
cutadapt -B file:/data/adapt.fa -q 30 -b file:/data/adapt.fa -O 5 --discard -o trimmed/Injected_trimmed_R1.fastq -p trimmed/Injected_trimmed_R2.fastq /data/Injected_R1.fastq.gz /data/Injected_R2.fastq.gz
```


### Demultiplexing

Demultiplexing is to distribute sequencing data according their Index/Barcode.
What do we need?
List of Barcodes used in Forward (R1) and Reverse (R2)
Instead of using a list we organise data using FastA format:
  * barcode_fwd.fa :
```
>fwd_primer1
TTACATGTTTTGTCCTCATTTGC
```
  * barcode_rev.fa :
```
>rev_primer1
GCAAGCATCCCATCTCAATATC
```

First we need to create a directory where the demultiplexed data will be stored:

```
mkdir demux

```

```
cutadapt -e 0 --no-indels -g file:/data/barcode_fwd.fa -G file:/data/barcode_rev.fa -o demux/{name1}-{name2}_R1.fastq.gz -p demux/{name1}-{name2}_R2.fastq.gz trimmed/Injected_trimmed_R1.fastq trimmed/Injected_trimmed_R2.fastq

```
Note: several files are now available.

Same demultiplexing for the UnInjected sample.

At the end of this step, the sequences are then cleaned and demultiplexed.

### Merging Paired end reads

R1 (forward) and R2 (reverse) are sequences coming from the same DNA fragment.
If they overlap they can be merged and recreate a full length amplimer sequence.
To do merge the paired ends, there is a variety of tools. 

First, let's create a directory to store the merged data:

```
mkdir mergedPE
```
Now let's merge the paired end to recreate the amplimer.

```
flash -t 1 -O demux/fwd_BC1-rev_BC1_R1.fastq.gz demux/fwd_BC1-rev_BC1_R2.fastq.gz -o merged -d mergedPE
```
Flash has merged the paired end reads and create the following file: merged.extendedFrags.fastq

### Alignment on Target

To align the merged reads on the target, the target must first be indexed.
To avod permission issues, I will need to copy it in a new directory.

```
mkdir ref
cp /data/ref.fa ref/
```
Index the reference:

```
bwa index ref/ref.fa

```
Then we need to create a directory where we store the alignment.

```
mkdir aln
```

```
bwa mem -t 1 ref/ref.fa mergedPE/merged.extendedFrags.fastq | samtools view -bS - > aln/injected_aln.bam

```

### Visualisation of the alignment:

Prior to load the alignment, the bam file must be sorted:

```
cd aln
```

```
samtools sort injected_aln.bam injected_aln_sorted
```
it will generate the sorted file: injected_aln_sorted.bam

This file needs to be indexed as well :

```
samtools index injected_aln_sorted.bam

```
Now the alignment is ready for visualisation using IGV.

```
igv
```

After launching this command, you should see a window appearing on your local screen.

  * Click on tab genomes then load Genome from file
  * select the reference fasta file
  * Click on tab Files then Load from File ...

### Exploration of "quick" estimation

use samtools mpileup to stack the alignemnt

```
samtools mpileup -C50 -gf ../ref.fa  injected_aln_sorted.bam > injected_mpileup.vcf.gz
```

```
bcftools view -cgNv injected_mpileup.vcf.gz > injected_editions.vcf
```






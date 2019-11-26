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
  * Reverse PCR primer: GCAAGCATCCCATCTCAATATC
  * sgRNA: GGGCGGGAGCCTCATCAAGAGG

Fastq sequencing data can be found: 

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
fastqc -o qc/ Injected_R1.fastq.gz

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


```
cutadapt -e 0.15 --no-indels -g file:barcode_fwd.fa -G file:barcode_rev.fa -o {name1}-{name2}_R1.fastq.gz -p {name1}-{name2}_R2.fastq.gz Injected_R1.fastq.gz Injected_R2.fastq.gz

```


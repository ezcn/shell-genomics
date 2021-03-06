
## Learning Objectives

* Download files form FTP repositories
* Working with compressed files
* Working with FASTA files

## Lesson

In this lesson, we will learn how to download data in [fasta format](https://en.wikipedia.org/wiki/FASTA_format) from the web and how to manipulate them using basic shell commands.
We will use human [non-coding RNAs](https://en.wikipedia.org/wiki/Non-coding_RNA) (ncRNA) as a sample dataset.

## Exercises
### 1 - Downloading data

The data we need are on the [Ensembl](http://www.ensembl.org/index.html) FTP site, a repository of all publicly available files from the project.

There is an [html interface](http://www.ensembl.org/info/data/ftp/index.html) of the last release.  In this interface, we need to locate the link to the FAST file of human ncRNA on the page where it should be easy to locate the link to the FASTA file of the ncRNA

*or*
 We need to navigate the repository .
You can navigate the public [FTP](ftp://ftp.ensembl.org/pub/) repository to find the file with all the sequences of human ncRNA (a little help: follow  last\_release>FASTA>homo_sapiens>ncrna)

*in both cases*

we need to copy the full address that will look like this one:

```
ftp://ftp.ensembl.org/pub/release-81/fasta/homo_sapiens/ncrna/Homo_sapiens.GRCh38.ncrna.fa.gz
```

We will use this address as an argument to the command [wget](http://www.gnu.org/software/wget/manual/wget.html#Overview) to retrieve the data. Type in your terminal the command line below to download the file on your local machine.

```
$ wget <web address of the fasta file>
```

### 2 - Looking at the data

The file we just downloaded is compressed and in the [gzip](https://www.gnu.org/software/gzip/manual/gzip.html) format. To see how big it is  use the *-l* and *-h* option of *ls*:

```
$ ls -lh

-rw-rw-r-- 1 usr usr 8,4M ago 13 15:30 Homo_sapiens.GRCh38.ncrna.fa.gz
```

We can either work with the gzipped file or unzip it before, however because data sets can be really big it is probably worth working with the gzipped file.

We can perform many operations on the compressed file like we do on uncompressed files using "z" commands. We can view the file content using:

```
$ zless filename.gz
$ zmore filename.gz
```
or combine them using [pipe](https://en.wikipedia.org/wiki/Pipeline_%28Unix%29)(|):

```
$ zcat filename.gz | more
$ zcat filename.gz | less
```

For any further analysis, we need to understand how the file is structured. We can take a peak at a bit of it.  Can you identify how a single ncRNA is described using the example below?

```

>ENST00000517275 ncrna:known chromosome:GRCh38:2:231757887:231757950:1 gene:ENSG00000253084 gene_biotype:snRNA transcript_biotype:snRNA
GTGCTTGCTTCGGCAGCACATATACTAAAATTGGAATGATACAGAGAAGATGAGCATGGC
CCCT

```

As you can see each ncRNA is described by at least two lines, a header (starting with '>') and at least one line of DNA sequence.  To learn what information is included in the header, we can download and read the [README companion file](ftp://ftp.ensembl.org/pub/release-81/fasta/homo_sapiens/ncrna/README) located in the same ftp directory of the fasta file we just downloaded.

```

>ENST00000347977 ncrna:miRNA chromosome:NCBI35:1:217347790:217347874:-1 gene:ENSG00000195671 gene_biotype:ncRNA transcript_biotype:ncRNA
   ^             ^     ^     ^                                          ^                    ^                           ^
   ID            |     |  LOCATION                            GENE: gene stable ID       GENE: gene biotype           TRANSCRIPT: transcript biotype
                 |   STATUS
              SEQTYPE

```


### 3 -  Extracting information from the data

We want to ask the following questions:

* a. How many ncRNA are there in humans?
* b. How many ncRNA are located on chromosome 18?
* c. How many ncRNA are micro-RNA (miRNA)?
* d. How many different gene_biotype tags there are? How many are there of each tag?
* e. To which ncRNA does the the sequence "CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT" belong?


Let's answer these questions!

#### *Question a.*

**How many ncRNA are there in humans?**

We will take advantage of the file structure. We know that each ncRNA has a heading line that starts with a ">".  We can search for ">" inside the compressed file using [zgrep / zegrep](http://www.gnu.org/software/grep/manual/grep.html) to count how many ">" there are using the option *-c*. Alternatively, we can also combine grep with the command [wc](https://en.wikipedia.org/wiki/Wc_%28Unix%29) and the *-l* option to count the number of lines.
```
$ zgrep -c ">" Homo_sapiens.GRCh38.ncrna.fa.gz
37197
```
OR
```
$ zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz   | wc -l
37197

```

#### *Question b.*

**How many ncRNA are located on chromosome 18**

To answer this question need to look closely at the heading structure. Let's visualize some headings to see what they have in common/different. We will use the command [head](https://en.wikipedia.org/wiki/Head_%28Unix%29) that shows only the first ten lines.

```

$ zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz    | head
>ENST00000629478 proj_ncrna:known chromosome:GRCh38:CHR_HG1832_PATCH:210374154:210374267:-1 gene:ENSG00000281499 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000516494 proj_ncrna:known chromosome:GRCh38:CHR_HG2128_PATCH:67546651:67546754:1 gene:ENSG00000252303 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000627793 proj_ncrna:novel chromosome:GRCh38:CHR_HG126_PATCH:72577040:72577203:1 gene:ENSG00000281822 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000391058 ncrna:known chromosome:GRCh38:1:61852499:61852602:1 gene:ENSG00000212360 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000613107 ncrna:novel chromosome:GRCh38:22:15273855:15273961:1 gene:ENSG00000276138 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000517275 ncrna:known chromosome:GRCh38:2:231757887:231757950:1 gene:ENSG00000253084 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000384655 ncrna:known chromosome:GRCh38:4:53265461:53265567:-1 gene:ENSG00000207385 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000384433 ncrna:known chromosome:GRCh38:15:64671263:64671369:1 gene:ENSG00000207162 gene_biotype:snRNA transcript_biotype:snRNA
>ENST00000362613 ncrna:known chromosome:GRCh38:9:83596444:83596584:-1 gene:ENSG00000199483 gene_biotype:snRNA transcript_biotype:snRNA

```


The header contains the information of the chromosome in the "location" and we can use it to grep and count ncRNA of chromosome 18:


```
$ zgrep -c chromosome:GRCh38:18 Homo_sapiens.GRCh38.ncrna.fa.gz
843
```
OR (longer but equivalent)
```
$ zcat Homo_sapiens.GRCh38.ncrna.fa.gz  | grep ">" |  grep chromosome:GRCh38:18 |  wc -l
843

```
#### *Question c.*

**How many ncRNA are micro-RNA (miRNA)?**

If any information exists on miRNA we can certainly use grep and count it:

```
$ zgrep -c  miRNA Homo_sapiens.GRCh38.ncrna.fa.gz
4554

```

If we visualize part of our search we will see that the information on miRNA is in two fields: the gene_biotype and the transcript_biotype.

```
$ zgrep -c  miRNA Homo_sapiens.GRCh38.ncrna.fa.gz | less

```

#### *Question d.*

**d. How many different gene_biotype tags there are? How many are there of each tag?**

To answer this question we want to extract the field which contains the "gene\_biotype" information from the heading using the [cut](https://en.wikipedia.org/wiki/Cut_%28Unix%29) command. This is the fifth field in the heading. We can specify the fields are delimited by spaces with the option *-d* and the field number as 5 with the option *-f*:

```
$ zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz | cut -d " " -f5 | more

```

Now we want to know how many [uniq](https://en.wikipedia.org/wiki/Uniq) biotypes are there and count how many ncRNA belong to each unique type using the option *-c*:

```
$ zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz | cut -d " " -f5 | uniq -c
2036 gene_biotype:snRNA
 563 gene_biotype:rRNA
   1 gene_biotype:vaultRNA
  20 gene_biotype:sRNA
   2 gene_biotype:Mt_rRNA
1019 gene_biotype:snoRNA
4554 gene_biotype:miRNA
  22 gene_biotype:Mt_tRNA
  51 gene_biotype:scaRNA
13742 gene_biotype:lincRNA
   1 gene_biotype:macro_lncRNA
1001 gene_biotype:sense_intronic
   8 gene_biotype:ribozyme
  32 gene_biotype:3prime_overlapping_ncrna
 337 gene_biotype:sense_overlapping
3361 gene_biotype:processed_transcript
10447 gene_biotype:antisense

```

To be more elegant we can get rid of the "gene\_biotype:" word. We can use the substitute command (s) of the [sed](http://www.gnu.org/software/sed/manual/sed.html) editor:  

```
zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz | cut -d " " -f5  | uniq -c  | sed 's/gene_biotype://'


```

Or we can apply cut twice:

```
$ zgrep ">" Homo_sapiens.GRCh38.ncrna.fa.gz | cut -d " " -f5  | cut -d ":" -f2 |  uniq -c

```
Try the commands in your terminal, what do you notice?


#### *Question e.*

**To which ncRNA does the the sequence "CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT" belong?**

We have already learned to use `grep` to search for text in a file.  However, in this case we want to see a few more lines around what we are searching for and using `grep` as we did before will not be informative:

```
$ zcat Homo_sapiens.GRCh38.ncrna.fa.gz  |  grep  CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT
CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT

```
We can see few lines before the match using the *-B* option followed by the number of lines to catch the heading:

```

$ zcat Homo_sapiens.GRCh38.ncrna.fa.gz  |  grep -B2  CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT
>ENST00000517119 ncrna:known chromosome:GRCh38:13:30281765:30281869:1 gene:ENSG00000252928 gene_biotype:snRNA transcript_biotype:snRNA
GTGTCTGCATTGGCAGCACATACACTAAAATTGGAATAACACAGAGAAGACCAGCATGTT
CCCTGCACGAGATGCACACAAACTCCTGGAGTGTTCTGTATTTTT

```
Similarly the *-A n* option shows *n* line after the one we are grepping.

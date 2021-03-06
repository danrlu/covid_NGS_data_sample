# Key file types in genomic epidemiology workflow 
(Below is with Illumina sequencing. Nanopore is slightly different)

| data analysis | step 1 | step 2 | step 3 | step 4 | 
|---|-----|------|-------|--|
| file content | raw sequencing reads | reads mapped to reference genome | consensus genome | tree |
| file type | fastq file | bam file | fasta file | json file |

<br>

## Read file: `sample.fastq` or `sample.fq`

- The raw file generated by the sequencer is usually in FASTQ format. 

- FASTQ file stores sequencing reads.

- Below is 1 read in the FASTQ file. 
```
@A00111:640:HVVF3DRXX:1:2101:30038:1063 1:N:0:CTCTGATTGCCT+TTGGTCTGCCGA
CTATCATTAACTGAATCCATAGGTTAATGAGGCGAACCGGGGGAACTGAAACATCTAAGTACCCCGAGGAAAAGAAATCAACCGAGATTCCCCCAGTAGCGGCGA
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFF:FFF:FFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFF:FFFFFFFF
```
-  
   + Row 1: read name starting with @ and followed by arbituary info based on sequencer and users.
   + Row 2: actual nucleotide sequence of the read
   + Row 3: usually a + sign
   + Row 4: quality (aka confidence) of each base in ASCII. It has to be the same length as row 2.

- So each read is represented by 4 rows. If one FASTQ file contains 1000 reads, it will have 4 x 1000 = 4000 rows.

- If the sequencing is done in single-end mode, it will generate 1 FASTQ file for each sample. If it is done in paired-end mode, it will generate 2 FASTQ files for each sample, usually named as `sample_R1.fastq` and `sample_R2.fastq`.

- A real [example from SRA](https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR13690103), the NIH repository for sequencing read data. See the `Data-access` tab,  `Original Format` section. 

https://sra-pub-sars-cov2.s3.amazonaws.com/sra-src/SRR13690103/RR065e_01847_W-R175-A1_B01_covid_1.fq.gz.1

https://sra-pub-sars-cov2.s3.amazonaws.com/sra-src/SRR13690103/RR065e_01847_W-R175-A1_B01_covid_2.fq.gz.1  

- Further reading: https://support.illumina.com/bulletins/2016/04/fastq-files-explained.html

<br>

## Mapped read file: `sample.bam`

- To characterize an individual genome, aka to identify the mutations, we need to "compare" the sequencing reads originated from the sample genome to the reference genome.

- Such "comparison" results in a [BAM file](https://samtools.github.io/hts-specs/SAMv1.pdf) that contains comprehensive information of how each sequencing read matches or differs from the reference genome. 

- BAM files are binary to reduce file size, so they require special tools to open and read. 

- This is an essential intermediate step in the workflow but BAM files are usually not directly used in genomic epidemiology investigations.

- A more concise VCF file that only contains the information about mutations can further be generated from BAM files. 

<br>

## Consensus genome: `sample.fasta` or `sample.fa`

- With a reference genome and the information in a BAM file , aka a reference genome and how the sample genome differs from it, one can re-constitute the full sample genome that was sequenced. 

- The resulting genomic sequence (the 'consensus genome') is stored in FASTA files.

- A FASTA file stores nucleotide sequences or amino acid sequences.

- Below is an example FASTA file with 2 very small hypothetical genomes (COVID genomes are 30,000 bases long which will take some scrolling):

```
>sample1_name
NNNNNNNNNCTTGTAGATCTGTTCTCTAAACGAACTTTAAAATCTGTGTGGCTGTCACTC
GGCTGCATGCTTAGTGCACTCACGCAGTATAATTAATAAC

>sample2_name
NNNNNNNNNCTCTTGTAGATCTGTTCTCTAAACGAACTTTAAAATCTGTGTGGCTGTCAC
TCGGCTGCATGCTTAGTGCACTCACGCAGTATAATTAATA
```
-
  + Each sequence starts with `>` followed by the name of the sequence.
  + Nucleotide positions can have [more than 1 nucleotide composition](https://genome.ucsc.edu/goldenPath/help/iupac.html.). For example the beginning of example genomes are filled with N, which means they can be any nucleotide, due to low confidence in what these bases should be. 

- FASTA is the file people submit to GISAID or NCBI GenBank. For example [here](https://www.ncbi.nlm.nih.gov/nuccore/MN908947.3?report=fasta) is one of the reference COVID genome.

<br>

## Phylogeny tree: `all_samples.json`

- With genomes of multiple samples, one can build a phylogeny tree based on the similarity and differences in their genomes. 

- There are many file types used to store phylogeny trees. Nextstrain use JSON files. These JSON files contain both structure of the tree and many useful metadata. 

- Here is [what Nextstrain do with the FASTA](https://docs.nextstrain.org/projects/augur/en/stable/index.html). The Augur pipeline process genome data, build phylogeny trees and write its output in JSON files, which is then the input for Auspice to display the phylogeny tree. 

- A [very simple JSON](https://github.com/czbiohub/covidtracker/raw/master/auspice/covidtracker_pawnee-examples.json) that one can drag and drop to https://auspice.us/

- [Example Nextstrain JSON](https://github.com/nextstrain/augur/blob/master/augur/data/schema-export-v2.json).


# Key file types in genomic epidemiology sequencing workflow
(Below is for Illumina sequencing. Nanopore sequencing is slightly different)

<br>

| data analysis | step 1 | step 2 | step 3 | step 4 | 
|---|-----|------|-------|--|
| file content | raw sequencing reads | reads mapped to reference genome* | sample genome | tree |
| file type | FASTQ file | BAM file | FASTA file | JSON file |

\* The reference genome is the original COVID genome sequence from Wuhan.

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

- Example FASTQ [R1](https://github.com/czbiohub/sc2-illumina-pipeline/raw/master/test_data/sample1_artic_R1.fq.gz) and [R2](https://github.com/czbiohub/sc2-illumina-pipeline/raw/master/test_data/sample1_artic_R2.fq.gz)

- A real [example from SRA](https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR13690103), the NIH repository for sequencing read data. See the `Data-access` tab,  `Original Format` section. 

<br>

## Mapped read file: `sample.bam` or `sample.sam`

- To characterize an individual genome, i.e. to identify the mutations, we need to compare the sequencing reads originated from the sample genome to the reference genome.

- Such a comparison results in a [BAM file](https://samtools.github.io/hts-specs/SAMv1.pdf) that contains comprehensive information about how each sequencing read matches or differs from the reference genome. 

- BAM files are binary to reduce file size, so they require special tools to open and read. The non-binary, human readable version of BAM files are SAM files but they are much larger in size.

- BAM files are an essential intermediate step in the workflow, but they are usually not directly used in genomic epidemiology investigations.

- A more concise [VCF file](https://samtools.github.io/hts-specs/VCFv4.2.pdf) that only contains the information about mutations can further be generated from BAM files. 

<br>

## Consensus genome: `sample.fasta` or `sample.fa`

- Knowing a reference genome and how the sample genome differs from it (stored in a BAM file), one can re-constitute the full sample genome that was sequenced. 

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

- FASTA is the file people submit to GISAID or NCBI GenBank. For example [here](https://www.ncbi.nlm.nih.gov/nuccore/MN908947.3?report=fasta) is one of the reference COVID genomes.

<br>

## Phylogeny tree: `all_samples.json`

- With genomes of multiple samples, one can build a phylogeny tree based on the similarity and differences in their genomes. 

- There are many file types used to store phylogeny trees. Nextstrain uses a specialized JSON format. These JSON files contain both the structure of the tree and a lot of useful metadata. 

- [Here] is what Nextstrain does with the genomes(https://docs.nextstrain.org/projects/augur/en/stable/index.html). In short, the Augur pipeline processes genome data, builds phylogeny trees, and write its output in JSON files, which is then the input for Auspice to display the phylogeny tree. 

- A [very simple JSON](https://github.com/czbiohub/covidtracker/raw/master/auspice/covidtracker_pawnee-examples.json) that one can drag and drop to https://auspice.us/

- [Example Nextstrain JSON](https://github.com/nextstrain/augur/blob/master/augur/data/schema-export-v2.json).


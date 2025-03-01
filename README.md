## Obtaining and installing StringTie

In order to build StringTie from this GitHub repository
the following steps can be taken:
 
```
git clone https://github.com/mpertea/stringtie2
cd stringtie2
make release
```

Note that simply running `make` will produce an executable 
which is more suitable for debugging and runtime checking but which can be
significantly slower than the optimized version which is obtained by using 
`make release`.


### Installation of the super-reads module

This optional module can be used to de-novo assemble, align and pre-process
RNA-Seq reads, preparing them to be used as "super-reads" by Stringtie.

Mode detailed information is provided in the SuperReads_RNA/README.md 
Quick installation instructions for this module (assuming the above Stringtie installation 
was completed):

```
 cd SuperReads_RNA
 ./install.sh
```

### Using super-reads with Stringtie

(under construction)

## Running StringTie

Run stringtie from the command line like this:
'''
stringtie [options] <aligned_reads.bam>
'''
The main input of the program is a SAMTools BAM file with RNA-Seq mappings
sorted by genomic location (for example the accepted_hits.bam file produced
by TopHat).

The following optional parameters can be specified (use -h/--help to get the
usage message):
```
 --version : print just the version at stdout and exit
 --conservative : conservative transcriptome assembly, same as -t -c 1.5 -f 0.05
 --rf assume stranded library fr-firststrand
 --fr assume stranded library fr-secondstrand
 -G reference annotation to use for guiding the assembly process (GTF/GFF3)
 -o output path/file name for the assembled transcripts GTF (default: stdout)
 -l name prefix for output transcripts (default: STRG)
 -f minimum isoform fraction (default: 0.01)
 -L use long reads settings (default:false)
 -m minimum assembled transcript length (default: 200)
 -a minimum anchor length for junctions (default: 10)
 -j minimum junction coverage (default: 1)
 -t disable trimming of predicted transcripts based on coverage
    (default: coverage trimming is enabled)
 -c minimum reads per bp coverage to consider for multi-exon transcript
    (default: 1)
 -s minimum reads per bp coverage to consider for single-exon transcript
    (default: 4.75)
 -v verbose (log bundle processing details)
 -g maximum gap allowed between read mappings (default: 50)
 -M fraction of bundle allowed to be covered by multi-hit reads (default:1)
 -p number of threads (CPUs) to use (default: 1)
 -A gene abundance estimation output file
 -B enable output of Ballgown table files which will be created in the
    same directory as the output GTF (requires -G, -o recommended)
 -b enable output of Ballgown table files but these files will be 
    created under the directory path given as <dir_path>
 -e only estimate the abundance of given reference transcripts (requires -G)
 -x do not assemble any transcripts on the given reference sequence(s)
 -u no multi-mapping correction (default: correction enabled)
 -h print this usage message and exit

Transcript merge usage mode: 
  stringtie --merge [Options] { gtf_list | strg1.gtf ...}
With this option StringTie will assemble transcripts from multiple
input files generating a unified non-redundant set of isoforms. In this mode
the following options are available:
  -G <guide_gff>   reference annotation to include in the merging (GTF/GFF3)
  -o <out_gtf>     output file name for the merged transcripts GTF
                    (default: stdout)
  -m <min_len>     minimum input transcript length to include in the merge
                    (default: 50)
  -c <min_cov>     minimum input transcript coverage to include in the merge
                    (default: 0)
  -F <min_fpkm>    minimum input transcript FPKM to include in the merge
                    (default: 1.0)
  -T <min_tpm>     minimum input transcript TPM to include in the merge
                    (default: 1.0)
  -f <min_iso>     minimum isoform fraction (default: 0.01)
  -g <gap_len>     gap between transcripts to merge together (default: 250)
  -i               keep merged transcripts with retained introns; by default
                   these are not kept unless there is strong evidence for them
  -l <label>       name prefix for output transcripts (default: MSTRG)
```

## Input files


StringTie takes as input a binary SAM (BAM) file sorted by reference position. 
This file contains spliced read alignments such as the ones produced by TopHat or HISAT2.
A text file in SAM format should be converted to BAM and sorted using the 
samtools program:
```
samtools view -Su alns.sam | samtools sort - alns.sorted
```
The file resulted from the above command (alns.sorted.bam) can be used 
directly as input to StringTie. 

Any SAM spliced read alignment (a read alignment across at least one junction)
needs to contain the XS tag to indicate the strand from which the RNA that produced
this read originated. TopHat alignments already include this tag, but if you use
a different read mapper you should check that this tag is also included for spliced alignment
records. For example HISAT2 should be run with the `--dta` option in order to tag spliced 
alignments this way. As explained above, the alignments in SAM format should be sorted and
preferrably converted to BAM.

Optionally, a reference annotation file in GTF/GFF3 format can be provided to StringTie. 
In this case, StringTie will check to see if the reference transcripts are expressed in the 
RNA-Seq data, and for the ones that are expressed it will compute coverage and FPKM values.
Note that the reference transcripts need to be fully covered by reads in order to be included
in StringTie's output. Other transcripts assembled from the data by StringTie and not present
in the reference file will be printed as well ("novel" transcripts).


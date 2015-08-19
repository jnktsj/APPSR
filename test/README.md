## Test datasets
This direcotry contains the following files:
###### FASTQ files
* `good.fq`: good quality reads (3′adapter: `TGGAATTC`)
* `poor.fq`: poor quality reads (3′adapter: `CGCCTTGG`)
* `processed.fq`: reads already processed, i.e., reads that do not
  contain any 3′adapters

###### Processed files with APPSR
* `trimmed.fq`: reads in `poor.fq` processed by `adapt-trim`
* `clipped.tab`: reads in `good.fq` clipped by `adapt-clip`
* `clean.fa`: reads in `good.fq` collapsed by `read-collapse`

## Tutorial
### Quality score offset estimation
Using `good.fq`, let's estimate the quality score offset of the FASTQ.

Type:

    $ qual-offset good.fq

Then you will get:

    Sanger/Illumina-1.8+:base=33

### Adapter prediction
With the same FASTQ, `good.fq`, let's predict its adapter sequence.

Type:

    $ adapt-pred good.fq

Then you will get:

    Predicted_3'adapter_1=TGGAATTCTCGGGTGCCAAGGAACTCC

### Quality trimming
This time, let's use `poor.fq` to do quality trimming.

Type:

    $ qual-trim -q 20 poor.fq > [filename]

You can find trimmed reads in `[filename]`. For checking the
consistency, compare your result to `trimmed.fq`.

### Adapter clipping
Let's go back to `good.fq` and clip the 3′adapter.

Type:

    $ adapt-clip -3 TGGAATTC good.fq > [filename]

The first column in the file represents extracted small RNA reads.
For checking the consistency, compare your result to `clipped.tab`.

### Read collapsing
As the last step of the preprocessing, let's collapse reads using the
above adapter-clipping result.

Type:

    $ read-collapse clipped.tab > [filename]

You can get clean non-redundant reads with the counts. For checking
the consistency, compare your result to `clean.fa`.

### Exhaustive adapter search and quality control
You may want to search adapter sequences in an exhanstive manner and
do quality control for your small RNA libraries. Using the three FASTQ
files, `good.fq`, `processed.fq`, and `poor.fq`, let's see how
`adapt-qc` works.

For preparing read mapping, let's make genome index first. Download
human genome sequences. In this example, let's use
[hg38](http://hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/analysisSet/hg38.analysisSet.chroms.tar.gz).
Although you can use any read mapping software packages, we will use
[Bowtie](http://bowtie-bio.sourceforge.net) in this short tutorial.

Here is the genome index preparation:

```shell
# concatenate all chromosomes into one file
cat *.fa > hg38.fa

bowtie-build hg38.fa [index_name]
```

Now let's run `adapt-qc`!

##### Case 1: `good.fq`
With the following command:

    $ adapt-qc "/path_to/bowtie /path_to/genome_index -p [cpu_num] -v0 -k1 -S -f @in > @out" good.fq

You will get:

    Optimal_3'adapter=TGGAATTC
    
    # Report: sampled_reads=100000 (total_reads * 1.00)
    # 3'adapter   reads_extracted   (reads_extracted/sampled_reads)%   reads_mapped   (reads_mapped/sampled_reads)%    params_k:r
      TGGAATTC    95845             95.84                              84409          84.41                            9:1.2;9:1.3;9:1.4;11:1.2;11:1.3;11:1.4
      RAW_INPUT       0              0.00                                  0           0.00                            NO_TREATMENT

##### Case 2: `processed.fq`
When the adapter was already clipped from the reads, you will get:

    Optimal_3'adapter=RAW_INPUT
    
    # Report: sampled_reads=100000 (total_reads * 1.00)
    # 3'adapter   reads_extracted   (reads_extracted/sampled_reads)%   reads_mapped   (reads_mapped/sampled_reads)%   params_k:r
      TAATACTG        0              0.00                              0               0.00                           9:1.2;9:1.4;11:1.3;11:1.4
      TGGCAGTG        0              0.00                              0               0.00                           9:1.3;11:1.2
      RAW_INPUT   98607             98.61                              73773          73.77                           NO_TREATMENT
    # Input reads look already clean!

##### Case 3: `poor.fq`
When the quality of reads in a FASTQ is not good, you will get:

    Optimal_3'adapter=NULL
    
    # Report: sampled_reads=100000 (total_reads * 1.00)
    # 3'adapter   reads_extracted   (reads_extracted/sampled_reads)%   reads_mapped   (reads_mapped/sampled_reads)%   params_k:r
      CGCCTTGG    15661             15.66                              4617           4.62                            9:1.2;9:1.3;9:1.4;11:1.2;11:1.3;11:1.4
      RAW_INPUT       0              0.00                                 0           0.00                            NO_TREATMENT



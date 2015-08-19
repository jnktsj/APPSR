# APPSR
APPSR: automated preprocessing pipeline for meta-analysis of small RNA sequencing data.

(version: 1.0)

## Introduction
APPSR is a suite of tools for preprocessing small RNA sequencing
libraries.  This package enables users to implement an automated
preprocessing pipeline with only a few commands (see
[Examples](https://github.com/jnktsj/test#examples)).

APPSR is composed of the following six programs:
* [`adapt-pred`](https://github.com/jnktsj/test#adapt-pred-3adapter-prediction)
  predicts 3′ adapter sequences in an input FASTQ
* [`adapt-clip`](https://github.com/jnktsj/test#adapt-clip-adapter-clipping)
  clips 5′ or/and 3′ adapter sequences from input reads
* [`adapt-qc`](https://github.com/jnktsj/test#adapt-qc-exhaustive-3adapter-search-and-quality-control)
  search 3′ adapter sequences exhaustively and does quality control
* [`qual-offset`](https://github.com/jnktsj/test#qual-offset-quality-offset-estimation)
  estimates ASCII-encoded quality score offsets
* [`qual-trim`](https://github.com/jnktsj/test#qual-trim-quality-trimming)
  trims low quality bases in input reads
* [`read-collapse`](https://github.com/jnktsj/test#read-collapse-read-collapsing)
  merges identical reads while retaining the counts

## Requirement
APPSR requires Python >=2.5 under a Linux/Unix environment.

## Programs
To see the usage for each program, type:

    $ <program-name> -h

or

    $ <program-name> --help

If the FASTQ files are compressed, decompress the files and pass those
to the program using a pipe. For example:

    $ zcat <FASTQ.gz> | <program-name> -

### `adapt-pred`: 3′ adapter prediction
`adapt-pred` predicts 3′ adapter sequences from input FASTQ.
#### Usage

    $ adapt-pred [options] <fastq>

#### Options
###### -k BP
K-mer length to use to compute k-mer frequency in the input FASTQ.
The default value is 9 nucleotides (nt).
###### -r FLOAT
Cutoff ratio for filtering less frequent k-mers. For each k-mer, a
ratio of the frequency of the most abundant k-mer to the frequency of
a target k-mer will be computed. If a ratio is lower than the cutoff
specified with `-r`, the k-mer with the ratio will be discarded.
###### -a
This option shows other predicted 3′adapter candidates (if any).

### `adapt-clip`: adapter clipping
`adapt-clip` clips either or both 5′ and 3′adapter sequences from
input reads, and extract small RNA inserts.
#### Usage

    $ adapt-clip [options] <fastq>

#### Options
###### -3 SEQ
3′adapter sequence. An input adapter sequence should be equal or
longer than the adapter match length specified with `-l`.  When the
`-s` option is used, the input adapter sequence should be at least one
base longer than the adapter match length.
###### -5 SEQ
5′adapter sequence. An input adapter sequence should be equal or
longer than the adapter match length specified with `-l`.  When the
`-s` option is used, the input adapter sequence should be at least one
base longer than the adapter match length.
###### --cut-3p BP
Cut specified number of bases from 3′ ends. This option can be combined
with the adapter clipping process to trim down specific number of
bases additionally.
###### --cut-5p BP
Cut specified number of bases from 5′ ends. This option can be combined
with the adapter clipping process to trim down specific number of
bases additionally.
###### -l BP
Adapter match length in bp. The default is 7nt.
###### -m BP
Minimum read length in bp. Extracted small RNA reads will be discarded
if the lengths are *shorter* than the specificed length with `-m`. The
default is 16nt.
###### -x BP
Maximum read length in bp. Extracted small RNA reads will be discarded
if the lengths are *longer* than the specificed length with `-x`. The
default is 36nt.
###### -s
Sensitive adapter search with 1 mismatch. When this option is used,
the length of input adapter(s) should be at least one base longer than
the adapter match length specified with `-l`.
###### -B
Only print the reads with both 5′ and 3′ adapter matches.
###### -a
Print all reads with and without adapter matches if the reads are in
the range specified with `-m` and `-x`.

### `adapt-qc`: exhaustive 3′ adapter search and quality control
`adapt-qc` searches 3′ adapter sequences exhaustively and conducts
quality control for an input FASTQ. If a 3′adapter sequence is
specified with `-3`, the program only executes quality control using a
given genome mapping command.
#### Usage

    $ adapt-qc [options] <mapping_cmd> <fastq>

`<mapping_cmd>` is the genome mapping command to be tested.
For this argument, any read mapping software package can be used. 
The requirements for this argument are:
* Specify FASTA as the input read format
* Specify the input read filename as `@in`
* Specify SAM as the output format for the mapping results
* Specify the output SAM filename as `@out`
* Pass `<mapping_cmd>` as a string in the command for `adapt-qc`

For example, when you want to use
[Bowtie](http://bowtie-bio.sourceforge.net) as a mapping engine, the
entire command line for `adapt-qc` will be:

    $ adapt-qc "/path_to/bowtie /path_to/genome_index -p8 -v0 -k1 -S -f @in > @out" <fastq>

[Bowtie](http://bowtie-bio.sourceforge.net) options used:
* `-p <int>`: Number of `<int>` CPUs
* `-v <int>`: Number of `<int>` mismatches
* `-k <int>`: report up to `<int>` valid alignments
* `-S`: SAM output
* `-f`: FASTA input

#### Options
##### General Options
###### -l BP
Adapter match length in bp. The default is 7nt. `adapt-qc` only
considers perfect adapter matches. In other words, `adapt-qc` does not
use the `-s` option in `adapt-clip`.
###### -m BP
Minimum read length in bp. The default is 16nt. For more detail, see
the `-m` option in `adapt-clip`.
###### -x BP
Maximum read length in bp. The default is 36nt. For more detail, see
the `-x` option in `adapt-clip`.
##### Option for Evaluation
###### -3 SEQ1,SEQ2,...
Comma-separated list of 3′ adapter(s) for quality control.  When the
option is specified, `adapt-qc` maps the processed reads after
clipping each 3′ adapter in every run and checks the genome mapping
rate.
##### Option for Exhaustive Search
###### -p FLOAT
Subsampling fraction of reads in an input FASTQ.  In the default,
`adapt-qc` uses all reads, i.e., `-p 1.0`.  Small read sets can make
`adapt-qc` faster.
###### -k BEG:END:INT
k-mers to predict a 3′ adapter in the input FASTQ. `BEG` is the smallest
k-mer to start, `END` is the largest k-mer to end, and `INT` is an
interval of the k-mers. The default is `9:11:2`, i.e., from 9mer to
11mer in a 2nt interval (k = 9, 11).
###### -r BEG:END:INT
Cutoff ratios for filtering less abundant k-mers. As in
option `-k`, `BEG` is the smallest ratio to start, `END`
is the largest ratio to end, and `INT` is an interval of the
ratios. The default is `1.2:1.4:0.1`, i.e., from 1.2 to 1.4 in a 0.1
interval (r = 1.2, 1.3, 1.4).
###### --temp PATH
Path for the temporary directory. `adapt-qc` creates a temporary directory
during a computation. In the default setting, the program makes the
directory in the current directory.

### `qual-offset`: quality offset estimation
`qual-offset` estimates ASCII-encoded quality score offsets.
#### Usage

    $ qual-offset <fastq>

### `qual-trim`: quality trimming
`qual-trim` trims low quality bases in input reads with the same
quality trimming algorith as the one used by BWA. Since small RNA
libraries are typically single-ended, only single-ended reads are
assumed.

#### Usage

    $ qual-trim [options] <fastq>

###### -b BASE
ASCII-encoded quality score offset, e.g. 33 or 64.  The default is 33
(Phred+33).
###### -p PROB
Error probability cutoff. The default is 0.1.
###### -q SCORE
Quality score cutoff. By default, the cutoff is automatically calculated
using the quality score equation for the inferred platform. These equations 
use the error probability cutoff specified using `-p`, i.e.,
`error_probability=0.1` is equivalent to `quality_score=10` in Phred
score.
###### -l BP
Minimum read length in bp for output trimmed reads. The default is 16nt.
###### --illumina5
In ASCII-encoded quality scores generated by Illumina 1.5+, there are
sometimes `B` letters in a quality score line in a FASTQ file.  Those
`B` letters indicate the portion of a read that should not be used in
further analysis due to the low quality erroneous base callings.  This
option converts all `B` letters to the lowest ASCII-encoded quality
scores for filtering. It can be skipped to use this option when the
quality cutoff `-p` and `-q` are equal or larger than 0.1 and 10.
###### --solexa
Input FASTQ is in Solexa encoding.

### `read-collapse`: read collapsing
`read-collapse` merges identical reads while retaining the counts. The
only allowed input is the output from `adapt-clip`. During 
computation, the program keeps extracted sequences in memory. For
example, memory usage and runtime of 100 million small RNA reads will
be around ~2GB and ~5 mins, respectively.

To restrict memory usage, you can try:

    $ cut -f1 ${FILE} | sort | uniq -c | awk '{print ">"$2"_"$1"\n"$2}'

This would takes ~15 mins with ~10MB memory to process 100 million reads.
#### Usage

    $ read-collapse [options] <adapt-clip-output>

###### -f
Output in FASTA format. This is the default format for the output.  It
will be composed of 2 lines for a record:

    > [extracted_read] _ [read_count]
      [extracted_read]

###### -t
Output in TAB format. It will be composed of 1 line for a record:

    [extracted_insert] /tab/ [read_count]

###### -q
Output in FASTQ format. It will be composed of 4 lines for a record:

    @ [extracted_read] _ [read_count] length=[read_length]
      [extracted_read]
    + [extracted_read] _ [read_count] length=[read_length]
      '~' * [read_length]

Note that `~` is in the ASCII 126, which is the highest printable
character in the ASCII code.


## Examples
Here is a shell script with APPSR implemented as an automated small RNA
pipeline.

```shell
# !/bin/sh

# If you don't know the quality offset
QBASE=`qual-offset ${FASTQ} | cut -f2 -d'='`

# If you don't know the 3′ adapter sequence
ADAPT=`adapt-pred ${FASTQ} | cut -f2 -d'='`

qual-trim -b ${QBASE} ${FASTQ} | adapt-clip -3 ${ADAPT} - | \
read-collapse - > clean_reads.fa
```
Note
* `${FASTQ}` is an input FASTQ file
* `${QBASE}` is an estimated quality offset
* `${ADAPT}` is a predicted 3′ adapter sequence

If you want to do exhaustive adapter search and quality control, try:

```shell
# !/bin/sh

# If you don't know the quality offset
QBASE=`qual-offset ${FASTQ} | cut -f2 -d'='`

# Bowtie command as a mapping engine
MAPCMD="/path_to/bowtie /path_to/genome_index -p${CPU} -v0 -k1 -S -f @in > @out"

# Subsample 1 million reads from an input FASTQ
SUBSAMPLE_RATE=1.0
SUBSAMPLE_READS=1000000
TOTAL_READS=`wc -l ${FASTQ} | awk '{print $1/4}'`
if [ ${TOTAL_READS} -gt ${SUBSAMPLE_READS} ]; then
    SUBSAMPLE_RATE=`echo "${SUBSAMPLE} ${TOTAL_READS}" | awk '{print $1/$2}'`
fi

# Here is the exhaustive adapter prediction part
ADAPT=`adapt-qc -p "${SUBSAMPLE_RATE}" "${MAPCMD}" ${FASTQ} | head -n 1 | cut -f2 -d'='`

if [ "${ADAPT}" != "NULL" -a "${ADAPT}" != "RAW_INPUT" ]; then
    qual-trim -b ${QBASE} ${FASTQ} | adapt-clip -3 ${ADAPT} - | \
    read-collapse - > clean_reads.fa
fi
````

Note
* The shell script subsamples reads up to 100k reads. Delete the part
if you want to use all reads.
* `${MAPCMD}` is a genome mapping command. Although the above example
uses [Bowtie](http://bowtie-bio.sourceforge.net) to map reads, you can
try any command lines and read mapping software packages.

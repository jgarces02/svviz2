# svviz2

[![Build Status](https://travis-ci.org/nspies/svviz2.svg?branch=master)](https://travis-ci.org/nspies/svviz2)


This is a near complete rewrite of [svviz1](https://github.com/svviz/svviz). New features:

- uses [bwa mem](https://github.com/lh3/bwa) under the hood for realignments
  - substantial improvements in reliability and speed when realigning long reads
  - enables realignment against entire genome, identifying potential second-best hits
  - calculates a quantitative mapping quality score taking account of ref and alt hits genome-wide
  - uses weighted mapq scores to calculate evidence for ref and alt alleles, including genotype likelihoods
- substantially improved visualizations
  - "quick consensus" reduces background error rate in pacbio/nanopore and other long-read technologies
  - optionally uses tandem repeat finder (trf) to identify tandem repeats near candidate SV
  - visualization engine has been refactored into a separate [genomeview](https://github.com/nspies/genomeview) module, facilitating future improvements
- integrated dotplots
  - visualizes ref vs alt, allowing for visual identification of tandem repeats and other complex sequence
  - if bwa is being used for realignment, visualizes any second-best hit regions against candidate SV locus
  - if long-reads are provided as input, picks several long reads to plot as dotplots against ref and alt

Installation
------------

svviz2 requires **python 3.3** or greater. To perform tandem repeat detection, download [tandem repeats finder](http://tandem.bu.edu/trf/trf.download.html), rename the binary to "trf" and move it into your `PATH`. To visualize the dotplots, the [rpy2](https://rpy2.bitbucket.io) package must be installed. To convert visualizations to pdf format, either [inkscape](https://inkscape.org/), [rsvg-convert](https://github.com/GNOME/librsvg) or (macOS only) [webkitToPDF](https://github.com/nspies/webkitToPDF) must be installed into your `PATH`.

To install, run the following command, ideally from within a virtualenv:
```
pip install -U git+https://github.com/nspies/svviz2.git
```

A few more notable changes with respect to version 1.x
------------------------------------------------------

- variants are input in VCF format; please create an issue if you find a well-defined variant that is not supported by the current version of svviz2
- VCF files must more or less conform to the spec -- svviz2 uses pysam which uses htslib to load VCF files

Note that svviz2 does not natively support parallelization. You are probably best off parallelizing over variants (or samples). One simple way to do this is using the `--first-variant` and `--last-variant options`. If it appears that svviz2 is using more than 1 core during realignment, it may be because numpy can in some circumstances use multiple threads (see [here](https://stackoverflow.com/questions/30791550/limit-number-of-threads-in-numpy/31622299#31622299) to deactivate this behavior).

Documentation
-------------

More in-depth documentation is available at [https://svviz2.readthedocs.io](https://svviz2.readthedocs.io).

Usage
-----

```
ssw library not found
usage: svviz2 [options] --ref REF --variants VARIANTS BAM [BAM2 ...]

svviz2 version 2.0a3

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  bam                   sorted, indexed bam file containing reads of interest to plot; can be specified multiple
                        times to load multiple samples
  --ref REF, -r REF     reference fasta file (a .faidx index file will be created if it doesn't exist so you need
                        write permissions for this directory)
  --variants VARIANTS, -V VARIANTS
                        the variants to analyze, in vcf or bcf format (vcf files may be compressed with gzip)

Optional arguments:
  --outdir OUTDIR, -o OUTDIR
                        output directory for visualizations, summaries, etc (default: current working directory)
  --format FORMAT       format for output visualizations; must be one of pdf, png or svg (default: pdf,
                        or svg if no suitable converter is found)
  --savereads           output the read realignments against the appropriate alt or ref allele (default: false)
  --min-mapq MIN_MAPQ   only reads with mapq>=MIN_MAPQ will be analyzed; when analyzing paired-end data,
                        at least one read end must be near the breakpoints with this mapq (default:0)
  --align-distance ALIGN_DISTANCE
                        sequence upstream and downstream of breakpoints to include when performing re-alignment
                        (default: infer from data)
  --batch-size BATCH_SIZE
                        Number of reads to analyze at once; larger batch-size values may run more quickly
                        but will require more memory (default=10000)
  --downsample DOWNSAMPLE
                        Ensure the total number of reads per event per sample does not exceed this number
                        by downsampling (default: infinity)
  --aligner ALIGNER     The aligner to use for realigning reads; either ssw (smith-waterman) or
                        bwa (default=bwa)
  --only-realign-locally
                        Only when using bwa as the aligner backend, when this option is enabled,
                        reads will only be aligned locally around the breakpoints and not also against
                        the full reference genome (default: False)
  --fast                More aggressively skip reads that are unlikely to overlap
                        the breakpoints (default: false)
  --first-variant FIRST_VARIANT
                        Skip all variants before this variant; counting starts with first variant
                        in input VCF as 0 (default: 0)
  --last-variant LAST_VARIANT
                        Skip all variants after this variant; counting starts with first variant
                        in input VCF as 0 (default: end of vcf)
  --render-only
  --no-render
  --dotplots-only
  --no-dotplots
  --report-only
  --no-report
  --only-plot-context ONLY_PLOT_CONTEXT
                        Only show this many nucleotides before the first breakpoint, and the last breakpoint
                        in each region (default: show as much context as needed to show all reads fully)
  --also-plot-context ALSO_PLOT_CONTEXT
                        Generates two plots per event, one using the default settings, and one generated
                        by zooming in on the breakpoints as per the --only-plot-context option
```

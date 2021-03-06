# 1000 Genomes integration methods

This repository packages methods that are used in the [1000 Genomes Project](http://www.1000genomes.org/) phase3 integration pipeline, specifically for small non-SNP (and non-biallelic) sequence variants, such as indels and multiallelic or complex loci.  They are packaged here to ease the distribution of computational effort for this task.

Please contact me (<erik.garrison@bc.edu>) with any questions.

## Getting started

First, use git to clone the repository:

    git clone --recursive https://github.com/ekg/1000G-integration.git

Note the use of `--recursive`, this ensures that the many submodules included in the project are correctly cloned as well.

Build using make.  Executables (`freebayes` and `glia`) will be in `bin/`.  You should modify scripts/run\_region.sh to reflect their location, e.g.:

    ## Change this to your 1000G-integration directory, or unset if
    ## you put freebayes and glia into your system-wide path:
    bin=/path/to/1000G-integration/bin

You will also need samtools.  In my tests, I used version 0.1.19-44428cd.

## Execution

Now, you can run the graph-based regenotyping pipeline using:

    scripts/run_region.sh [outdir] \
                          [reference] \
                          [region] \
                          [union] \
                          [contamination] \
                          [cnvmap] \
                          [scratch] \
                          [merge_script]

This will generate `outdir/region.vcf.gz` in gzip format, along with a number of `*.err` files for each component in the process.  Additionally, a `*.sites.vcf.gz` file is made, and if decompression of the generated gzipped VCF does not fail, `outdir/region.ok` is touched.

The `cnvmap` and `contamination` estimate files for the 2535 samples in the
1000G release are both in `resources/`.  The 1000 Genomes reference should be
used here.  You will also need to download the whole-genome [union allele list](http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/working/20130723_phase3_wg/union/ALL.wgs.union_from_bc.20130502.snps_indels_complex.sites.vcf.gz) and its .tbi index from the 1000 Genomes FTP site.  Supply this as `union` in the `run_region.sh` command.

You will also need to provide a `merge_script` which takes a genomic region in samtools format (e.g. 5:300-400).  This script should produce a merged, uncompressed BAM stream on stdout of all low-coverage and exome sequencing data mapped to the target region in the 2535 samples in the 1000G phase3.  The exact functioning of this script is likely system-dependent, but an example that performs the merge in less than 700Mb of memory is provided in the scripts directory.  This method requires a merged SAM header which has been generated with bamtools, and is provided in the resources directory.

## Considerations

The method does assume that you have stored your data in per-sample (exome and low-coverage) BAMs, and that these need to be merged, processed by glia, written to disk (temporarily) and then processed by freebayes for best performance.  However, you may have already generated per-region files for the data.

You will need suitable scratch space (specified as `scratch`) on the nodes where you execute the script.  If this is not available, the glia and freebayes components can run on purely streamed data at a cost of doubled runtime memory requirements.  If this is the case, you will need to modify the `run_region.sh` script to not write the temporary file.  Please contact me if this is necessary.

You will also need enough memory.  In this alignment data, glia and freebayes will rarely use more than 4-5Gb at runtime.  You can detect such failures by grepping for out-of-memory errors ("bad alloc"s) in the `*.err` files.

In practice, I use a set of targets that have approximately equal amounts of sequencing data in them (this was estimated from ~20 exome and ~20 low coverage files).  This kind of even partitioning can improve performance, but again this is possibly system-dependent and thus the merging strategy is left up to the user.

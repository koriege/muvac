# Muvac

Muvac implements multiple variant calling options from Exome-Seq/WG-Seq or RNA-Seq data. It offers free GATK best-practices in an optimized, parallelized fashion.

Muvac leverages on bashbone, which is a bash/biobash library for workflow and pipeline design within but not restricted to the scope of Next Generation Sequencing (NGS) data analyses. muvac makes use of bashbones best-practice parameterized and run-time tweaked software wrappers and compiles them into a multi-threaded pipeline for analyses of model AND non-model organisms.

## Features

- Most software related parameters will be inferred directly from your data so that all functions require just a minimalistic set of input arguments
- Benefit from a non-root stand-alone installer without need for any prerequisites
- Get genomes, annotations from Ensembl, variants from GATK resource bundle and RAW sequencing data from NCBI Sequence Read Archive (SRA)
- Extensive logging and different verbosity levels
- Start, redo or resume from any point within your data analysis pipeline

## Covered Tasks

- For paired-end and single-end derived raw sequencing or prior mapped read data
- Data preprocessing (quality check, adapter clipping, quality trimming, error correction, artificial rRNA depletion)
- Read alignment and post-processing
  - knapsack problem based slicing of alignment files for parallel task execution
  - sorting, filtering, unique alignment extraction, removal of optical duplicates
  - read group modification, split N-cigar reads, left-alignment and base quality score recalibration
- Variant detection from DNA or RNA sequencing experiments
  - Integration of multiple solutions for germline and somatic calling
  - VCF normalization

# License

The whole project is licensed under the GPL v3 (see LICENSE file for details). <br>
**except** the the third-party tools set-upped during installation. Please refer to the corresponding licenses

Copyleft (C) 2020, Konstantin Riege

# Download

This will download you a copy which includes the latest developments

```bash
git clone --recursive https://github.com/Hoffmann-Lab/muvac.git
```

To check out the latest release (irregularly compiled) do

```bash
cd muvac
git checkout --recurse-submodules $(git describe --tags)
```


# Installation

## Full installation from scratch

```bash
scripts/setup.sh -i all -d <path/to/installation>
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -h
```

## Upgrade to a new release or if bashbone was previously installed

```bash
scripts/setup.sh -i upgrade -d <path/of/installation>
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -h
```

## Update tools

The setup routine will always install the latest software via conda, which can be updated by running the related setup functions again.

```bash
scripts/setup.sh -i conda_tools -d <path/of/installation>
```

Trimmomatic, segemehl, mdless and gztool will be installed next to the conda environments. If new releases are available, they will be automatically fetched and installed upon running the related setup functions again.

```bash
scripts/setup.sh -i trimmomatic,segemehl,mdless,gztool -d <path/of/installation>
```

# Usage

To load muvac, bashbone respectively, execute
```bash
source <path/of/installation/latest/muvac/activate.sh>
bashbone -h
```

Now enable bashbone to use its conda environments in order to make scripts and functions to operate properly. Conda as well as bashbone can be disabled afterwards.
```bash
# enable/disable conda
bashbone -c
# disable bashbone
bashbone -x
```

Shortcut:

```bash
source <path/of/installation/latest/muvac/activate.sh> -c true
```

## Retrieve SRA datasets

Use the enclosed script to fetch sequencing data from SRA

```bash
source <path/of/installation/latest/muvac/activate.sh> -c true
sra-dump.sh -h
```

## Genome setup

Human genome chromosomes must follow GATK order and naming schema: chrM,chr1..chr22,chrX,chrY. This requirement needs to be fulfilled in all associated VCF files, too. To obtain properly configured genomes and dbSNP files, see blow.

## Retrieve genomes

Use the enclosed script to fetch human hg19/hg38 or mouse mm9/mm10/mm11 genomes and annotations. Plug-n-play CTAT genome resource made for gene fusion detection and shipped with STAR index can be selected optionally.

```bash
source <path/of/installation/latest/muvac/activate.sh> -c true
dlgenome.sh -h
```

## Index genomes

```bash
source <path/of/installation/latest/muvac/activate.sh> -c true
muvac.sh -x -g <path/to/genome.fa> -gtf <path/to/genome.gtf>
```

## Retrieve GATK resources

To obtain panel of normals, common somatic variants and population variants with allele frequencies visit

- HG38: <https://console.cloud.google.com/storage/browser/gatk-best-practices/somatic-hg38/>
- HG19: <https://console.cloud.google.com/storage/browser/gatk-best-practices/somatic-b37/>

Afterwards, place the files next to your genome fasta file with equal name plus extension suffix as shown below.

- genome.fa.small_common.vcf.gz.tbi
- genome.fa.pon.vcf.tbi
- genome.fa.af_only_gnomad.vcf.gz.tbi

Analogously, place a custom dbSNP file next to the genome as genome.fa.vcf. This is automatically done by `dlgenome.sh -s` (see above). To manually download the data, possible resources are

- HG38: ftp://ftp.ensembl.org/pub/current_variation/vcf/homo_sapiens/
- HG19: ftp://ftp.ensembl.org/pub/grch37/current/variation/vcf/homo_sapiens/

## Examples

This section showcases some usages without explaining each parameter in a broader detail. Check out the muvac help page for more configuration options. Most of them will be opt-out settings.

### Data pre-processing and mapping

Data pre-processing without mapping by segemehl or STAR and without SortMeRNA for artificial rRNA depletion.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq> [-2 <fastq>] -no-rrm -no-sege -no-star
```

Data pre-processing, mapping by segemehl and STAR and alignment post-processing (i.e. unique read extraction, sorting, indexing, removal of duplicons, clipping of overlapping mate pairs).

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq> [-2 <fastq>]
```

Data pre-processing with Illumina universal adapter removal, mapping by segemehl and STAR and alignment post-processing (i.e. unique read extraction, sorting, indexing). Sequences can be found in the Illumina Adapter Sequences Document (<https://www.illumina.com/search.html?q=Illumina Adapter Sequences Document>) or Illumina Adapter Sequences HTML (<https://support-docs.illumina.com/SHARE/adapter-sequences.htm>) and the resource of Trimmomatic (<https://github.com/usadellab/Trimmomatic/tree/main/adapters>), FastQC respectively (<https://github.com/s-andrews/FastQC/blob/master/Configuration>).

The following excerpt is independent of the indexing type, i.e. single, unique dual (UD) or combinatorial dual (CD).

Nextera (Transposase Sequence), TruSight, AmpliSeq, stranded total/mRNA Prep, Ribo-Zero Plus: CTGTCTCTTATACACATCT

TruSeq (Universal) Adapter with A prefix due to 3' primer A-tailing : AGATCGGAAGAGC

TruSeq full length DNA & RNA R1: AGATCGGAAGAGCACACGTCTGAACTCCAGTCA R2: AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT

TruSeq full length DNA MethC R1: AGATCGGAAGAGCACACGTCTGAAC R2: AGATCGGAAGAGCGTCGTGTAGGGA

TruSeq Small RNA 3': TGGAATTCTCGGGTGCCAAGG
TruSeq Small RNA 5': GTTCAGAGTTCTACAGTCCGACGATC

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq> [-2 <fastq>] -a1 AGATCGGAAGAGC [-a2 AGATCGGAAGAGC]
```

Data pre-processing, mapping by segemehl and STAR and disabled post-processing (i.e. unique read extraction, sorting, indexing, removal of duplicons, clipping of overlapping mate pairs).

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq> [-2 <fastq>] -no-uniq -no-sort -no-idx -no-rmd -no-cmo
```

Multiple inputs can be submitted as comma separated list.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq,fastq,...> [-2 <fastq,fastq,...>]
```

Tweak the amount of allowed mismatches in % during mapping

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq> [-2 <fastq>] -d 10
```

### Variant calling

Perform pre-processing, mapping and post-processing with subsequent germline variant calling.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq,fastq,...> [-2 <fastq,fastq,...>] -no-pon
```

Perform generation of a custom panel of normals for subsequent somatic variant calling.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq,fastq,...> [-2 <fastq,fastq,...>]

```

Perform pre-processing, mapping and post-processing with subsequent somatic variant calling with known sites and a panel of normals e.g. provided by GATK resources.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -p <pon> -s <dbsnp> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-n1 <fastq,fastq,...> [-n2 <fastq,fastq,...>] -t1 <fastq,fastq,...> [-t2 <fastq,fastq,...>]
```

### Variant calling from RNA

To enable RNA split read mapping and required post-processing functions, use the `-split` option to call e.g. germline variants.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> \
-1 <fastq,fastq,...> [-2 <fastq,fastq,...>] -split -no-pon
```

### Start, redo or resume

List all possible break points and keywords to control Muvac.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh -dev
```

Use comma separated lists to e.g. skip md5 check and quality analysis.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh [...] -skip md5,fqual
```

Example how to resume from the segemehl mapping break point after previous data pre-processing.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh [...] -resume sege
```

Single tasks can be re-computed with the `redo` parameter and a comma separated list of arguments.

```bash
source <path/of/installation/latest/muvac/activate.sh>
muvac.sh [...] -redo bqsr,idx,hc
```

# Third-party software

| Tool | Source | DOI |
| ---  | ---    | --- |
| Arriba        | <https://github.com/suhrig/arriba/>                                 | NA |
| BamUtil       | <https://genome.sph.umich.edu/wiki/BamUtil>                         | 10.1101/gr.176552.114 |
| BBTools       | <https://jgi.doe.gov/data-and-tools/software-tools/bbtools>         | 10.1371/journal.pone.0185056 |
| BWA           | <https://github.com/lh3/bwa>                                        | 10.1093/bioinformatics/btp324 |
| BWA-mem2      | <https://github.com/bwa-mem2/bwa-mem2>                              | 10.1109/IPDPS.2019.00041 |
| BWA-meth      | <https://github.com/brentp/bwa-meth>                                | arXiv:1401.1129 |
| BCFtools      | <http://www.htslib.org/doc/bcftools.html>                           | 10.1093/bioinformatics/btr509 |
| BEDTools      | <https://bedtools.readthedocs.io>                                   | 10.1093/bioinformatics/btq033 |
| gztool        | <https://github.com/circulosmeos/gztool>                            | NA |
| cgpBigWig     | <https://github.com/cancerit/cgpBigWig>                             | NA |
| clusterProfiler | <https://guangchuangyu.github.io/software/clusterProfiler>        | 10.1089/omi.2011.0118 |
| Cutadapt      | <https://cutadapt.readthedocs.io/en/stable>                         | 10.14806/ej.17.1.200 |
| DANPOS3       | <https://github.com/sklasfeld/DANPOS3>                              | 10.1101/gr.142067.112 |
| deepTools2    | <https://deeptools.readthedocs.io/en/latest/index.html>             | 10.1093/nar/gkw257 |
| DESeq2        | <https://bioconductor.org/packages/release/bioc/html/DESeq2.html>   | 10.1186/s13059-014-0550-8 <br> 10.1093/biostatistics/kxw041|
| DEXSeq        | <https://bioconductor.org/packages/release/bioc/html/DEXSeq.html>   | 10.1101/gr.133744.111 |
| DIEGO         | <http://www.bioinf.uni-leipzig.de/Software/DIEGO>                   | 10.1093/bioinformatics/btx690 |
| DGCA          | <https://github.com/andymckenzie/DGCA>                              | 10.1186/s12918-016-0349-1 |
| dupsifter     | <https://github.com/huishenlab/dupsifter>                           | 10.1093/bioinformatics/btad729 |
| fastqc        | <https://www.bioinformatics.babraham.ac.uk/projects/fastqc>         | NA |
| featureCounts | <http://subread.sourceforge.net>                                    | 10.1093/bioinformatics/btt656 |
| fgbio         | <http://fulcrumgenomics.github.io/fgbio/>                           | NA |
| freebayes     | <https://github.com/ekg/freebayes>                                  | arXiv:1207.3907 |
| GATK4         | <https://github.com/broadinstitute/gatk>                            | 10.1101/gr.107524.110 <br> 10.1038/ng.806 |
| GEM           | <https://groups.csail.mit.edu/cgs/gem>                              | 10.1371/journal.pcbi.1002638 |
| GNU Parallel  | <https://www.gnu.org/software/parallel/>                            | 10.5281/zenodo.1146014 |
| GoPeaks       | <https://github.com/maxsonBraunLab/gopeaks>                         | 10.1186/s13059-022-02707-w |
| GoSemSim      | <http://bioconductor.org/packages/release/bioc/html/GOSemSim.html>  | 10.1093/bioinformatics/btq064 |
| GSEABase      | <https://bioconductor.org/packages/release/bioc/html/GSEABase.html> | NA |
| HTSeq         | <https://htseq.readthedocs.io>                                      | 10.1093/bioinformatics/btu638 |
| IDR           | <https://github.com/nboley/idr>                                     | 10.1214/11-AOAS466 |
| IGV           | <http://software.broadinstitute.org/software/igv>                   | 10.1038/nbt.1754 |
| Intervene     | <https://github.com/asntech/intervene>                              | 10.1186/s12859-017-1708-7 |
| kent/UCSC utilities | <https://hgdownload.soe.ucsc.edu/downloads.html#utilities_downloads> | 10.1093/bioinformatics/btq351 |
| khmer         | <https://khmer.readthedocs.io>                                      | 10.12688/f1000research.6924.1 |
| m6aViewer     | <http://dna2.leeds.ac.uk/m6a/>                                      | 10.1261/rna.058206.116 |
| Macs2         | <https://github.com/macs3-project/MACS>                             | 10.1186/gb-2008-9-9-r137 |
| MethylDackel  | <https://github.com/dpryan79/MethylDackel>                          | NA |
| metilene      | <https://www.bioinf.uni-leipzig.de/Software/metilene/>              | 10.1101/gr.196394.115 |
| moose2        | <http://grabherr.github.io/moose2/>                                 | 10.1186/s13040-017-0150-8 |
| PEAKachu      | <https://github.com/tbischler/PEAKachu>                             | NA |
| Picard        | <http://broadinstitute.github.io/picard>                            | NA |
| Platypus      | <https://rahmanteamdevelopment.github.io/Platypus>                  | 10.1038/ng.3036 |
| pugz          | <https://github.com/Piezoid/pugz>                                   | 10.1109/IPDPSW.2019.00042 |
| rapidgzip     | <https://github.com/mxmlnkn/rapidgzip>                              | 10.1145/3588195.3592992 |
| RAxML         | <https://cme.h-its.org/exelixis/web/software/raxml/index.html>      | 10.1093/bioinformatics/btl446 |
| Rcorrector    | <https://github.com/mourisl/Rcorrector>                             | 10.1186/s13742-015-0089-y |
| RSeQC         | <http://rseqc.sourceforge.net>                                      | 10.1093/bioinformatics/bts356 |
| REVIGO        | <https://code.google.com/archive/p/revigo-standalone>               | 10.1371/journal.pone.0021800 |
| RRVGO         | <https://ssayols.github.io/rrvgo>                                   | 10.17912/micropub.biology.000811 |
| Salmon        | <https://combine-lab.github.io/salmon/>                             | 10.1038/nmeth.4197 |
| SalmonTE      | <https://github.com/hyunhwan-jeong/SalmonTE>                        | 10.1142/9789813235533_0016 |
| SAMtools      | <http://www.htslib.org/doc/samtools.html>                           | 10.1093/bioinformatics/btp352 |
| SEACR         | <https://github.com/FredHutch/SEACR>                                | 10.1186/s13072-019-0287-4 |
| segemehl      | <http://www.bioinf.uni-leipzig.de/Software/segemehl>                | 10.1186/gb-2014-15-2-r34 <br> 10.1371/journal.pcbi.1000502 |
| SnpEff        | <https://pcingola.github.io/SnpEff>                                 | 10.4161/fly.19695 |
| SortMeRNA     | <https://bioinfo.lifl.fr/RNA/sortmerna>                             | 10.1093/bioinformatics/bts611 |
| STAR          | <https://github.com/alexdobin/STAR>                                 | 10.1093/bioinformatics/bts635 |
| STAR-Fusion   | <https://github.com/STAR-Fusion/STAR-Fusion/wiki>                   | 10.1101/120295 |
| Trimmomatic   | <http://www.usadellab.org/cms/?page=trimmomatic>                    | 10.1093/bioinformatics/btu170 |
| UMI-tools     | <https://github.com/CGATOxford/UMI-tools>                           | 10.1101/gr.209601.116 |
| VarDict       | <https://github.com/AstraZeneca-NGS/VarDict>                        | 10.1093/nar/gkw227 |
| VarScan       | <http://dkoboldt.github.io/varscan>                                 | 10.1101/gr.129684.111 |
| vcflib        | <https://github.com/vcflib/vcflib>                                  | 10.1371/journal.pcbi.1009123 |
| Vt            | <https://genome.sph.umich.edu/wiki/Vt>                              | 10.1093/bioinformatics/btv112 |
| WGCNA         | <https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA> | 10.1186/1471-2105-9-559 |

# Supplementary information

Muvac can be executed in parallel instances and thus are able to be submitted as jobs into a queuing system like a Sun Grid Engine (SGE). This could be easily done by utilizing `commander::qsubcmd. This function makes use of array jobs, which further allows to wait for completion of all jobs, handle single exit codes and alter used resources via `commander::qalter.

```bash
source <path/of/installation/latest/muvac/activate.sh>
declare -a cmds=()
for i in *R1.fastq.gz; do
	j=${i/R1/R2}
	sh=job_$(basename $i .R1.fastq.gz)
	commander::makecmd -a cmd1 -c {COMMANDER[0]}<<- CMD
		muvac.sh -v 2 -t <threads> -g <fasta> -gtf <gtf> -o <outdir> -l <logfile> -tmp <tmpdir> -1 $i -2 $j
	CMD
  # or simply cmds+=("muvac.sh [...]")
done
commander::qsubcmd -r -p <env> -t <threads> -i <instances> -n <jobname> -o <logdir> -a cmds
commander::qstat
commander::qalter -p <jobname|jobid> -i <instances>
```

In some rare cases a glibc pthreads bug (<https://sourceware.org/bugzilla/show_bug.cgi?id=23275>) may cause pigz failures (`internal threads error`) and premature termination of tools leveraging on it e.g. Cutadapt and pigz. One can circumvent this by e.g. making use of an alternative pthreads library e.g. compiled without lock elision via `LD_PRELOAD`

```bash
source <path/of/installation/latest/bashbone/activate.sh>
LD_PRELOAD=</path/to/no-elision/libpthread.so.0> <command>
```

# Closing remarks

Bashbone is a continuously developed library and actively used in my daily work. As a single developer it may take me a while to fix errors and issues. Feature requests cannot be handled so far, but I am happy to receive pull request.

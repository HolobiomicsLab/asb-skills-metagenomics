---
name: spades-de-novo-genome-assembly
description: Use when raw paired-end genomic DNA FASTQ reads are available and a full-genome
  assembly baseline is needed — e.g. to benchmark targeted assembly (CasCollect) runtimes/contig
  counts or to recover complete CRISPR/Cas systems by exhaustive assembly.
when_to_use_negative:
- The goal is rapid CRISPR/Cas discovery from unassembled reads — CasCollect's targeted
  assembly reduces runtime by almost two orders of magnitude versus complete assembly
- Input is a metagenomic sample without this substitution — plain SPAdes should be
  replaced by metaSPAdes in the CasCollect pipeline
- Only reads already assembled into contigs are available — there is no read set to
  feed SPAdes
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SPAdes
  role: De novo assembler run on the full trimmed read set to provide the conventional
    complete-assembly baseline for runtime and contig-count comparison
  repo: https://github.com/sandialabs/CasCollect
- name: bbduk
  role: Implements the --trim option (quality trim, remove adapters, merge overlapping
    reads) producing the cleaned reads fed to SPAdes
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: CRISPRCasFinder
  role: Searches the resulting assembly contigs to complete the annotation step of
    the conventional pathway
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:96f9c4ed116a6d22508006aace1c183ebd0576c17a5dba174dd69b4f2cd25325
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - SPAdes
  - bbduk
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spades-de-novo-genome-assembly

## Summary

Runs a complete SPAdes de novo assembly on trimmed paired-end Illumina reads to serve as the conventional genome-assembly baseline against which CasCollect's targeted assembly of CRISPR/Cas loci is benchmarked (runtimes and contig counts).

## When to use

Use when raw paired-end genomic DNA FASTQ reads are available and a full-genome assembly baseline is needed — e.g. to benchmark targeted assembly (CasCollect) runtimes/contig counts or to recover complete CRISPR/Cas systems by exhaustive assembly.

## When NOT to use

- The goal is rapid CRISPR/Cas discovery from unassembled reads — CasCollect's targeted assembly reduces runtime by almost two orders of magnitude versus complete assembly
- Input is a metagenomic sample without this substitution — plain SPAdes should be replaced by metaSPAdes in the CasCollect pipeline
- Only reads already assembled into contigs are available — there is no read set to feed SPAdes

## Inputs

- Paired-end genomic DNA sequencing reads in FASTQ format (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39 isolate reads, read lengths 90/100/226 bp)
- Quality- and adapter-trimmed read set (output of bbduk-based trimming/cleaning)

## Outputs

- Complete genome assembly as SPAdes contigs/scaffolds in FASTA format
- Wall-clock runtime per assembly run (minutes)
- Contig counts after applying the 500 bp annotation size cutoff

## How to apply

Quality-trim and adapter-remove the reads first (e.g. via the bbduk-based --trim step used in CasCollect), then run a complete SPAdes assembly on the full trimmed read set for each isolate, as done for E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39. Record wall-clock runtime per assembly to allow head-to-head comparison with targeted assembly. Apply the 500 bp contig size cutoff when tallying annotatable contigs so counts are comparable to the CasCollect pipeline convention. For metagenomic inputs, substitute metaSPAdes in place of SPAdes, as the CasCollect pipeline does. Downstream annotation of the contigs (e.g. with CRISPRCasFinder) completes the conventional assembly-and-annotation pathway that CasCollect is meant to replace.

## Related tools

- **SPAdes** (De novo assembler run on the full trimmed read set to provide the conventional complete-assembly baseline for runtime and contig-count comparison) — https://github.com/sandialabs/CasCollect
- **bbduk** (Implements the --trim option (quality trim, remove adapters, merge overlapping reads) producing the cleaned reads fed to SPAdes) — https://jgi.doe.gov/data-and-tools/bbtools/
- **CRISPRCasFinder** (Searches the resulting assembly contigs to complete the annotation step of the conventional pathway)

## Examples

```
spades.py --pe1-1 KLY_R1.trimmed.fastq.gz --pe1-2 KLY_R2.trimmed.fastq.gz -o KLY_spades_out -t 1 -m 20
```

## Evaluation signals

- Assembly completes on the full trimmed read set and produces a contigs FASTA with contig counts consistent with isolate genome size (baseline ~161K contigs on average after the 500 bp cutoff in the paper's benchmarking)
- Wall-clock runtime is recorded and is comparable to reported complete-assembly timings (17.4, 33.3 and 31.5 min for E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39, respectively)
- Annotation of assembled contigs recovers all previously known cas genes in each isolate genome, matching the paper's finding that both pipelines identified all known cas genes
- Contig tally applies the same 500 bp size cutoff used for the targeted pipeline so counts are directly comparable (e.g. 146 vs 19 for E. coli KLY, 162 vs 111 for VA-134, 44 vs 89 for M39)

## Limitations

- Complete assembly of the full read set is far slower than targeted assembly — the paper reports CasCollect reduces identification/annotation time by almost two orders of magnitude versus complete assembly
- Contig counts can be larger for complete assembly than targeted assembly on the same isolate (e.g. 146 vs 19 for E. coli KLY), reflecting the cost of assembling the whole genome
- No software version numbers or accession IDs for SPAdes are given in the article, hindering exact reproduction

## Evidence

- [results] SPAdes as de novo assembler: "The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps"
- [results] metaSPAdes substitution for metagenomes: "For metagenomes, metaSPAdes is used in-place-of SPAdes."
- [results] 500 bp contig annotation cutoff: "Our Perl script for automating the annotation had set a size cutoff of 500 bp, which reduced the total number of contigs analyzed"
- [readme] Required software including SPAdes: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
- [methods] Trimming option feeding assembly: "--trim              runs quality trim, remove adapters, merge overlapping"

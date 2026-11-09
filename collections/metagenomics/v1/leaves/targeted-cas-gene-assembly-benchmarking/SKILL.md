---
name: targeted-cas-gene-assembly-benchmarking
description: Use when evaluating whether a targeted-assembly approach (e.g. CasCollect
  in protein mode with --trim --meta) outperforms full metaSPAdes assembly or PRICE
  for cas gene/operon recovery from pooled metagenomic FASTQ reads, with runtime and
  contig-count comparisons.
when_to_use_negative:
- Input is a single isolate genome rather than a pooled metagenome — the isolate benchmarks
  in the paper showed CasCollect was sometimes slower than complete assembly (e.g.
  23.3 vs 17.4 min for E. coli KLY)
- No CRISPR/Cas content is expected in the reads — targeted seed generation from Cas
  HMMs will find no seeds and the runtime advantage vanishes
- You need a whole-genome assembly or genome-wide variant calls — CasCollect only
  assembles the read subset around CRISPR/Cas loci
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: CasCollect
  role: 'Targeted assembly pipeline under test: HMMER seed generation, VSEARCH seed
    expansion, metaSPAdes assembly, CRISPRCasFinder annotation'
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: Queries the 120 Cas protein HMMs to identify seed reads matching cas genes
- name: VSEARCH
  role: Performs seed-expansion identity matching (default 95%) across five cycles
    in protein mode
  repo: https://github.com/torognes/vsearch
- name: metaSPAdes
  role: Assembler for both the CasCollect read subset (--meta) and the complete-assembly
    baseline
- name: CRISPRCasFinder
  role: Annotates assembled contigs for CRISPR arrays and cas operons
- name: bbduk
  role: Implements the --trim quality trim, adapter removal and read merging step
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: Strips and prepares pooled isolate reads to build the simulated metagenome
  repo: https://github.com/lh3/seqtk
- name: PRICE
  role: Targeted-assembly baseline assembler against which CasCollect is compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:d0f5a8978b267edabbbc6db852955c43df813f32db941d32bbdbd7ef078c70da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - Seqtk
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# targeted-cas-gene-assembly-benchmarking

## Summary

Benchmark a targeted CRISPR/Cas assembly pipeline (CasCollect) against complete metagenome assembly (metaSPAdes) and the PRICE assembler on a simulated metagenome, comparing wall-clock runtime, contig output counts, and recovery of all known cas genes and correctly assembled cas operons.

## When to use

Use when evaluating whether a targeted-assembly approach (e.g. CasCollect in protein mode with --trim --meta) outperforms full metaSPAdes assembly or PRICE for cas gene/operon recovery from pooled metagenomic FASTQ reads, with runtime and contig-count comparisons.

## When NOT to use

- Input is a single isolate genome rather than a pooled metagenome — the isolate benchmarks in the paper showed CasCollect was sometimes slower than complete assembly (e.g. 23.3 vs 17.4 min for E. coli KLY)
- No CRISPR/Cas content is expected in the reads — targeted seed generation from Cas HMMs will find no seeds and the runtime advantage vanishes
- You need a whole-genome assembly or genome-wide variant calls — CasCollect only assembles the read subset around CRISPR/Cas loci

## Inputs

- Paired-end bacterial isolate FASTQ reads: E. coli KLY (SRR1424625), P. aeruginosa VA-134 (SRR2939129), S. pyogenes M39 (SRR5280756)
- Mouse paired-end sequencing reads for spiking (SRR1752459)
- Default set of 120 Cas protein family profile HMMs shipped with CasCollect
- 105 CRISPR repeat sequences (optional DNA-mode seeds) included with CasCollect

## Outputs

- cascollect_metagenome_benchmark.csv tabulating runtimes and contig counts for CasCollect, complete assembly, and PRICE
- Assembled contigs from each assembler (500 bp annotation cutoff) annotated by HMMER and CRISPRCasFinder
- Verified cas gene/operon annotations confirming recovery of all known cas genes from the three source isolates

## How to apply

Build a simulated metagenome by pooling paired-end isolate read sets (E. coli KLY SRR1424625, P. aeruginosa VA-134 SRR2939129, S. pyogenes M39 SRR5280756) with mouse reads (SRR1752459) spiked in, preparing reads with Seqtk. Run CasCollect in protein mode with --trim and --meta, which uses 120 default Cas protein HMMs queried by HMMER for seed generation, five VSEARCH seed-expansion cycles at the default 95% identity (-match 0.95, -cycle 5), metaSPAdes assembly of the read subset, and CRISPRCasFinder annotation; record wall-clock runtime and contig count. Run a complete metaSPAdes assembly of the same metagenome as the full-assembly baseline and the PRICE assembler as a second baseline, recording the same metrics. Annotate contigs from all three runs with a 500 bp size cutoff using HMMER/CRISPRCasFinder and verify that all previously known cas genes from the three source isolates are recovered and cas operons are correctly assembled. Expect approximately 1.6 h for CasCollect versus 2.5 h for complete assembly (~36% runtime reduction) and contig counts of roughly 723 (CasCollect), 10467 (complete), and 56 (PRICE); write the comparison to cascollect_metagenome_benchmark.csv.

## Related tools

- **CasCollect** (Targeted assembly pipeline under test: HMMER seed generation, VSEARCH seed expansion, metaSPAdes assembly, CRISPRCasFinder annotation) — https://github.com/sandialabs/CasCollect
- **HMMER** (Queries the 120 Cas protein HMMs to identify seed reads matching cas genes)
- **VSEARCH** (Performs seed-expansion identity matching (default 95%) across five cycles in protein mode) — https://github.com/torognes/vsearch
- **metaSPAdes** (Assembler for both the CasCollect read subset (--meta) and the complete-assembly baseline)
- **CRISPRCasFinder** (Annotates assembled contigs for CRISPR arrays and cas operons)
- **bbduk** (Implements the --trim quality trim, adapter removal and read merging step) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (Strips and prepares pooled isolate reads to build the simulated metagenome) — https://github.com/lh3/seqtk
- **PRICE** (Targeted-assembly baseline assembler against which CasCollect is compared)

## Examples

```
CasCollect.py --trim --meta -o cascollect_meta pooled_metagenome_R1.fq.gz pooled_metagenome_R2.fq.gz
```

## Evaluation signals

- CasCollect runtime approximates 1.6 h on the simulated metagenome, versus ~2.5 h for the complete assembly (≈36% reduction)
- Contig counts fall near the reported values: 723 (CasCollect), 10467 (complete assembly), 56 (PRICE)
- HMMER/CRISPRCasFinder annotation at the 500 bp cutoff recovers all previously known cas genes from E. coli KLY, P. aeruginosa VA-134, and S. pyogenes M39
- Cas operons are correctly assembled and annotated from the metagenome, matching the known isolate operon structures
- CasCollect used protein mode with default settings: 120 Cas protein HMMs, five VSEARCH expansion cycles at 95% identity

## Limitations

- The ~36% runtime reduction on this metagenome is far smaller than the almost two-orders-of-magnitude speedups reported elsewhere; the advantage depends on dataset composition and mode
- No software version numbers or SRA accession details are consistently given for tools and datasets, complicating exact reproduction
- On individual isolates, CasCollect was not always faster than complete assembly (e.g. 23.3 vs 17.4 min for E. coli KLY; 21.6 vs 31.5 min for S. pyogenes M39, where complete assembly was slower)
- CasCollect requires all third-party dependencies (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) installed and in the user's path

## Evidence

- [other] CasCollect reduced the runtime modestly by 36% relative to the complete assembly, with a CasCollect runtime of 1.6 h versus 2.5 h for the complete assembly pipeline.: [evidence span withheld — anchor status: not_found]
- [results] The pipeline relies on several third-party programs that include: HMMER, VSEARCH, SPAdes and CRISPRCasFinder.: "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [results] For metagenomes, metaSPAdes is used in-place-of SPAdes.: "For metagenomes, metaSPAdes is used in-place-of SPAdes."
- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] Our Perl script for automating the annotation had set a size cutoff of 500 bp: "Our Perl script for automating the annotation had set a size cutoff of 500 bp"
- [readme] Before running, CasCollect will check for the necessary software in the users path and denote any missing software.: "Before running, **CasCollect** will check for the necessary software in the users path and denote any missing software."

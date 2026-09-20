---
name: profile-hmm-protein-search-for-seed-generation
description: Use when raw unassembled Illumina FASTQ reads must be screened for cas
  genes (or any specialty protein family) so matched reads can seed targeted assembly
  instead of whole-genome assembly.
when_to_use_negative:
- Input is already assembled contigs or a genome rather than raw short reads — the
  HMM seed search on reads is designed to bypass complete assembly
- The target gene family has no profile HMM or reference sequence available and none
  can be constructed — seed generation relies on HMM profiles or CRISPR repeat references
- Long-read or non-Illumina data, since CasCollect targets short DNA sequencing reads
  from Illumina sequencing
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: CasCollect
  role: Python-wrapped pipeline orchestrating seed generation, read-subset expansion,
    assembly and annotation; run in default protein mode with --meta for metagenomes
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: Searches translated reads against the default 120 Cas protein family profile
    HMMs to identify seed reads
- name: FragGeneScan
  role: Open reading frame prediction on short reads prior to HMMER profile search
  repo: https://github.com/hallamlab/FragGeneScanPlus
- name: VSEARCH
  role: 'Read-subset expansion: identity matching of reads to seed set over five cycles
    at 95% identity'
  repo: https://github.com/torognes/vsearch
- name: SPAdes
  role: Assembles the seed-plus-expanded read subset; metaSPAdes is used in place
    of SPAdes for metagenomes
- name: bbduk
  role: Optional trimming/cleaning of reads (quality trim, adapter removal, contaminant
    removal) before seed generation
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:3d88638cab31c264752acde19268fdd932cf7ade8aeff1fcb2c244aea321777a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - BBTools
  - CRISPRCasFinder
  - CasCollect
  - FragGeneScan
  - HMMER
  - PRICE
  - SPAdes
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# Profile HMM protein search for seed generation

## Summary

Identifies short DNA sequencing reads matching cas protein HMM profiles with HMMER to generate 'seed' reads that prime CasCollect's targeted assembly of CRISPR/Cas loci, avoiding complete genome/metagenome assembly.

## When to use

Use when raw unassembled Illumina FASTQ reads must be screened for cas genes (or any specialty protein family) so matched reads can seed targeted assembly instead of whole-genome assembly.

## When NOT to use

- Input is already assembled contigs or a genome rather than raw short reads — the HMM seed search on reads is designed to bypass complete assembly
- The target gene family has no profile HMM or reference sequence available and none can be constructed — seed generation relies on HMM profiles or CRISPR repeat references
- Long-read or non-Illumina data, since CasCollect targets short DNA sequencing reads from Illumina sequencing

## Inputs

- Raw unassembled paired-end Illumina FASTQ sequencing reads (isolate genomes or metagenomes, e.g. SRR10598175, ERR472738, SRR11195315, SRR5823773)
- Profile HMM query set (default: 120 Cas protein family profile HMMs, or user-provided HMMs)

## Outputs

- Seed reads matching cas protein HMM profiles, used to prime targeted assembly
- Downstream: assembled contigs of cas operons and CRISPR arrays with cas gene and CRISPR array annotations

## How to apply

Run FragGeneScan to predict ORFs on the raw paired-end reads, then search the translated reads with HMMER against CasCollect's default set of 120 Cas protein family profile HMMs; reads matching these HMMs become 'seed' reads. Feed the seed reads into the subsequent read-subset expansion step, where VSEARCH performs five cycles (default, adjustable with -cycle) of identity matching at 95% (default, adjustable with -match) to pull in flanking reads, removing already-matched reads between cycles to eliminate redundant searches. The expanded subset is assembled with SPAdes (metaSPAdes for metagenomes) and the resulting contigs are annotated with the default 120 HMMs and CRISPRCasFinder. This targeted approach reduced cas gene identification time from unassembled metagenomic data by almost two orders of magnitude versus complete assembly plus annotation (1.7 ± 0.3 h vs 50.6 ± 0.3 h on four real metagenomes). The pipeline is reconfigurable with user-provided HMMs and/or reference nucleotide sequences to target any specialty gene set.

## Related tools

- **CasCollect** (Python-wrapped pipeline orchestrating seed generation, read-subset expansion, assembly and annotation; run in default protein mode with --meta for metagenomes) — https://github.com/sandialabs/CasCollect
- **HMMER** (Searches translated reads against the default 120 Cas protein family profile HMMs to identify seed reads)
- **FragGeneScan** (Open reading frame prediction on short reads prior to HMMER profile search) — https://github.com/hallamlab/FragGeneScanPlus
- **VSEARCH** (Read-subset expansion: identity matching of reads to seed set over five cycles at 95% identity) — https://github.com/torognes/vsearch
- **SPAdes** (Assembles the seed-plus-expanded read subset; metaSPAdes is used in place of SPAdes for metagenomes)
- **bbduk** (Optional trimming/cleaning of reads (quality trim, adapter removal, contaminant removal) before seed generation) — https://jgi.doe.gov/data-and-tools/bbtools/

## Examples

```
python CasCollect.py --meta reads_1.fastq reads_2.fastq
```

## Evaluation signals

- CasCollect pre-run check (or Check.py) confirms all required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) is present in the user's path
- On bacterial isolate genomes with known systems, all previously identified cas genes are recovered (as shown for E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39)
- On the simulated metagenome, assembled contigs span entire cas operons and contain the expected CRISPR arrays
- Targeted-assembly runtime is far below complete metaSPAdes assembly plus annotation (mean ~1.7 h vs ~50.6 h per metagenome, with 500 bp contig cutoff for annotation)
- HMMER search returns a non-trivial seed read set; empty or near-empty seed output suggests the HMM set or ORF prediction step failed

## Limitations

- No software version numbers or accession IDs are given for the third-party tools (HMMER, VSEARCH, SPAdes, CRISPRCasFinder) or datasets
- Default parameters (95% identity, five expansion cycles, 120 HMMs) may need adjustment via -match and -cycle for divergent or rare gene families
- In DNA mode, VSEARCH identity matching is restricted to nucleotide (CRISPR repeat) queries; protein targeting requires the HMM route
- Metagenomic runs default to 1 CPU and 20 Gb RAM, which may be slow or insufficient for very large datasets

## Evidence

- [results] CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes: "we developed CasCollect. CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [results] Default 120 Cas protein HMMs used by HMMER for seeding: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] VSEARCH expansion cycles and 95% identity default: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] Five expansion cycles with removal of matched reads: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] metaSPAdes used for metagenomes: "For metagenomes, metaSPAdes is used in-place-of SPAdes."
- [results] Pipeline relies on HMMER, VSEARCH, SPAdes, CRISPRCasFinder: "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [abstract] Runtime improvement of almost two orders of magnitude: "Benchmarking our targeted assembly pipeline demonstrates significantly improved timing by almost two orders of magnitude compared with conventional assembly and annotation"
- [readme] CasCollect requires BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"

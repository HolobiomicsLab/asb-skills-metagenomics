---
name: coverage-progression-benchmark-analysis
description: Use when quantifying whether iterative seed-expansion cycles (CasCollect
  -cycle 0..N) progressively close coverage gaps on a reference contig, and when a
  full-dataset baseline coverage comparison is needed.
when_to_use_negative:
- The target region cannot be represented as a single reference contig (no operon
  FASTA to map reads against), since the analysis is reference-based coverage measurement
- The question is assembly or annotation quality of cas operons rather than read-level
  coverage of an already-identified contig
tools:
- name: CasCollect
  role: Generates the seed read subsets at each expansion cycle count via '-cycle
    N', using the default 120 Cas protein HMMs in protein mode
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Performs the identity matching that drives read subset expansion, at the default
    95% match threshold
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: Identifies seed reads matching the Cas protein profile HMMs that prime expansion
- name: bowtie2
  role: Maps each cycle's seed read subset and the full raw dataset onto the cas operon
    reference in end-to-end paired-end mode
- name: SAMtools
  role: '''samtools depth -a'' extracts per-base pileup coverage at every reference
    position for coverage-fraction computation'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:f61e70d95edb26e7de1f7b644023ba392c164c66975bd916fbbbc3863c41d2cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - HMMER
  - SAMtools
  - VSEARCH
  - bowtie2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# coverage-progression-benchmark-analysis

## Summary

Benchmarks how CasCollect read subset expansion cycles improve sequencing read coverage of a target cas operon by mapping each cycle's seed read subset (plus the full raw dataset baseline) onto a reference contig and computing the fraction of operon bases covered.

## When to use

Use when quantifying whether iterative seed-expansion cycles (CasCollect -cycle 0..N) progressively close coverage gaps on a reference contig, and when a full-dataset baseline coverage comparison is needed.

## When NOT to use

- The target region cannot be represented as a single reference contig (no operon FASTA to map reads against), since the analysis is reference-based coverage measurement
- The question is assembly or annotation quality of cas operons rather than read-level coverage of an already-identified contig

## Inputs

- Paired-end Illumina FASTQ reads from a raw metagenomic dataset (e.g. SRA accession SRR10598175)
- Reference FASTA of the target cas operon contig (e.g. largest ground water cas operon contig)
- CasCollect default set of 120 Cas protein family profile HMMs

## Outputs

- cas_operon_coverage_progression.csv: percent operon coverage versus expansion cycle count (0-5) plus full-dataset baseline
- Per-cycle bowtie2 BAM/SAM alignments of seed read subsets against the cas operon reference
- Per-base pileup depth tracks from 'samtools depth -a' for each read set

## How to apply

Download the raw paired-end FASTQ dataset (e.g. ground water metagenome SRR10598175) and extract the largest cas operon contig as a reference FASTA. For each cycle count from 0 to 5, run CasCollect in default protein mode (-fwd/-rev/-out with the default 120 Cas protein HMMs) overriding the default of five expansion cycles with '-cycle N', so vsearch-driven seed expansion stops after N rounds at the default 95% identity match; matched reads are removed from the database between cycles to avoid redundant searches. Map each cycle's seed read subset, plus the complete raw dataset, onto the operon reference with bowtie2 in end-to-end paired-end mode. Extract per-base coverage for every reference position with 'samtools depth -a' and compute the fraction of operon bases covered by each read set. Tabulate percent operon coverage versus cycle count together with the full-dataset baseline into a CSV (e.g. cas_operon_coverage_progression.csv) and check that the progression reproduces the reported values (~33.2% seed, 96.1% after one cycle, 99.5% after two cycles, within a few percent tolerance) with saturation by cycle two.

## Related tools

- **CasCollect** (Generates the seed read subsets at each expansion cycle count via '-cycle N', using the default 120 Cas protein HMMs in protein mode) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Performs the identity matching that drives read subset expansion, at the default 95% match threshold) — https://github.com/torognes/vsearch
- **HMMER** (Identifies seed reads matching the Cas protein profile HMMs that prime expansion)
- **bowtie2** (Maps each cycle's seed read subset and the full raw dataset onto the cas operon reference in end-to-end paired-end mode)
- **SAMtools** ('samtools depth -a' extracts per-base pileup coverage at every reference position for coverage-fraction computation)

## Examples

```
CasCollect -fwd SRR10598175_1.fastq.gz -rev SRR10598175_2.fastq.gz -out cc_cycle1 -cycle 1
```

## Evaluation signals

- Cycle 0 seed set yields approximately 33.2% operon coverage; after one expansion cycle ~96.1% with no coverage gaps; after two cycles ~99.5% (within a few percent tolerance)
- Coverage is monotonically non-decreasing across cycles 0-2 and saturates by cycle two
- Full raw dataset baseline coverage is reproduced as a sanity check that mapping and depth extraction are correct
- 'samtools depth -a' output covers every reference position (zero-depth positions retained), so the covered-base fraction denominators match the operon length

## Limitations

- The default number of read subset expansion cycles is five and the default identity match is 95%; changing '-cycle' or '-match' alters the progression being benchmarked
- Matched reads are removed from the database between cycles, so coverage gains depend on the elimination of redundant searches and cannot simply be re-run cumulatively
- Percentages are sequence-identity and HMM-set dependent: results are specific to the default 120 Cas protein HMMs and the chosen reference operon

## Evidence

- [results] Default expansion cycles adjustable: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] Default 95% identity match: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] Reads removed between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [methods] VSEARCH drives identity matching: "-match number       percent match for vsearch"

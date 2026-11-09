---
name: read-subset-expansion-parameter-tuning
description: Use when running CasCollect on raw paired-end FASTQ data and you need
  to decide how many expansion cycles or what identity threshold will yield sufficient
  read coverage of a target CRISPR/Cas locus without redundant searching.
when_to_use_negative:
- The input is already assembled contigs or an annotated genome rather than raw unassembled
  sequencing reads — CasCollect's seed-and-expansion design targets raw FASTQ data
- You need whole-metagenome or whole-genome assembly rather than targeted recovery
  of CRISPR/Cas regions
- The target gene set has no suitable profile HMMs or repeat references, since seed
  generation depends on HMM profiles (protein mode) or CRISPR array repeats (DNA mode
  via VSEARCH)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CasCollect
  role: Pipeline that performs seed generation and vsearch-driven read subset expansion;
    -cycle and -match flags control expansion depth and identity
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Implements the identity matching that expands the seed read subset each cycle
    at the default 95% match threshold
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: Scans reads against the default 120 Cas protein family profile HMMs to generate
    seed reads
- name: bowtie2
  role: Maps each cycle's seed read subset (and full dataset baseline) end-to-end
    in paired-end mode onto the operon reference to measure coverage
- name: SAMtools
  role: '''samtools depth -a'' yields per-base pileup coverage for computing the fraction
    of operon bases covered'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:5a112b2996d7b6621bdbaf7bbbd70ac323cf02fc3fc197f9487f113fa343febc
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
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# read-subset-expansion-parameter-tuning

## Summary

Tuning CasCollect's vsearch-driven read subset expansion (default 95% identity, five cycles via -match/-cycle) to control how seed read subsets grow around cas-gene seeds, and assessing whether added cycles close coverage gaps on a target operon.

## When to use

Use when running CasCollect on raw paired-end FASTQ data and you need to decide how many expansion cycles or what identity threshold will yield sufficient read coverage of a target CRISPR/Cas locus without redundant searching.

## When NOT to use

- The input is already assembled contigs or an annotated genome rather than raw unassembled sequencing reads — CasCollect's seed-and-expansion design targets raw FASTQ data
- You need whole-metagenome or whole-genome assembly rather than targeted recovery of CRISPR/Cas regions
- The target gene set has no suitable profile HMMs or repeat references, since seed generation depends on HMM profiles (protein mode) or CRISPR array repeats (DNA mode via VSEARCH)

## Inputs

- Raw paired-end Illumina FASTQ read files (e.g. SRA accession SRR10598175 ground water metagenome)
- Reference FASTA of the target cas operon contig (for coverage benchmarking)
- Default CasCollect Cas protein profile HMM set (120 HMMs) or user-supplied HMMs

## Outputs

- Seed read subset FASTQ files per expansion cycle count
- CasCollect assembled/annotated cas operon contigs from the expanded subset
- Per-base pileup coverage (samtools depth -a) and percent-coverage table per cycle count (e.g. cas_operon_coverage_progression.csv)

## How to apply

Run CasCollect in default protein mode, which identifies seed reads matching the default 120 Cas protein profile HMMs via HMMER, then let vsearch expand the seed subset at the default 95% identity (-match to adjust). Override the default of five expansion cycles with '-cycle N' when you want to stop expansion at a specific round; matched reads are removed from the database between cycles to eliminate redundant searches. To diagnose whether expansion is still adding useful reads, map each cycle's seed read subset (cycle 0, 1, 2, ...) onto the target operon reference with bowtie2 end-to-end paired-end mode, extract per-base coverage with 'samtools depth -a', and compute the fraction of reference bases covered. Coverage typically saturates quickly — for a ground water cas operon, one cycle raised coverage from ~33.2% to 96.1% with no gaps, and a second cycle reached 99.5% — so compare against the full-dataset baseline to judge diminishing returns.

## Related tools

- **CasCollect** (Pipeline that performs seed generation and vsearch-driven read subset expansion; -cycle and -match flags control expansion depth and identity) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Implements the identity matching that expands the seed read subset each cycle at the default 95% match threshold) — https://github.com/torognes/vsearch
- **HMMER** (Scans reads against the default 120 Cas protein family profile HMMs to generate seed reads)
- **bowtie2** (Maps each cycle's seed read subset (and full dataset baseline) end-to-end in paired-end mode onto the operon reference to measure coverage)
- **SAMtools** ('samtools depth -a' yields per-base pileup coverage for computing the fraction of operon bases covered)

## Examples

```
CasCollect -fwd reads_1.fq.gz -rev reads_2.fq.gz -out gw_cas_cycle1 -cycle 1
```

## Evaluation signals

- Coverage progression is monotone and saturates: expect ~33.2% at cycle 0, ~96.1% at cycle 1 (no gaps), ~99.5% at cycle 2 for the ground water cas operon benchmark, within a few percent tolerance
- Per-base pileup from 'samtools depth -a' has zero positions at depth 0 after sufficient expansion cycles (no coverage gaps)
- Increasing -cycle beyond saturation adds little coverage; verify cycle-2 to cycle-5 coverage gain is marginal before paying extra runtime
- Read subsets shrink appropriately across cycles because matched reads are removed from the database between cycles (no redundant re-searching)
- Assembled contigs from the expanded subset pass CRISPRCasFinder annotation with a 500 bp contig size cutoff

## Limitations

- Expansion defaults to five cycles and 95% identity; alternative thresholds require explicit -cycle/-match overrides and may trade sensitivity for subset size
- The 95% identity match in vsearch expansion may miss divergent flanking reads, leaving residual coverage gaps if saturation is not reached
- No software version numbers are given for CasCollect or its third-party dependencies, complicating exact reproduction
- Coverage gains beyond cycle 1–2 are marginal for well-covered loci, so very high cycle counts mostly increase runtime

## Evidence

- [results] Default expansion cycles: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] Default identity threshold: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] Redundancy elimination between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [other] Coverage finding: [evidence span withheld — anchor status: not_found]
- [results] HMM seed set: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"

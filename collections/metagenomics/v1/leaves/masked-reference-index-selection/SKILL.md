---
name: masked-reference-index-selection
description: Use when cleaning microbial FASTQ reads against a human host reference
  and microbial retention (not just host removal) is a priority, e.g. bacterial Illumina
  or ONT metagenomic reads that are being over-removed by the unmasked human-t2t-hla
  index.
when_to_use_negative:
- When the target organism is viral or phage and retention of viral sequences is the
  priority — use the index masked against RefSeq viral and Millard Lab phage genomes
  (human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401) instead
- When maximal human read removal matters more than microbial retention — the default
  unmasked human-t2t-hla index is precise by default but masked indexes slightly relax
  depletion (≤0.001% reduction)
- When input data contains no microbial host-organism overlap concern, e.g. pure host
  sequencing, where masking provides no benefit
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Primary decontamination tool; fetches masked indexes (hostile index fetch)
    and removes host-aligned reads via hostile clean
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Alignment backend for short (paired Illumina) read decontamination, run with
    --sensitive and -k 1
- name: Minimap2
  role: Alignment backend for ONT long-read decontamination with -x map-ont preset;
    also used by hostile mask to align 150mers for masking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:7b89738fcd603aea9def377f6f1235c7d49cb46f6463bee122a7a0a7871d91fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - Hostile
  - Minimap2
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# masked-reference-index-selection

## Summary

Select and apply Hostile's masked human reference indexes (e.g. human-t2t-hla-argos985-mycob140, masked with 150mers from 985 FDA-ARGOS bacterial and 140 mycobacterial genomes) to maximize bacterial read retention during host depletion while keeping human read removal essentially unchanged.

## When to use

Use when cleaning microbial FASTQ reads against a human host reference and microbial retention (not just host removal) is a priority, e.g. bacterial Illumina or ONT metagenomic reads that are being over-removed by the unmasked human-t2t-hla index.

## When NOT to use

- When the target organism is viral or phage and retention of viral sequences is the priority — use the index masked against RefSeq viral and Millard Lab phage genomes (human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401) instead
- When maximal human read removal matters more than microbial retention — the default unmasked human-t2t-hla index is precise by default but masked indexes slightly relax depletion (≤0.001% reduction)
- When input data contains no microbial host-organism overlap concern, e.g. pure host sequencing, where masking provides no benefit

## Inputs

- Paired-end bacterial Illumina FASTQ(.gz) reads (short-read cleaning via Bowtie2)
- Single-end bacterial ONT long-read FASTQ(.gz) files
- Simulated mycobacterial Illumina paired and ONT read sets
- Real human read benchmarks (1000 Genomes Project samples) or simulated human reads for depletion control
- Masked or unmasked Hostile index (human-t2t-hla or human-t2t-hla-argos985-mycob140)

## Outputs

- Cleaned FASTQ files (e.g. long.clean.fastq.gz) with host reads removed
- JSON clean logs with reads_in, reads_out, reads_removed, reads_removed_proportion
- Comparison table (e.g. masked_retention_results.csv) of retention and human-depletion metrics for masked vs unmasked indexes

## How to apply

Fetch the appropriate masked index with 'hostile index fetch --name <index>' (e.g. human-t2t-hla-argos985-mycob140, masked with 150mers of 985 FDA-ARGOS bacterial and 140 mycobacterial genomes, 319,752 masked positions/0.010%). Pass the index name via --index to 'hostile clean': for paired short reads Hostile uses Bowtie2 (--sensitive, -k 1, no secondary alignments); for ONT long reads it uses Minimap2 (-x map-ont, --secondary no). The rationale is that masking removes bacterial-matching 150mers from the human reference so microbial reads no longer align spuriously. Compare retention against the unmasked default human-t2t-hla index by parsing the JSON clean logs (reads_in, reads_out, reads_removed, reads_removed_proportion) and computing microbial retention percentages; the paper found masked use raises bacterial retention to ≥99.997% (up to 99.99994% for simulated bacterial Illumina) with ≤0.001% reduction in human read removal.

## Related tools

- **Hostile** (Primary decontamination tool; fetches masked indexes (hostile index fetch) and removes host-aligned reads via hostile clean) — https://github.com/bede/hostile
- **Bowtie2** (Alignment backend for short (paired Illumina) read decontamination, run with --sensitive and -k 1)
- **Minimap2** (Alignment backend for ONT long-read decontamination with -x map-ont preset; also used by hostile mask to align 150mers for masking)

## Examples

```
hostile index fetch --name human-t2t-hla-argos985-mycob140; hostile clean --fastq1 reads_1.fastq.gz --fastq2 reads_2.fastq.gz --index human-t2t-hla-argos985-mycob140
```

## Evaluation signals

- Masked-index run shows bacterial Illumina retention of 99.99994% and exactly 251 ONT bacterial reads removed
- 100% retention of masked mycobacterial Illumina and ONT reads
- Human retention increases stay within marginal bounds: Illumina 0.131738% to 0.131979%; ONT 0.038029% to 0.038069%
- Real human read removal remains ≥99.6% (i.e. human removal loss ≤0.001% vs the unmasked index)
- Simulated bacterial retention across all runs meets the ≥99.989% (default) and ≥99.997% (masked) thresholds from the abstract

## Limitations

- Masked positions are limited (e.g. 0.010% for human-t2t-hla-argos985-mycob140), so gains apply only to organisms whose 150mers were used for masking; other taxa may still be over-removed
- Hostile requires 4GB RAM for short reads (Bowtie2) and 13GB for long reads (Minimap2)
- Downloaded index checksums were not verified in earlier Hostile versions (verification added only in latest release, issue #20)
- Prior Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination (issue #24)

## Evidence

- [abstract] Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance: "Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance"
- [methods] Masking uses Minimap2 to align 150mers of the supplied target genomes with the reference genome, and bedtools to mask all aligned regions with N.: "Masking uses Minimap2 to align 150mers of the supplied target genomes with the reference genome, and bedtools to mask all aligned regions with N."
- [readme] masked with 150mers for 985 FDA-ARGOS bacterial & 140 mycobacterial genomes: "`human-t2t-hla` masked with 150mers for [985](https://github.com/bede/hostile/blob/main/paper/supplementary-table-2.tsv) [FDA-ARGOS](https://www.ncbi.nlm.nih.gov/bioproject/231221) **bacterial** &"
- [abstract] our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [methods] To optionally download and cache the default index (`human-t2t-hla`) ahead of time, run `hostile index fetch`.: "To optionally download and cache the default index (`human-t2t-hla`) ahead of time, run `hostile index fetch`."

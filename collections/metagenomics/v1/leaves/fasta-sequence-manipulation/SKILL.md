---
name: fasta-sequence-manipulation
description: Use when a metagenomic assembly needs coverage-based edge correction
  of contigs — e.g. FASTA contigs plus a k-mer depth table where terminal regions
  of suspected interspecies misjoins show coverage deviating from the contig median.
when_to_use_negative:
- Input is raw short reads rather than assembled contig FASTA — this skill operates
  on contig records, not read-level data
- Misassemblies must be corrected using long-range links (mate-pair/long reads) rather
  than coverage depth — MetaPlatanus' scaffolding/link modules, not edge trimming,
  are the right mechanism
- Coverage depth table is absent, so the 1.5x median-deviation condition cannot be
  evaluated for any contig
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MetaPlatanus
  role: Defines the coverage-depth edge-correction strategy and the k-mer coverage
    cutoff convention (default 4 with MEGAHIT, 2 otherwise) that the trimming reproduces
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Assembler of the initial contigs whose FASTA records are manipulated; its
    use sets the default min_cov_contig cutoff to 4
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:2560778e9f4ff60a740f7836314fb19ef5ea2bf4943e6f3c743dacb6015c11fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - MEGAHIT
  - MetaPlatanus
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# fasta-sequence-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Split and trim contig FASTA records by per-contig k-mer coverage depth, implementing MetaPlatanus' contig edge-correction: edge regions whose local coverage deviates more than 1.5x from the contig median are deleted to remove interspecies misjoinings within contigs.

## When to use

Use when a metagenomic assembly needs coverage-based edge correction of contigs — e.g. FASTA contigs plus a k-mer depth table where terminal regions of suspected interspecies misjoins show coverage deviating from the contig median.

## When NOT to use

- Input is raw short reads rather than assembled contig FASTA — this skill operates on contig records, not read-level data
- Misassemblies must be corrected using long-range links (mate-pair/long reads) rather than coverage depth — MetaPlatanus' scaffolding/link modules, not edge trimming, are the right mechanism
- Coverage depth table is absent, so the 1.5x median-deviation condition cannot be evaluated for any contig

## Inputs

- Contig sequences in FASTA format (synthetic or assembled contigs)
- Per-contig (or per-base) k-mer coverage depth table
- Optional assembler context flag (MEGAHIT vs other) to set min_cov_contig (4 or 2)

## Outputs

- trimmed_contigs.fasta — FASTA with edge segments deleted
- edge_trim_report.tsv — per-contig original length, trimmed length, median coverage, and trimmed-edge coordinates

## How to apply

Compute per-contig k-mer coverage depths and each contig's median coverage from the depth table. Scan from both ends of each FASTA record and mark edge regions where local coverage violates the 1.5x deviation condition relative to the contig median; split the record at the first violating region and discard the offending edge segment. Respect the k-mer coverage cutoff convention MetaPlatanus uses for contig assembly (-min_cov_contig: default 4 with MEGAHIT, 2 otherwise) when deciding which contig regions are usable. Write trimmed records to trimmed_contigs.fasta and report original length, trimmed length, median coverage, and trimmed-edge coordinates per contig in edge_trim_report.tsv. Rationale: MetaPlatanus employs coverage depth and k-mer frequencies to correct mis-assemblies and reduce inter-species misassemblies, and deleting misjoin-edge segments is one such correction.

## Related tools

- **MetaPlatanus** (Defines the coverage-depth edge-correction strategy and the k-mer coverage cutoff convention (default 4 with MEGAHIT, 2 otherwise) that the trimming reproduces) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Assembler of the initial contigs whose FASTA records are manipulated; its use sets the default min_cov_contig cutoff to 4) — https://github.com/voutcn/megahit

## Examples

```
python edge_trim.py --contigs contigs.fasta --depth kmer_depth.tsv --min_cov_contig 4 --out trimmed_contigs.fasta --report edge_trim_report.tsv
```

## Evaluation signals

- Each trimmed boundary in edge_trim_report.tsv falls within the expected region given the known synthetic coverage profile
- Every retained region satisfies the 1.5x deviation condition relative to the contig median coverage
- trimmed length <= original length for every contig, and trimmed length equals original length minus the reported edge coordinates
- Median coverage values in the report match medians recomputed from the depth table
- Output FASTA remains valid: headers preserved, no terminal N-padding introduced by the trim

## Limitations

- The edge-trim condition is coverage-based only; misjoinings not accompanied by a coverage-depth discontinuity at the contig edge will be missed
- The cutoff convention depends on assembler choice (4 with MEGAHIT, 2 otherwise); applying the wrong default can over- or under-trim low-coverage contigs
- Low-abundance species have few reads, so per-contig coverage estimates are noisy and the 1.5x median comparison may be unstable

## Evidence

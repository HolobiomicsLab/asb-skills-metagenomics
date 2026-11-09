---
name: fungal-protein-database-alignment
description: Use when metagenomic sequencing reads must be assigned fungal functions
  via a fungal protein database, and the Diamond blastx threshold combination (identity,
  read coverage, e-value) needs to be selected or applied for fungal functional profiling.
when_to_use_negative:
- Input is a nucleotide/amplicon dataset intended for taxonomic profiling — use the
  FunOMIC-T marker-gene mapping route instead of Diamond blastx against FunOMIC-P
- A bacterial contamination filtration step is required before functional annotation
  — the current pipeline has none, so reads must be pre-filtered elsewhere
- The database needed is not fungal (e.g., bacterial-only functional profiling against
  a non-fungal protein set)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Diamond blastx v2.0.8
  role: Aligns sequencing reads (translated DNA vs. protein) against the FunOMIC-P
    fungal protein database with an e-value < 10e-10
  repo: https://github.com/bbuchfink/diamond
- name: FunOMIC
  role: Pipeline and source of the built-in FunOMIC-P functional database (3.4 million
    non-redundant fungal proteins)
  repo: https://github.com/ManichanhLab/FunOMIC
- name: ART
  role: Simulates Illumina-format sequencing reads from mixed mock community fungal
    and bacterial coding genes for threshold optimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:d08849f5fa06f660eedf22af2d6f1bf8dff432e4a38c6c9698d4ecf17c449149
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - ART
  - Diamond blastx v2.0.8
  - FunOMIC
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fungal-protein-database-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align (meta)genomic sequencing reads against the FunOMIC-P fungal protein database using Diamond blastx with tuned identity/coverage/e-value thresholds to achieve fungal functional annotation, as established in FunOMIC's mock-community threshold optimization.

## When to use

Use when metagenomic sequencing reads must be assigned fungal functions via a fungal protein database, and the Diamond blastx threshold combination (identity, read coverage, e-value) needs to be selected or applied for fungal functional profiling.

## When NOT to use

- Input is a nucleotide/amplicon dataset intended for taxonomic profiling — use the FunOMIC-T marker-gene mapping route instead of Diamond blastx against FunOMIC-P
- A bacterial contamination filtration step is required before functional annotation — the current pipeline has none, so reads must be pre-filtered elsewhere
- The database needed is not fungal (e.g., bacterial-only functional profiling against a non-fungal protein set)

## Inputs

- Illumina-format FASTQ sequencing reads (e.g., simulated from mock community coding genes with ART)
- FunOMIC-P functional database (3,413,239 non-redundant fungal proteins, 95% identity clustered)
- Fungal coding gene sequences (CDS) of mock community members for simulation
- Known fungal vs. bacterial read origin labels for evaluating threshold combinations

## Outputs

- Diamond blastx alignment hits (tabular) of reads against FunOMIC-P
- Per-threshold-combination retained fungal read hit rates (nine combinations)
- Final fungal functional annotation of reads at coverage >95%, identity >99%, e-value < 10e-10

## How to apply

Download the FunOMIC-P database (3.4 million non-redundant fungal proteins from 4,816 fungal genomes plus the fungal UniProt subset, clustered at 95% identity with CD-HIT) from https://manichanh.vhir.org/funomic/. Align input reads with Diamond blastx (v2.0.8 in the article) using a permissive e-value cutoff of < 10e-10 in every run. To select thresholds, sweep a 3x3 grid of identity percentage (>90%, >95%, >99%) crossed with read coverage (>90%, >95%, >99%), computing for each combination the fraction of fungal reads whose hits pass the filters; assemble the nine pass rates into a single tab-separated table. The article's validation shows read coverage >95% with identity >99% yields the highest retained fungal read hit rate (~70%) and is therefore adopted for fungal functional annotation. High inter-kingdom conservation of protein-coding genes means bacterial reads may still pass, so interpret functional hits from bacteria-dominated samples with care.

## Related tools

- **Diamond blastx v2.0.8** (Aligns sequencing reads (translated DNA vs. protein) against the FunOMIC-P fungal protein database with an e-value < 10e-10) — https://github.com/bbuchfink/diamond
- **FunOMIC** (Pipeline and source of the built-in FunOMIC-P functional database (3.4 million non-redundant fungal proteins)) — https://github.com/ManichanhLab/FunOMIC
- **ART** (Simulates Illumina-format sequencing reads from mixed mock community fungal and bacterial coding genes for threshold optimization)

## Examples

```
diamond blastx -d FunOMIC-P.dmnd -q mock_reads.fq -e 1e-10 -o hits.tsv --outfmt 6; awk '$3>99 && $4>95' hits.tsv | wc -l
```

## Evaluation signals

- The threshold sweep table contains all nine combinations (identity >90/>95/>99% x coverage >90/>95/>99%)
- The combination with read coverage >95% and identity percentage >99% shows the highest retained fungal read hit rate (~70%), confirming it as optimal
- Bacterial mock reads (e.g., the 999,021-read specificity test) map at very low rates; specificity ~0.9999 as in the FunOMIC-T test
- All runs apply e-value < 10e-10 consistently so threshold differences reflect identity/coverage only

## Limitations

- Bacterial contamination is not totally preventable due to high inter-kingdom conservation of protein-coding genes, even under the strictest mapping threshold
- No bacterial read filtration step is included before functional annotation; it is only proposed for a future update
- Around 70% of fungal read hits map at the optimal threshold, so roughly 30% of true fungal hits are still lost under the strictest filters

## Evidence

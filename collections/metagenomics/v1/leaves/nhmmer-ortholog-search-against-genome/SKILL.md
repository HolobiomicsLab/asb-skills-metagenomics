---
name: nhmmer-ortholog-search-against-genome
description: Use when KEGG module evaluation reports missing KOs in a MAG/genome and
  you want to recover unannotated or truncated gene sequences via profile HMM searches
  against the genome's proteins.
when_to_use_negative:
- When the input already contains complete, trusted KO annotations and only KEGG module
  completeness summarization is needed — use --skip_hmm or --no_genome instead
- When genome-scale model gapfilling rather than annotation recovery is the goal —
  that requires --gsmm_mode (denovo or existing) in addition to the HMM search
- When the target KOs are not represented in the selected BRITE clade of KEGG GENES,
  since profiles are built only from the subset indicated by --hmm_mode
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: KEMET
  role: 'Command-line Python tool orchestrating the HMM-based missing-KO search: builds
    profiles, runs nhmmer, applies the quality filter, and writes hmm_search_results.tsv'
  repo: https://github.com/Matteopaluh/KEMET
- name: HMMER
  role: Suite (v3.1b2) supplying hmmbuild to construct profile HMMs from MAFFT alignments
    and nhmmer to search profiles against genome protein sequences
- name: MAFFT
  role: Builds the non-redundant multiple sequence alignment of KEGG GENES orthologs
    from which the profile HMM is generated
- name: Python
  role: Implementation language of the kemet.py script driving the workflow
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:963da7a54390af9a8de29df6f72759c5fdb610d77fb440b0bd77a63a5fae1cbb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - HMMER
  - KEMET
  - MAFFT
  - Python
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nhmmer-ortholog-search-against-genome

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover missing KEGG ortholog (KO) annotations from MAG/genome FASTA files by searching profile HMMs (built with hmmbuild from KEGG GENES ortholog alignments) against the input protein sequences using nhmmer, as implemented in KEMET's HMM-based search module.

## When to use

Use when KEGG module evaluation reports missing KOs in a MAG/genome and you want to recover unannotated or truncated gene sequences via profile HMM searches against the genome's proteins.

## When NOT to use

- When the input already contains complete, trusted KO annotations and only KEGG module completeness summarization is needed — use --skip_hmm or --no_genome instead
- When genome-scale model gapfilling rather than annotation recovery is the goal — that requires --gsmm_mode (denovo or existing) in addition to the HMM search
- When the target KOs are not represented in the selected BRITE clade of KEGG GENES, since profiles are built only from the subset indicated by --hmm_mode

## Inputs

- MAG/genome FASTA file (nucleotide sequences; proteins are derived for the HMM search)
- Optional precomputed KEGG annotation file (.emapper.annotations from eggNOG-mapper, .ko from KAAS, or kofamkoala output)
- KEGG GENES database subset downloaded via the KEGG API (BRITE C-level clade restricted to target KOs)

## Outputs

- hmm_search_results.tsv with recovered KO annotations passing the 0.43 default threshold
- Profile HMM models built with hmmbuild from MAFFT non-redundant alignments
- Score-per-HMM-length values for each candidate nhmmer hit

## How to apply

First retrieve a taxonomically relevant subset of the KEGG GENES database via the KEGG API, restricted to the C-level BRITE functional clade containing the target KOs, then cluster and deduplicate the ortholog sequences and build a non-redundant multiple sequence alignment with MAFFT. Build a profile HMM from that alignment with hmmbuild (HMMER v3.1b2), covering only the subset of KOs selected by the --hmm_mode argument (onebm, module, or kos). Search this profile HMM against the protein sequences of the input MAG/genome FASTA with nhmmer, computing for each candidate hit a score normalized by the HMM length. Retain hits whose score-per-HMM-length exceeds the default quality filter of 0.43 (overridable with --threshold_value) and write the recovered KO annotations to hmm_search_results.tsv. This step can be run in isolation by invoking kemet.py with --hmm_mode and --skip_gsmm, or skipped entirely with --skip_hmm after only the module completeness evaluation.

## Related tools

- **KEMET** (Command-line Python tool orchestrating the HMM-based missing-KO search: builds profiles, runs nhmmer, applies the quality filter, and writes hmm_search_results.tsv) — https://github.com/Matteopaluh/KEMET
- **HMMER** (Suite (v3.1b2) supplying hmmbuild to construct profile HMMs from MAFFT alignments and nhmmer to search profiles against genome protein sequences)
- **MAFFT** (Builds the non-redundant multiple sequence alignment of KEGG GENES orthologs from which the profile HMM is generated)
- **Python** (Implementation language of the kemet.py script driving the workflow) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --skip_gsmm --threshold_value 0.43 --log
```

## Evaluation signals

- hmm_search_results.tsv is produced and contains only hits whose score normalized by HMM length exceeds the chosen --threshold_value (default 0.43)
- Mock-removal benchmarks in the paper recovered 20/36 (55% TPR) for MAGs and 8/12 (67% TPR) for complete genomes — partial recovery at these rates is consistent with correct application
- HMM profiles correspond only to the KO subset indicated by --hmm_mode (onebm, module, or kos), not the full KEGG GENES database
- Truncated gene tests (30% and 70% deletion of original length) should still yield hits for recovered KOs, confirming the profile tolerates partial sequences
- Running with --skip_gsmm stops cleanly after the HMM analysis; running with --skip_hmm should produce only the KEGG MODULES recap tables and no hmm_search_results.tsv

## Limitations

- Mock-removal recovery is partial: only 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes, so not every removed KO is recovered
- The threshold of 0.43 is a trade-off — hits below it are discarded even if biologically real; users may override with --threshold_value but no cross-validated optimum is documented
- Search scope is limited to the KOs in the --hmm_mode subset of the KEGG GENES database; KOs outside the selected BRITE clade are never searched

## Evidence

- [methods] KEMET builds its search profiles by generating an HMM from the aligned ortholog sequences with the hmmbuild command from the HMMER suite v3.1b2, using only the subset of KOs indicated in the --hmm_mode argument.: [evidence span withheld — anchor status: not_found]
- [methods] Search the profile HMM against the protein sequences of the input MAG/genome FASTA file using nhmmer, computing the score normalized by HMM length for each candidate hit.: [evidence span withheld — anchor status: not_found]

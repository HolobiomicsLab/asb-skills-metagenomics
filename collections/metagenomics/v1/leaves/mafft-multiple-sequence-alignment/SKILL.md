---
name: mafft-multiple-sequence-alignment
description: Use when a set of candidate ortholog sequences for target KOs must be
  aligned into a non-redundant MSA so a profile HMM can be built and searched against
  protein sequences of a MAG/genome to recover missing KEGG orthologs.
when_to_use_negative:
- When only KEGG Module completeness evaluation is needed and no missing-KO recovery
  is desired — run KEMET with --skip_hmm to stop after the completeness evaluation
  instead
- When the input is a KEGG annotation file without a genome (use --no_genome), since
  no HMM search against genome sequences can be performed
- When a KO assignment tool such as kofamkoala, eggNOG-mapper, or KAAS has already
  annotated all KOs of interest and no annotation expansion is required
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: MAFFT
  role: Builds the non-redundant multiple sequence alignment of retrieved ortholog
    sequences from which the profile HMM is constructed
- name: HMMER
  role: hmmbuild constructs the profile HMM from the MAFFT alignment; nhmmer searches
    the HMM against input protein sequences computing score normalized by HMM length
- name: KEMET
  role: Command-line Python tool orchestrating the KEGG GENES retrieval, alignment,
    hmmbuild, and nhmmer search; --hmm_mode selects the KO subset and --threshold_value
    the quality filter
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Implementation language of the KEMET toolbox that wraps the alignment and
    HMM search steps
  repo: https://github.com/Matteopaluh/KEMET
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:e0619e8d175cfc8c3ef47515d154df788b7efa30875f8a9addf1cda52b9388ca
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
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mafft-multiple-sequence-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Within KEMET's missing-KO search workflow, MAFFT builds a non-redundant multiple sequence alignment of taxonomically relevant KEGG GENES ortholog sequences, which is then converted into a profile HMM with hmmbuild (HMMER v3.1b2) for nhmmer-based recovery of missing KOs from MAGs/genomes.

## When to use

Use when a set of candidate ortholog sequences for target KOs must be aligned into a non-redundant MSA so a profile HMM can be built and searched against protein sequences of a MAG/genome to recover missing KEGG orthologs.

## When NOT to use

- When only KEGG Module completeness evaluation is needed and no missing-KO recovery is desired — run KEMET with --skip_hmm to stop after the completeness evaluation instead
- When the input is a KEGG annotation file without a genome (use --no_genome), since no HMM search against genome sequences can be performed
- When a KO assignment tool such as kofamkoala, eggNOG-mapper, or KAAS has already annotated all KOs of interest and no annotation expansion is required

## Inputs

- Taxonomically relevant subset of the KEGG GENES database ortholog protein sequences for the selected KOs (retrieved via the KEGG API, restricted to the target C-level BRITE functional clade)
- MAG/genome FASTA file whose protein sequences are searched with nhmmer
- KEGG annotation file (eggnog / kaas / kofamkoala format) for the KEGG MODULES recap tables

## Outputs

- Non-redundant multiple sequence alignment of ortholog sequences produced by MAFFT
- Profile HMM built from the alignment with hmmbuild
- hmm_search_results.tsv file with recovered KO annotations passing the score-per-HMM-length quality filter

## How to apply

First retrieve a taxonomically relevant subset of the KEGG GENES database via the KEGG API, restricted to the C-level BRITE functional clade covering the target KOs (the subset selected via KEMET's --hmm_mode argument, e.g. onebm/module/kos). Cluster and deduplicate the retrieved ortholog sequences, then align them into a non-redundant multiple sequence alignment with MAFFT. Feed the MAFFT alignment to hmmbuild (HMMER v3.1b2) to construct the profile HMM, which nhmmer searches against the input MAG/genome FASTA protein sequences, scoring each hit normalized by HMM length. Retain hits whose score-per-HMM-length exceeds the quality filter (default 0.43, overridable with --threshold_value) and write recovered KO annotations to hmm_search_results.tsv. The rationale is that a curated ortholog alignment yields a sensitive HMM able to detect unannotated genes, including partially truncated sequences in MAGs.

## Related tools

- **MAFFT** (Builds the non-redundant multiple sequence alignment of retrieved ortholog sequences from which the profile HMM is constructed)
- **HMMER** (hmmbuild constructs the profile HMM from the MAFFT alignment; nhmmer searches the HMM against input protein sequences computing score normalized by HMM length)
- **KEMET** (Command-line Python tool orchestrating the KEGG GENES retrieval, alignment, hmmbuild, and nhmmer search; --hmm_mode selects the KO subset and --threshold_value the quality filter) — https://github.com/Matteopaluh/KEMET
- **Python** (Implementation language of the KEMET toolbox that wraps the alignment and HMM search steps) — https://github.com/Matteopaluh/KEMET

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --threshold_value 0.43
```

## Evaluation signals

- hmm_search_results.tsv is produced containing KO annotations for hits whose score-per-HMM-length exceeds the threshold (default 0.43)
- KO mock removal tests recovered the correct gene and annotation in 55% (20/36) of cases for MAGs and 67% (8/12) for complete genomes, so a comparable recovery rate is expected
- Running KEMET with --skip_hmm should stop before the HMM analysis, confirming the alignment/HMM stage is only active when --hmm_mode (onebm/module/kos) is set
- Recovered KOs should increase the completeness of the affected KEGG Modules in the KEGG MODULES recap tables generated from eggnog/kaas/kofamkoala annotations
- Recovered annotations can be validated by checking that genes with partial sequence truncations (30% or 70% deletion simulated in tests) are still detected by the HMM search

## Limitations

- Recovery is imperfect: KO mock removal tests achieved only 55% (20/36) true positive rate on MAGs and 67% (8/12) on complete genomes
- Detection depends on the score-per-HMM-length quality filter (default 0.43); hits below this threshold are discarded, and the threshold must be tuned via --threshold_value
- The profile HMM is only built for the subset of KOs indicated in the --hmm_mode argument (onebm/module/kos), so KOs outside that selection are never searched
- MAG-based analyses are affected by the gapped nature of metagenomic assemblies, though KEMET can still detect genes with partial sequence truncations

## Evidence

- [other] KEMET builds its search profiles by generating an HMM from the aligned ortholog sequences with the hmmbuild command from the HMMER suite v3.1b2, using only the subset of KOs indicated in the --hmm_mode argument.: [evidence span withheld — anchor status: not_found]
- [other] Cluster and deduplicate the retrieved ortholog sequences and build a non-redundant multiple sequence alignment with MAFFT.: [evidence span withheld — anchor status: not_found]

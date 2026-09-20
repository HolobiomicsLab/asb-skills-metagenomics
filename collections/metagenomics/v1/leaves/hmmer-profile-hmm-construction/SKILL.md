---
name: hmmer-profile-hmm-construction
description: Use when candidate KO annotations are missing from a MAG/genome annotation
  table (eggNOG, KAAS, KOFAMkoala) and you want to recover them via profile HMM searches
  against taxonomically relevant KEGG GENES ortholog subsets.
when_to_use_negative:
- Input has no genome/MAG FASTA available and only precomputed KEGG annotations —
  use --no_genome / --skip_hmm, as the pipeline stops after KEGG MODULES Completeness
  Evaluation
- The goal is only KEGG Module completeness summarization of existing annotations
  without searching for missing KOs — the --skip_hmm flag makes this skill unnecessary
- Input sequences are not protein-coding genes or the KOs of interest lack a taxonomically
  relevant subset of KEGG GENES sequences to build a reliable profile HMM
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: KEMET
  role: 'Orchestrates the HMM-based missing-KO search workflow: retrieves KEGG GENES
    subsets, runs MAFFT/hmmbuild/nhmmer, and applies the quality filter'
  repo: https://github.com/Matteopaluh/KEMET
- name: MAFFT
  role: Builds the non-redundant multiple sequence alignment of retrieved ortholog
    sequences from which the profile HMM is constructed
- name: HMMER
  role: Suite v3.1b2 providing hmmbuild to construct the profile HMM and nhmmer to
    search it against protein sequences, scoring hits normalized by HMM length
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:91a9dc8c9251c59976afa62a887cd9dfd7d99b0bc6d4170c07ac20d196c96669
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
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hmmer-profile-hmm-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct profile hidden Markov models from curated ortholog sequence alignments (hmmbuild, HMMER) and search them against protein sequences from MAGs/genomes (nhmmer) to recover missing KEGG ortholog (KO) annotations, as implemented in KEMET.

## When to use

Use when candidate KO annotations are missing from a MAG/genome annotation table (eggNOG, KAAS, KOFAMkoala) and you want to recover them via profile HMM searches against taxonomically relevant KEGG GENES ortholog subsets.

## When NOT to use

- Input has no genome/MAG FASTA available and only precomputed KEGG annotations — use --no_genome / --skip_hmm, as the pipeline stops after KEGG MODULES Completeness Evaluation
- The goal is only KEGG Module completeness summarization of existing annotations without searching for missing KOs — the --skip_hmm flag makes this skill unnecessary
- Input sequences are not protein-coding genes or the KOs of interest lack a taxonomically relevant subset of KEGG GENES sequences to build a reliable profile HMM

## Inputs

- MAG/genome FASTA file (protein sequences searched by nhmmer)
- KEGG GENES ortholog sequences retrieved via KEGG API for a C-level BRITE functional clade (subset of KOs specified via --hmm_mode: onebm/module/kos)
- KEGG KO annotation file in eggnog / kaas / kofamkoala format (e.g. .emapper.annotations or .ko) for Module completeness context

## Outputs

- Profile HMM files built by hmmbuild from non-redundant MAFFT alignments
- hmm_search_results.tsv containing recovered KO annotations passing the quality filter (score/HMM length > default 0.43)
- KEGG MODULES recap tables of metabolic potential (if HMM step is not skipped)

## How to apply

Retrieve a taxonomically relevant subset of the KEGG GENES database via the KEGG API, restricted to the C-level BRITE functional clade covering the target KOs (selected through the --hmm_mode argument, e.g. onebm/module/kos). Cluster and deduplicate the retrieved ortholog sequences and build a non-redundant multiple sequence alignment with MAFFT. Build a profile HMM from the non-redundant alignment using hmmbuild from the HMMER suite v3.1b2. Search the profile HMM against the protein sequences of the input MAG/genome FASTA with nhmmer, normalizing each hit score by the HMM length, and retain hits whose score-per-HMM-length exceeds the quality filter (default 0.43, overridable with --threshold_value); recovered KO annotations are written to hmm_search_results.tsv. This approach can recover unannotated or partially truncated genes in MAGs, increasing KEGG Module completeness and improving downstream GSMM quality.

## Related tools

- **KEMET** (Orchestrates the HMM-based missing-KO search workflow: retrieves KEGG GENES subsets, runs MAFFT/hmmbuild/nhmmer, and applies the quality filter) — https://github.com/Matteopaluh/KEMET
- **MAFFT** (Builds the non-redundant multiple sequence alignment of retrieved ortholog sequences from which the profile HMM is constructed)
- **HMMER** (Suite v3.1b2 providing hmmbuild to construct the profile HMM and nhmmer to search it against protein sequences, scoring hits normalized by HMM length)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --threshold_value 0.43
```

## Evaluation signals

- hmm_search_results.tsv is produced and contains only hits whose score normalized by HMM length exceeds the quality filter (default 0.43, or the --threshold_value used)
- The set of searched KOs matches the subset selected by the --hmm_mode argument (onebm, module, or kos)
- Mock-KO removal validation context: correct recovery of removed KOs is expected at roughly 55% true positive rate for MAGs (20/36) and 67% for complete genomes (8/12)
- Profile HMMs are built from non-redundant MAFFT alignments of KEGG GENES sequences restricted to the target C-level BRITE functional clade
- Recovered KO annotations propagate to increased KEGG Module completeness and, after CarveMe reconstruction and FVA, to changed metabolic capabilities relative to pre-search annotations

## Limitations

- Recovery of removed KOs is imperfect: 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes in mock removal tests
- Hits scoring at or below the default threshold of 0.43 (score per HMM length) are discarded, so borderline legitimate homologs may be missed; the threshold must be tuned via --threshold_value
- Lower-quality MAGs need care: issues with missingness, contamination, and short contigs can affect HMM search results; restrict to high-quality genome sets when in doubt (IGGdb guidance: >=90% completeness, <=5% contamination for high-quality MAGs)

## Evidence

- [other] KEMET builds its search profiles by generating an HMM from the aligned ortholog sequences with the hmmbuild command from the HMMER suite v3.1b2, using only the subset of KOs indicated in the --hmm_mode argument.: [evidence span withheld — anchor status: not_found]
- [other] workflow step from source card describing search and filtering.: [evidence span withheld — anchor status: not_found]

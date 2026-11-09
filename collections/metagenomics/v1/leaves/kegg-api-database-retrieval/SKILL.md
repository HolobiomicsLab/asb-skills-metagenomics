---
name: kegg-api-database-retrieval
description: Use when a workflow needs a curated, non-redundant set of KEGG ortholog
  sequences for HMM profile building (KEMET --hmm_mode onebm/module/kos) or when refreshing
  a stale local KEGG GENES copy via the KEGG API.
when_to_use_negative:
- When the user only needs KEGG MODULE completeness tables from existing annotations
  and can pass --skip_hmm, bypassing database retrieval and HMM search entirely
- When KOs are already covered by an existing annotation program (eggnog / kaas /
  kofamkoala) output and no annotation expansion is requested
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KEMET
  role: Orchestrates the KEGG API retrieval of the KEGG GENES subset for the selected
    --hmm_mode KOs and maintains the updatable local KEGG GENES database
  repo: https://github.com/Matteopaluh/KEMET
- name: MAFFT
  role: Aligns the deduplicated retrieved ortholog sequences into the non-redundant
    MSA used for hmmbuild
- name: HMMER
  role: hmmbuild constructs the profile HMM from the retrieved-and-aligned ortholog
    subset; nhmmer searches it against the input genome FASTA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:6ad32a6e3769065c388dc99b0fbd38280b66d2495f032bc035bd9a073abf74df
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kegg-api-database-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve a taxonomically relevant subset of the KEGG GENES database through the KEGG API, restricted to the C-level BRITE functional clade containing the target KOs, as the first step of KEMET's HMM-based missing-ortholog search. This keeps downstream profile HMMs trained on ortholog sequences related to the genome under analysis.

## When to use

Use when a workflow needs a curated, non-redundant set of KEGG ortholog sequences for HMM profile building (KEMET --hmm_mode onebm/module/kos) or when refreshing a stale local KEGG GENES copy via the KEGG API.

## When NOT to use

- When the user only needs KEGG MODULE completeness tables from existing annotations and can pass --skip_hmm, bypassing database retrieval and HMM search entirely
- When KOs are already covered by an existing annotation program (eggnog / kaas / kofamkoala) output and no annotation expansion is requested

## Inputs

- Target list of KEGG KOs (from --hmm_mode: onebm, module, or kos)
- C-level BRITE functional clade identifier delimiting the relevant KEGG GENES subset
- KEGG API access to the KEGG GENES database

## Outputs

- Taxonomically relevant, deduplicated subset of KEGG GENES ortholog sequences
- Non-redundant MAFFT multiple sequence alignment per KO
- Profile HMMs built with hmmbuild, consumed by nhmmer to produce hmm_search_results.tsv

## How to apply

Query the KEGG API to pull the subset of the KEGG GENES database corresponding to the selected C-level BRITE functional clade that contains the KOs indicated by --hmm_mode (onebm, module, or kos). Cluster and deduplicate the retrieved ortholog sequences to build a non-redundant sequence set, then align it with MAFFT to form the non-redundant multiple sequence alignment from which hmmbuild (HMMER v3.1b2) constructs the profile HMM. Restricting retrieval to the taxonomically/clade-relevant subset is the rationale: profiles trained on related orthologs better discriminate true hits in the input MAG/genome FASTA searched with nhmmer. KEMET also allows users to update the downloaded KEGG GENES database through the KEGG API, so re-run retrieval when the local copy is outdated before repeating the search.

## Related tools

- **KEMET** (Orchestrates the KEGG API retrieval of the KEGG GENES subset for the selected --hmm_mode KOs and maintains the updatable local KEGG GENES database) — https://github.com/Matteopaluh/KEMET
- **MAFFT** (Aligns the deduplicated retrieved ortholog sequences into the non-redundant MSA used for hmmbuild)
- **HMMER** (hmmbuild constructs the profile HMM from the retrieved-and-aligned ortholog subset; nhmmer searches it against the input genome FASTA)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --log
```

## Evaluation signals

- The retrieved subset must map to the C-level BRITE functional clade corresponding to the --hmm_mode KO selection (onebm/module/kos)
- Retrieved ortholog sequences are clustered and deduplicated before MAFFT, so the MSA is non-redundant (no duplicate ortholog entries)
- Downstream nhmmer hits must exceed the score-per-HMM-length quality filter (default 0.43, overridable with --threshold_value) to appear in hmm_search_results.tsv
- Local KEGG GENES database reflects a recent KEGG API update rather than a stale download

## Limitations

- The default 0.43 score-per-HMM-length quality filter differentiates legit HMM hits and may be too strict or permissive for some KO families; override with --threshold_value
- MAG-based analyses are prone to reconstruction/annotation errors due to gapped metagenomic assemblies, so recovered hits from partial sequences need care
- Retrieval depends on KEGG API availability and covers only the KOs indicated in --hmm_mode, not the full KEGG GENES database

## Evidence

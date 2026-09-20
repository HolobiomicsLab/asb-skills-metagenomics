---
name: microbial-genome-annotation-expansion
description: Use when a MAG or genome FASTA has partial KEGG KO annotations (e.g.
  from eggNOG-mapper, KAAS, or KofamKOALA) and the goal is to recover missing orthologs
  via HMM profile searches to expand KEGG Module completeness.
when_to_use_negative:
- Only KEGG Module completeness tables are needed and no annotation expansion is desired
  — use --skip_hmm to stop after the Completeness Evaluation instead
- Input is a KEGG annotation file without genome/MAG sequences and --no_genome is
  set — the workflow stops after KEGG MODULES Completeness Evaluation and no HMM search
  runs
- Annotation format is not one of the supported KEGG KO annotation programs (eggnog
  / kaas / kofamkoala)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KEMET
  role: Command-line Python toolbox orchestrating KEGG GENES retrieval, HMM profile
    construction, and KO annotation expansion
  repo: https://github.com/Matteopaluh/KEMET
- name: MAFFT
  role: Builds the non-redundant multiple sequence alignment of retrieved ortholog
    sequences
- name: HMMER
  role: hmmbuild constructs profile HMMs from alignments; nhmmer searches them against
    input protein sequences
- name: eggNOG-Mapper
  role: Produces the initial KO annotations (input format for -a) whose missing KOs
    the HMM search expands
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:2544a106e21f20fcc7194a1acc735cf236d10f44a5ebe3899bbc5236bf239070
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
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# microbial-genome-annotation-expansion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Expand KEGG functional annotation of MAGs/genomes by searching profile hidden Markov models (built with hmmbuild from taxonomically relevant KEGG GENES subsets aligned with MAFFT) against input protein sequences using nhmmer, recovering missing KEGG orthologs (KOs) that feed KEGG Module completeness evaluation and evidence-based GSMM gapfilling.

## When to use

Use when a MAG or genome FASTA has partial KEGG KO annotations (e.g. from eggNOG-mapper, KAAS, or KofamKOALA) and the goal is to recover missing orthologs via HMM profile searches to expand KEGG Module completeness.

## When NOT to use

- Only KEGG Module completeness tables are needed and no annotation expansion is desired — use --skip_hmm to stop after the Completeness Evaluation instead
- Input is a KEGG annotation file without genome/MAG sequences and --no_genome is set — the workflow stops after KEGG MODULES Completeness Evaluation and no HMM search runs
- Annotation format is not one of the supported KEGG KO annotation programs (eggnog / kaas / kofamkoala)

## Inputs

- MAG/genome FASTA file (e.g. genomes/bin1.fasta)
- KEGG annotation file in eggnog (.emapper.annotations), kaas, or kofamkoala (.ko) format
- Subset of KOs to search, indicated via --hmm_mode (onebm / module / kos)
- Taxonomically relevant KEGG GENES database subset retrieved via the KEGG API (downloadable/updatable by KEMET)

## Outputs

- hmm_search_results.tsv with recovered KO annotations for hits passing the score-per-HMM-length filter (default 0.43)
- KEGG MODULES completeness recap tables of metabolic potential
- Nucleotidic HMM search evidence used for evidence-based GSMM/GEM gapfilling

## How to apply

Run kemet.py with a FASTA file of the MAG/genome, a KEGG annotation format flag (-a eggnog/kaas/kofamkoala), and --hmm_mode (onebm, module, or kos) specifying the subset of KOs to search; omit --skip_hmm so the HMM stage executes. KEMET retrieves a taxonomically relevant subset of the KEGG GENES database via the KEGG API restricted to the target KOs, clusters and deduplicates the ortholog sequences, and builds a non-redundant multiple sequence alignment with MAFFT, from which hmmbuild (HMMER v3.1b2) constructs profile HMMs. These profiles are searched against the input protein sequences with nhmmer, scoring each candidate hit normalized by HMM length; hits exceeding the quality filter (default 0.43, overridable with --threshold_value) are written to hmm_search_results.tsv and propagated into KEGG MODULES completeness recap tables and GSMM gapfilling evidence. The approach also detects truncated genes, so partial sequences in MAGs can still be recovered.

## Related tools

- **KEMET** (Command-line Python toolbox orchestrating KEGG GENES retrieval, HMM profile construction, and KO annotation expansion) — https://github.com/Matteopaluh/KEMET
- **MAFFT** (Builds the non-redundant multiple sequence alignment of retrieved ortholog sequences)
- **HMMER** (hmmbuild constructs profile HMMs from alignments; nhmmer searches them against input protein sequences)
- **eggNOG-Mapper** (Produces the initial KO annotations (input format for -a) whose missing KOs the HMM search expands)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo --threshold_value 0.43 --log -v
```

## Evaluation signals

- hmm_search_results.tsv contains recovered KO hits whose score normalized by HMM length exceeds the quality filter (default 0.43, or the value passed to --threshold_value)
- KO mock removal tests recovered the correct gene and annotation in 20/36 (55%) of cases for MAGs and 8/12 (67%) for complete genomes, so expect similar true-positive rates on comparable inputs
- Truncated gene sequences with 30% deletion were still recovered; recovery drops when 70% of the sequence is deleted, so flag hits from severely truncated genes for review
- KEGG MODULES recap tables are produced with block structure closer to KEGG convention than METABOLIC, capturing more Modules in the evaluation

## Limitations

- HMM hit quality depends on the threshold: the default 0.43 score-per-HMM-length filter can be overridden with --threshold_value, but setting it inappropriately trades precision for recall
- Recovery of severely truncated sequences is imperfect — mock removals simulating 70% deletion of gene length reduced correct gene and annotation recovery
- Gene calling and initial annotation were validated only with Prodigal v2.6.3 and eggNOG-mapper v2 defaults; other annotation formats are limited to eggnog/kaas/kofamkoala

## Evidence

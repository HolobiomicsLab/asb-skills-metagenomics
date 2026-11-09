---
name: hmm-profile-ortholog-recovery
description: Use when eggNOG-mapper/.emapper.annotations or similar KO annotations
  are suspected incomplete (e.g. fragmented MAG assemblies or truncated genes) and
  missing KOs from KEGG Modules need evidence-backed recovery via HMM profile search.
when_to_use_negative:
- The goal is only KEGG Module completeness summarization with no need to expand annotations
  — use KEMET with --skip_hmm instead of HMM searches
- The genome annotation comes from a format other than eggnog/kaas/kofamkoala, since
  KEMET's -a parameter only supports these KO annotation formats
- Genes are severely fragmented (e.g. ~70% sequence truncation), where recovery rates
  decline markedly and HMM hits may not be trustworthy, especially in MAGs
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: KEMET
  role: Runs the HMM profile search (--hmm_mode onebm), applies the 0.43 quality threshold,
    evaluates KEGG Module completeness, and optionally gapfills GSMMs with recovered
    KO evidence
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Provides the initial KO annotations (.emapper.annotations) whose missing/incomplete
    KOs the HMM search aims to recover
- name: Prodigal
  role: Performs gene calling (v2.6.3, default options) to generate the ORFs searched
    with profile HMMs
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:51bc805529dba5cc614708be37bdbd8d656504acf248dbdd3ab5f431ee222e20
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
  - KEMET
  - Prodigal
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hmm-profile-ortholog-recovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover missing or unannotated KEGG orthologs (KOs) by searching KEGG Module-specific profile HMMs with KEMET's --hmm_mode onebm against genome/MAG protein sequences, using a default score threshold of 0.43 to call legit hits and expand KEGG Module completeness beyond sequence-similarity annotations like eggNOG-Mapper.

## When to use

Use when eggNOG-mapper/.emapper.annotations or similar KO annotations are suspected incomplete (e.g. fragmented MAG assemblies or truncated genes) and missing KOs from KEGG Modules need evidence-backed recovery via HMM profile search.

## When NOT to use

- The goal is only KEGG Module completeness summarization with no need to expand annotations — use KEMET with --skip_hmm instead of HMM searches
- The genome annotation comes from a format other than eggnog/kaas/kofamkoala, since KEMET's -a parameter only supports these KO annotation formats
- Genes are severely fragmented (e.g. ~70% sequence truncation), where recovery rates decline markedly and HMM hits may not be trustworthy, especially in MAGs

## Inputs

- Genome or MAG nucleotide FASTA file (with predicted genes / protein sequences)
- eggNOG-mapper v2 annotation file (.emapper.annotations) mapping ORFs to KEGG KOs
- KEGG GENES database downloaded/updated via the KEGG API (used by KEMET for Module definitions and HMM profiles)

## Outputs

- KEGG MODULES recap tables of metabolic potential/completeness per genome or MAG
- HMM-search hits listing recovered KOs (genes and annotations) passing the 0.43 threshold
- Per-removal outcome table of mock-removal true positives (e.g. ko_mock_removal_tpr.csv)

## How to apply

Annotate the genome or MAG with eggNOG-mapper v2 (default parameters) after gene calling with Prodigal v2.6.3, then run KEMET on the FASTA file with './kemet.py [FASTA_file] -a eggnog --hmm_mode onebm', which searches profile HMMs for KOs belonging to incomplete KEGG Modules against the unmodified gene sequences. Apply the default HMM quality filter (--threshold_value 0.43) to classify hits as legit recovered KOs; use --skip_gsmm to stop after HMM analysis when only annotation expansion is desired. To quantify recovery, remove three KO annotations per genome (mock removal) from the .emapper.annotations and score a removal as a true positive when KEMET recovers both the removed gene and annotation; reported recovery was 55% (20/36) for MAGs and 67% (8/12) for complete genomes without truncation, dropping to 20–33% when gene sequences were truncated by 30% of their original length. Optionally feed recovered KO evidence into GSMM gapfilling via --gsmm_mode to improve draft genome-scale metabolic models.

## Related tools

- **KEMET** (Runs the HMM profile search (--hmm_mode onebm), applies the 0.43 quality threshold, evaluates KEGG Module completeness, and optionally gapfills GSMMs with recovered KO evidence) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Provides the initial KO annotations (.emapper.annotations) whose missing/incomplete KOs the HMM search aims to recover)
- **Prodigal** (Performs gene calling (v2.6.3, default options) to generate the ORFs searched with profile HMMs)

## Examples

```
./kemet.py [FASTA_file] -a eggnog --hmm_mode onebm --skip_gsmm --threshold_value 0.43
```

## Evaluation signals

- Mock-removal benchmark: removing three KOs per genome, 20/36 and 8/12 removals (55% and 67% true positive rate) must be correctly recovered for MAGs and complete genomes without truncation
- Every called HMM hit must pass the default quality threshold (--threshold_value 0.43) to count as a legit recovered KO
- Truncation benchmark: 30% gene-sequence truncation should yield recovery rates of 20–33% (42% assuming unbiased gene prediction), with monotonic decline at 70% truncation, most clearly in MAGs
- Recovered annotations must increase KEGG Module completeness block coverage relative to the eggNOG-only annotation and, when gapfilled into GSMMs, should improve model quality toward complete-genome accuracy

## Limitations

- Recovery depends on sequence integrity: 30% gene truncation reduced recovery to 20–33%, and 70% truncation caused a further decline, most clearly in the MAG dataset, because truncation impairs gene prediction
- Recovery is not guaranteed even for full-length genes — roughly 45% of MAG and 33% of complete-genome mock removals were not recovered under the 0.43 threshold
- KEMET only supports KO annotation formats eggnog, kaas, and kofamkoala, and the KEGG GENES database must be downloaded/updated through the KEGG API

## Evidence

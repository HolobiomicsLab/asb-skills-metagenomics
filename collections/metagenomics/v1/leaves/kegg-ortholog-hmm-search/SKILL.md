---
name: kegg-ortholog-hmm-search
description: Use when a MAG or genome FASTA has incomplete KEGG Module blocks and
  unannotated genes that may be genuine KOs recoverable by profile HMM searches, especially
  for partial/truncated gene sequences.
when_to_use_negative:
- You only need KEGG Module completeness tables from existing annotation — use --skip_hmm
  instead of running HMM searches
- The genome already has complete, manually curated KO annotation and no missing orthologs
  are expected
- Working with poor-quality MAGs below the >=90% completeness / <=5% contamination
  screening used in the paper's validation
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0610
tools:
- name: KEMET
  role: Runs nhmmer profile HMM searches for missing KOs in onebm mode and filters
    hits by score-per-length threshold
  repo: https://github.com/Matteopaluh/KEMET
- name: Python
  role: Implementation language of the KEMET command-line toolbox
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Provides the baseline KEGG KO annotations (-a eggnog) whose gaps the HMM search
    expands
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:5a7a525cd46593af3270454791a4ae7153cdadc10fc5e1d2dfc8021c1b589c78
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
  - Python
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kegg-ortholog-hmm-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Searches for missing KEGG orthologs (KOs) in MAGs/genomes using nhmmer profile HMM searches in KEMET's --hmm_mode onebm, filtering candidate hits with a score-per-length threshold (default 0.43) to expand KEGG Module completeness evaluation beyond initial eggNOG/KAAS/KofamKoala annotation.

## When to use

Use when a MAG or genome FASTA has incomplete KEGG Module blocks and unannotated genes that may be genuine KOs recoverable by profile HMM searches, especially for partial/truncated gene sequences.

## When NOT to use

- You only need KEGG Module completeness tables from existing annotation — use --skip_hmm instead of running HMM searches
- The genome already has complete, manually curated KO annotation and no missing orthologs are expected
- Working with poor-quality MAGs below the >=90% completeness / <=5% contamination screening used in the paper's validation

## Inputs

- MAG/genome FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file from eggnog (.emapper.annotations), kaas, or kofamkoala (.ko)
- KEGG GENES database downloaded/updated via the KEGG API
- Optional BLASTp hits against NCBI nr for threshold validation

## Outputs

- Expanded KEGG KOs assigned to previously unannotated/partial genes
- Updated KEGG Module completeness tables reflecting recovered KOs
- HMM hit score-per-length values filterable via --threshold_value (default 0.43)

## How to apply

Run kemet.py on the FASTA of the MAG/genome with an existing KEGG annotation format (-a eggnog/kaas/kofamkoala) and --hmm_mode onebm so nhmmer searches a subset of KOs that would restore incomplete KEGG Module blocks. Keep the default score-per-length filter (--threshold_value 0.43) unless tuning against ground truth: the KEMET authors fine-tuned this threshold against manually checked translated BLASTp hits from NCBI nr on two MAG datasets, finding 0.43 identifies the most hits with correct KEGG ortholog descriptors and fewest false positives, retaining 4.6-7.5% of hits depending on the dataset. The tool tolerates partial gene sequences — validation included simulated 30% and 70% truncation of KO-annotated genes, with 55% (20/36) and 67% (8/12) of KO mock removals correctly recovered in MAGs and complete genomes. Skip GSMM gapfilling with --skip_gsmm if only annotation expansion is desired, and use --log/-v/-q for progress and HMMER error control.

## Related tools

- **KEMET** (Runs nhmmer profile HMM searches for missing KOs in onebm mode and filters hits by score-per-length threshold) — https://github.com/Matteopaluh/KEMET
- **Python** (Implementation language of the KEMET command-line toolbox) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Provides the baseline KEGG KO annotations (-a eggnog) whose gaps the HMM search expands)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43 --skip_gsmm --log
```

## Evaluation signals

- Tuned --threshold_value retains 4.6-7.5% of hits across the two MAG datasets when validated against BLASTp/NCBI nr ground truth
- KO mock-removal recovery rates within the reported ranges: 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes
- Recovered hits carry correct KEGG ortholog descriptors and few false positives relative to manually curated BLASTp checks
- KOs recovered after simulated 30% or 70% gene-sequence truncation still restore the intended KEGG Module block
- Recap tables show increased KEGG Module completeness after HMM expansion versus the eggNOG-only baseline

## Limitations

- Hit-retention fraction depends on the input dataset (4.6-7.5% across the two MAG datasets), so the default 0.43 threshold is not uniformly optimal for all inputs
- Only 55% of KO mock removals in MAGs (20/36) were correctly recovered, so HMM recovery is imperfect for MAGs
- Score-per-length filtering is the sole discriminator between legit HMM hits; it may exclude genuine orthologs or admit false positives

## Evidence

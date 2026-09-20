---
name: kegg-ortholog-annotation-removal
description: Use when you need to quantitatively validate an annotation-expansion
  tool (e.g., KEMET) by removing known KOs from eggNOG-mapper .emapper.annotations
  files and measuring recovery rates from FASTA sequences.
when_to_use_negative:
- Your goal is KEGG Module completeness evaluation only — use KEMET with --skip_hmm
  rather than running mock-removal HMM searches
- The annotation files were produced by tools other than the supported KEGG KO annotation
  formats (eggnog / kaas / kofamkoala) with their default file extensions
- You need to test recovery from truncated sequences but your input FASTA contains
  full-length genes only — the 30%/70% truncation variant requires explicit sequence
  deletion simulation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0196
tools:
- name: KEMET
  role: Performs the HMM-based search (--hmm_mode onebm) to recover mock-removed KOs,
    applying a default --threshold_value 0.43 quality filter
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Produces the .emapper.annotations KO annotations whose entries are mock-removed
    prior to the HMM recovery test
- name: Python
  role: Used to programmatically remove the three KO annotations per genome and to
    score recovered vs. removed KOs to compute true positive rates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:4cb7afbeebf9720bea156f1fbee74ff511612c5f192c3a3c2eba8101675c81fe
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# kegg-ortholog-annotation-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that mock-removes KEGG ortholog (KO) annotations from eggNOG-mapper results and tests whether KEMET's HMM search (--hmm_mode onebm) correctly recovers the removed genes and annotations, yielding true positive rates for annotation expansion.

## When to use

Use when you need to quantitatively validate an annotation-expansion tool (e.g., KEMET) by removing known KOs from eggNOG-mapper .emapper.annotations files and measuring recovery rates from FASTA sequences.

## When NOT to use

- Your goal is KEGG Module completeness evaluation only — use KEMET with --skip_hmm rather than running mock-removal HMM searches
- The annotation files were produced by tools other than the supported KEGG KO annotation formats (eggnog / kaas / kofamkoala) with their default file extensions
- You need to test recovery from truncated sequences but your input FASTA contains full-length genes only — the 30%/70% truncation variant requires explicit sequence deletion simulation

## Inputs

- eggNOG-mapper v2 annotation files (.emapper.annotations) for each genome/MAG
- Gene sequence FASTA files (gene calls from Prodigal v2.6.3)
- List of three KOs to mock-remove per genome

## Outputs

- Per-removal outcome records (removed KO, genome, recovered/not recovered)
- Aggregate true positive rates per genome category (e.g., ko_mock_removal_tpr.csv with 20/36 = 55% for MAGs and 8/12 = 67% for complete genomes)

## How to apply

Prepare eggNOG-mapper v2 annotation files (.emapper.annotations) for each genome, generated from gene calls made with Prodigal v2.6.3 under default options. Programmatically remove three KO annotations from each genome's input annotation set, then run KEMET with './kemet.py [FASTA_file] -a eggnog --hmm_mode onebm' so that profile HMMs search the removed KOs against the unmodified gene sequences (no truncation). Apply the default HMM quality filter (--threshold_value 0.43) to classify hits as legit recovered KOs. Score each mock removal as a true positive only when both the removed gene and its annotation are correctly recovered, then aggregate true positive rates. In the published benchmark on 5 complete genomes and 12 MAGs this yielded 20/36 (55%) and 8/12 (67%) recovery for MAGs and complete genomes, respectively. The variant of this test simulating deletion of 30% or 70% of the tested gene's sequence length assesses recovery from truncated sequences.

## Related tools

- **KEMET** (Performs the HMM-based search (--hmm_mode onebm) to recover mock-removed KOs, applying a default --threshold_value 0.43 quality filter) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Produces the .emapper.annotations KO annotations whose entries are mock-removed prior to the HMM recovery test)
- **Python** (Used to programmatically remove the three KO annotations per genome and to score recovered vs. removed KOs to compute true positive rates)

## Examples

```
./kemet.py genome.fasta -a eggnog --hmm_mode onebm --threshold_value 0.43
```

## Evaluation signals

- Recovery counts must equal the published 20/36 (55% TPR) for MAGs and 8/12 (67% TPR) for complete genomes in the untruncated test
- A removal is scored as a true positive only when both the gene and the annotation are correctly recovered by the HMM search
- All HMM hits must pass the default quality filter (--threshold_value 0.43) before being counted as legit recovered KOs
- Each genome must have exactly three KO annotations removed before running KEMET
- Per-removal outcome rows in the output CSV must correspond one-to-one with the mock removals (17 genomes x 3 removals = 51 total removals: 36 for MAGs, 12 for complete genomes subsets reported)

## Limitations

- Recovery is not guaranteed — only 55% (MAGs) and 67% (complete genomes) of mock-removed KOs were correctly recovered without truncation
- Truncation of the tested gene sequence by 30% or 70% of original length was simulated separately, so full-sequence results do not transfer to truncated cases
- The default --threshold_value 0.43 filter may discard borderline HMM hits, lowering measured recovery rates

## Evidence

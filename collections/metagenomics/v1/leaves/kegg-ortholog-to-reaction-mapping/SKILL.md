---
name: kegg-ortholog-to-reaction-mapping
description: Use when a MAG or isolate genome has KEGG KO annotations (eggNOG/KAAS/KofamKOALA
  format) and you need to convert them into metabolic reactions for GSMM reconstruction
  or evidence-based gapfilling, especially after HMM-based KO recovery.
when_to_use_negative:
- Input is already a reconstructed GSMM that needs no additional KO evidence — network
  gap-filling without genomic support risks erroneous predictions
- The goal is only KEGG Module completeness summarization without model reconstruction
  — use --skip_gsmm or a Module evaluation tool alone
- Annotations come from a format other than eggnog/kaas/kofamkoala, which KEMET's
  -a flag does not parse
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_2259
tools:
- name: KEMET
  role: Performs HMM-based KO recovery and maps recovered KOs to reactions for GSMM
    reconstruction or gapfilling via --gsmm_mode denovo/existing
  repo: https://github.com/Matteopaluh/KEMET
- name: CarveMe
  role: Reconstructs reference/draft GSMMs from complete genomes and MAGs (v1.4.1,
    --fbc2 -u options)
- name: gapseq
  role: Alternative tool for draft GSMM reconstruction from genomes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:faf2f98c3a5a370b0d4e4ea9c584493dfc15d53998e4e1678b5ed0737df1740d
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
  - CarveMe
  - KEMET
  - Python
  - gapseq
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# kegg-ortholog-to-reaction-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps KEGG ortholog (KO) annotations — including KOs recovered via HMM profile searches with KEMET — onto BiGG/ModelSEED reactions to gapfill or reconstruct genome-scale metabolic models (GSMMs/GEMs) from MAGs or genomes, improving metabolic phenotype prediction.

## When to use

Use when a MAG or isolate genome has KEGG KO annotations (eggNOG/KAAS/KofamKOALA format) and you need to convert them into metabolic reactions for GSMM reconstruction or evidence-based gapfilling, especially after HMM-based KO recovery.

## When NOT to use

- Input is already a reconstructed GSMM that needs no additional KO evidence — network gap-filling without genomic support risks erroneous predictions
- The goal is only KEGG Module completeness summarization without model reconstruction — use --skip_gsmm or a Module evaluation tool alone
- Annotations come from a format other than eggnog/kaas/kofamkoala, which KEMET's -a flag does not parse

## Inputs

- MAG or genome FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggNOG (.emapper.annotations), KAAS, or KofamKOALA (.ko) format
- Pre-existing GSMM draft (CarveMe or gapseq) when using --gsmm_mode existing

## Outputs

- KEGG MODULES completeness recap tables of metabolic potential
- HMM-recovered KO annotations for missing orthologs
- De-novo or gapfilled GSMM/GEM with recovered KOs mapped to BiGG/ModelSEED reactions

## How to apply

Run KEMET (kemet.py) with a FASTA or KEGG annotation file, specifying -a with the annotation format (eggnog/kaas/kofamkoala, preserving default extensions like .emapper.annotations) and --hmm_mode onebm to recover missing KOs via profile HMMs (default quality threshold 0.43, adjustable via --threshold_value). Use --gsmm_mode denovo to reconstruct a de-novo GSMM/GEM with recovered KOs mapped to BiGG/ModelSEED reactions, or --gsmm_mode existing to add HMM-recovered reactions to a pre-existing CarveMe/gapseq draft model. Optionally use --skip_hmm to obtain only KEGG MODULES completeness tables, or --skip_gsmm to stop after HMM analysis. Rationale: MAG-derived GSMMs suffer from gapped assemblies, and HMM-recovered KO-to-reaction mapping closes roughly 10% of the accuracy gap toward complete-genome models without unsupported network gap-fill reactions.

## Related tools

- **KEMET** (Performs HMM-based KO recovery and maps recovered KOs to reactions for GSMM reconstruction or gapfilling via --gsmm_mode denovo/existing) — https://github.com/Matteopaluh/KEMET
- **CarveMe** (Reconstructs reference/draft GSMMs from complete genomes and MAGs (v1.4.1, --fbc2 -u options))
- **gapseq** (Alternative tool for draft GSMM reconstruction from genomes)

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --gsmm_mode denovo
```

## Evaluation signals

- KO mock removal tests: removing three KO annotations per genome and checking recovery gave 55% (20/36) true positive rate for MAGs and 67% (8/12) for complete genomes with --hmm_mode onebm
- FVA at fixed maximal growth rate on MAG-only vs KEMET-expanded vs complete-genome models should show ~10% improvement of expanded models over MAG-only ones
- The Selenomonas ruminantium expansion should recover cellobiose and related sugar exchange reactions absent from the MAG-only model
- Recovered-KO models should approach the accuracy of complete-genome models in predicting wet-lab metabolite production/consumption
- Recovered KOs must appear as mapped reactions in the output GSMM (denovo rebuild or additions to existing model), not merely as HMM hit lists

## Limitations

- MAG-based GSMMs are prone to reconstruction errors due to the gapped nature of metagenomic assemblies
- Network gap-fill reactions not supported by genomic evidence can produce erroneous predictions; KEMET's HMM evidence mitigates but does not eliminate this
- KEMET can identify unannotated genes with partial sequence truncations, but recovery of very short truncations (70% deletion simulated) reduces success rates
- HMM hit quality default threshold is 0.43; changing --threshold_value trades sensitivity for specificity
- No discussion/limitations section was extractable from the article text (placeholder only)

## Evidence

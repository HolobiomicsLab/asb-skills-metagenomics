---
name: otu-copy-number-normalization
description: Use when a 16S rRNA OTU table must be converted to per-organism abundances
  (correcting 16S copy-number variation) before predicting gene family (KO) abundances
  with PICRUSt.
when_to_use_negative:
- Input is shotgun metagenome data annotated directly to KOs (e.g. HUMAnN-annotated
  WGS reads) — no 16S copy-number correction applies.
- The OTU table is not PICRUSt-compatible (not closed-reference picked against Greengenes+IMG),
  since copy-number predictions are keyed to that reference tree's tips.
- Only taxonomic (not functional) profiling of the community is needed — normalization
  is a preparatory step for metagenome prediction, not an end in itself.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0621
tools:
- name: PICRUSt
  role: Performs the OTU table normalization by 16S copy number predictions prior
    to metagenome inference
  repo: http://picrust.github.com
- name: QIIME 1.5.0-dev
  role: Constructs the PICRUSt-compatible OTU table via closed-reference OTU picking
    (pick_reference_otus_through_otu_table.py)
- name: uclust
  role: OTU picking search engine used against Greengenes+IMG when building the input
    table
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:69465f602a95c586cc8aca6067e49af14b1244bbcecaeb82a00ea77694306206
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Greengenes
  - HUMAnN
  - PICRUSt
  - QIIME 1.5.0-dev
  - STAMP
  - uclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# otu-copy-number-normalization

## Summary

Normalize a PICRUSt-compatible OTU table (BIOM format, closed-reference picked against Greengenes+IMG) by predicted 16S rRNA copy number so OTU abundances represent estimated organism counts rather than marker gene counts, as the first step of PICRUSt metagenome prediction (Langille et al. 2013, Nature Biotechnology).

## When to use

Use when a 16S rRNA OTU table must be converted to per-organism abundances (correcting 16S copy-number variation) before predicting gene family (KO) abundances with PICRUSt.

## When NOT to use

- Input is shotgun metagenome data annotated directly to KOs (e.g. HUMAnN-annotated WGS reads) — no 16S copy-number correction applies.
- The OTU table is not PICRUSt-compatible (not closed-reference picked against Greengenes+IMG), since copy-number predictions are keyed to that reference tree's tips.
- Only taxonomic (not functional) profiling of the community is needed — normalization is a preparatory step for metagenome prediction, not an end in itself.

## Inputs

- PICRUSt-compatible OTU table in BIOM format (closed-reference picked against Greengenes+IMG with uclust via QIIME 1.5.0-dev)
- Predicted 16S copy number per OTU from the PICRUSt pre-calculated reference files

## Outputs

- 16S copy-number-normalized OTU table (estimated organism counts per sample)
- Normalized OTU table feeding into predicted_metagenome.biom (predicted KO abundance table with per-sample NSTI values)

## How to apply

Load the PICRUSt-compatible OTU table in BIOM format, constructed via closed-reference OTU picking (QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py with uclust) against Greengenes+IMG. Divide each OTU's abundance by the predicted 16S copy number for that OTU, so abundances represent estimated organism counts rather than marker gene counts; this corrects for copy-number variation, one of the quantitative effects PICRUSt explicitly models. The normalized table is then fed directly into the metagenome inference step, where each normalized OTU abundance is multiplied by per-taxon KO abundances and summed per sample. Do not skip this step before metagenome prediction — unnormalized marker gene counts bias gene family abundances toward taxa with high 16S copy numbers.

## Related tools

- **PICRUSt** (Performs the OTU table normalization by 16S copy number predictions prior to metagenome inference) — http://picrust.github.com
- **QIIME 1.5.0-dev** (Constructs the PICRUSt-compatible OTU table via closed-reference OTU picking (pick_reference_otus_through_otu_table.py))
- **uclust** (OTU picking search engine used against Greengenes+IMG when building the input table)

## Evaluation signals

- Downstream metagenome prediction (normalized abundances × per-taxon KO abundances summed per sample) reproduces reported results, e.g. HMP Spearman r≈0.82 across 6,885 KOs and r≈0.9 for synthetic mock communities.
- Per-sample weighted NSTI values match published values for known datasets (e.g. 0.12 ± 0.02 s.d. for the 335 coral mucus samples, Study ID 1854).
- Output table is in BIOM format and can be carried directly into predict_metagenome-style inference to produce predicted_metagenome.biom.
- Normalized values are interpretable as estimated organism counts, i.e. each OTU count divided by its predicted 16S copy number (integer-normalized, no longer raw marker gene counts).

## Limitations

- 16S copy-number variation and lateral gene transfer quantitatively affect inference of microbial function from 16S data; some gain/loss or laterally transferred genes will be poorly predicted (broad confidence intervals).
- Accuracy depends on availability of related reference genomes — track NSTI to gauge expected prediction quality; phylogenetic gaps in the reference database degrade predictions.
- Only bacterial and archaeal 16S marker sequences are included; viral or eukaryotic metagenome components are not inferred.
- Strain-level variation cannot be distinguished if the marker gene sequence is identical among strains.

## Evidence

- [intro] Copy-number variation is a modeled quantitative effect: [evidence span withheld — anchor status: not_found]

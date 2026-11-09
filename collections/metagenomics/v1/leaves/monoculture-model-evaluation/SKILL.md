---
name: monoculture-model-evaluation
description: Use when validating a Zebra genome cover recurrence fit with a known-composition
  control (e.g., a monoculture SAM alignment against the WoL database), to confirm
  predicted cover tracks observed cover and that true hits separate from low-coverage
  reference-overlap artifacts.
when_to_use_negative:
- Input is a pre-computed feature/BIOM abundance table rather than .sam alignments
  — calculate_coverages.py requires .sam files (bowtie2 or SHOGUN bowtie2 option)
  and only works with the Web of Life database
- No ground-truth control sample (e.g., monoculture or mock community) is available,
  since validation relies on knowing the true positive organism
- The goal is to filter production alignments — use the Zebra static saturated genome
  cover threshold / filter_sam.py workflow instead of the monoculture model evaluation
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Zebra
  role: Provides the recurrence to estimate genome cover and confirm saturation, used
    to dynamically predict genome cover as a function of read count
  repo: https://github.com/biocore/zebra_filter
- name: calculate_coverages.py
  role: Calculates the aggregated unique genome cover of each target genome from the
    input .sam alignments, producing the coverage statistics used to fit the recurrence
  repo: https://github.com/biocore/zebra_filter
- name: bowtie2
  role: Generates the .sam alignments of the S. aureus monoculture against the Web
    of Life reference database
- name: Python
  role: Runtime for calculate_coverages.py and for fitting the recurrence and writing
    the predicted-versus-observed comparison table
  repo: https://github.com/biocore/zebra_filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00758-22
    title: 'Zebra: Static and Dynamic Genome Cover Thresholds with Overlapping References'
  source_content_hash: sha256:bc51e9c82af2b2132e7d1b37011ab0ebf53f1ca79f3fea5167ae7773540807fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/zebra_filter
  tools:
  - Python
  - Zebra
  - bowtie2
  - calculate_coverages.py
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# monoculture-model-evaluation

## Summary

Validation procedure for the Zebra dynamic genome cover prediction model: fit the recurrence relating genome cover to read count, then evaluate predicted versus observed cover in a Staphylococcus aureus monoculture to confirm the model separates true hits (S. aureus and genuine contaminants) from false artifacts of reference overlap.

## When to use

Use when validating a Zebra genome cover recurrence fit with a known-composition control (e.g., a monoculture SAM alignment against the WoL database), to confirm predicted cover tracks observed cover and that true hits separate from low-coverage reference-overlap artifacts.

## When NOT to use

- Input is a pre-computed feature/BIOM abundance table rather than .sam alignments — calculate_coverages.py requires .sam files (bowtie2 or SHOGUN bowtie2 option) and only works with the Web of Life database
- No ground-truth control sample (e.g., monoculture or mock community) is available, since validation relies on knowing the true positive organism
- The goal is to filter production alignments — use the Zebra static saturated genome cover threshold / filter_sam.py workflow instead of the monoculture model evaluation

## Inputs

- S. aureus monoculture .sam alignment file(s) against the Web of Life (WoL) reference database
- WoL genome metadata file (databases/WoL/metadata.tsv) with genome lengths
- Aggregated coverage statistics output from calculate_coverages.py (gotu, covered_length, total_length, coverage_ratio, strain)

## Outputs

- s_aureus_genome_cover_fit.tsv with predicted and observed genome cover tabulated side by side for the monoculture hit and low-coverage spurious hits
- Fitted Zebra recurrence parameters for genome cover as a function of read count
- Verdict on whether S. aureus/true contaminants separate from reference-overlap artifacts

## How to apply

Generate a .sam alignment of the S. aureus monoculture reads against the Web of Life (WoL) reference database using bowtie2. Compute unique genome cover per reference genome from the alignments with calculate_coverages.py from the Zebra_filter repository, supplying databases/WoL/metadata.tsv to resolve genome lengths (coverage_ratio = covered_length / total_length, counting only unique coverage so overlapping reads do not inflate cover). Fit the Zebra recurrence that dynamically predicts genome cover as a function of read count using the aggregated coverage statistics. Tabulate predicted versus observed genome cover side by side for the S. aureus monoculture hit and for low-coverage spurious hits, writing the comparison to s_aureus_genome_cover_fit.tsv. Accept the fit if predicted cover tracks observed cover for the monoculture sample and the S. aureus/true-contaminant group cleanly separates from reference-overlap artifacts in the predicted-versus-observed comparison, per the article's reported finding.

## Related tools

- **Zebra** (Provides the recurrence to estimate genome cover and confirm saturation, used to dynamically predict genome cover as a function of read count) — https://github.com/biocore/zebra_filter
- **calculate_coverages.py** (Calculates the aggregated unique genome cover of each target genome from the input .sam alignments, producing the coverage statistics used to fit the recurrence) — https://github.com/biocore/zebra_filter
- **bowtie2** (Generates the .sam alignments of the S. aureus monoculture against the Web of Life reference database)
- **Python** (Runtime for calculate_coverages.py and for fitting the recurrence and writing the predicted-versus-observed comparison table) — https://github.com/biocore/zebra_filter

## Examples

```
python calculate_coverages.py -i s_aureus_alignments/ -o s_aureus_output.tsv -d databases/WoL/metadata.tsv
```

## Evaluation signals

- Predicted genome cover from the fitted recurrence tracks observed genome cover for the S. aureus monoculture sample
- In the predicted-versus-observed comparison, S. aureus and true contaminants group separately from false artifacts of reference overlap
- s_aureus_genome_cover_fit.tsv contains predicted and observed values side by side with the genome id, covered_length, total_length, and coverage_ratio columns from calculate_coverages.py output
- coverage_ratio values fall in (0, 1] since covered_length cannot exceed total_length of the reference genome
- Low-coverage spurious hits remain below the cover region occupied by the monoculture true positive

## Limitations

- calculate_coverages.py currently only works with the Web of Life (WoL) database and expects .sam alignments from bowtie2 or SHOGUN (bowtie2 alignment option)
- Ambiguous reads overlapping multiple reference genomes create artifacts, and this problem grows as the WoL reference database expands
- Only unique coverage counts — multiple reads covering the same genomic location do not increase coverage, so shallow per-sample sequencing yields low cover regardless of presence; saturation is assessed by compositing across like samples
- It is a threshold on saturated genome cover, not genome cover itself, which indicates a spurious reference hit or distant relative

## Evidence

- [abstract] Dynamic genome cover prediction examined in monoculture: "We show how to dynamically predict genome cover by read count and examine our model in Staphylococcus aureus monoculture."
- [abstract] Model separates true hits from artifacts: "Our modeling cleanly separates both S. aureus and true contaminants from the false artifacts of reference overlap."
- [methods] Coverage calculation from SAM alignments: "the first, _calculate_coverages.py_, calculates the genome coverage from .sam alignments"
- [intro] Unique coverage counting rule: "only unique coverage is counted, multiple reads covering the same location of the genome will not increase the coverage"
- [readme] Coverage ratio definition and output columns: "coverage_ratio: Portion of hit genome covered. Calculated by covered_length / total_length"
- [readme] Restriction to WoL database: "Currently only works with the Web of Life database (https://biocore.github.io/wol/). The Web of Life metadata file is also required"

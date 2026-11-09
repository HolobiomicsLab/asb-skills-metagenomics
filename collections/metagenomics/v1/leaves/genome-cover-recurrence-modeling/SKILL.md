---
name: genome-cover-recurrence-modeling
description: Use when sam alignments against the WoL reference database contain low-coverage
  hits that may be reference-overlap artifacts, and you need a model-predicted versus
  observed genome cover comparison to judge whether hits are true contaminants or
  spurious.
when_to_use_negative:
- Input is already a filtered abundance table or biom table with no .sam alignments
  — the recurrence must be fit from per-genome coverage of raw alignments
- Alignments were generated against a database other than the Web of Life — calculate_coverages.py
  currently only works with the WoL database
- Only a single shallow sample is available with no like samples to aggregate, since
  genome cover may not saturate without 'artificially deep' composited sequencing
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Zebra
  role: Provides the recurrence to estimate genome cover, confirm saturation, and
    threshold the genome cover metric across like samples
  repo: https://github.com/biocore/zebra_filter
- name: calculate_coverages.py
  role: Computes aggregated unique genome cover per reference genome from .sam alignments,
    using the WoL metadata file for genome lengths
  repo: https://github.com/biocore/zebra_filter
- name: bowtie2
  role: Generates the .sam alignments against the WoL reference database that feed
    the coverage calculation
- name: Woltka
  role: 'Optional downstream integration: regenerate abundance tables from filtered
    alignments'
  repo: https://github.com/qiyunzhu/woltka
- name: Python
  role: Runtime for calculate_coverages.py and fitting/tabulating the recurrence comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00758-22
    title: 'Zebra: Static and Dynamic Genome Cover Thresholds with Overlapping References'
  source_content_hash: sha256:5b170d11ae18c50f83561ddb6d9b309cc20d93b616baa4cd1d183e9bfa2ae997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/zebra_filter
  tools:
  - Python
  - Woltka
  - Zebra
  - bowtie2
  - calculate_coverages.py
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genome-cover-recurrence-modeling

## Summary

Dynamically predict genome cover as a function of read count using the Zebra recurrence, evaluated on a Staphylococcus aureus monoculture, to separate true genomic hits from false reference-overlap artifacts produced by ambiguous reads mapping to overlapping reference genomes.

## When to use

Use when .sam alignments against the WoL reference database contain low-coverage hits that may be reference-overlap artifacts, and you need a model-predicted versus observed genome cover comparison to judge whether hits are true contaminants or spurious.

## When NOT to use

- Input is already a filtered abundance table or biom table with no .sam alignments — the recurrence must be fit from per-genome coverage of raw alignments
- Alignments were generated against a database other than the Web of Life — calculate_coverages.py currently only works with the WoL database
- Only a single shallow sample is available with no like samples to aggregate, since genome cover may not saturate without 'artificially deep' composited sequencing

## Inputs

- .sam alignment files against the Web of Life (WoL) reference database (e.g., produced by bowtie2)
- WoL genome metadata file (databases/WoL/metadata.tsv) providing genome lengths
- Aggregated per-genome coverage statistics (gotu, covered_length, total_length, coverage_ratio, strain) from calculate_coverages.py

## Outputs

- s_aureus_genome_cover_fit.tsv — side-by-side table of predicted versus observed genome cover for the true hit and low-coverage spurious hits
- Fitted Zebra recurrence parameters predicting genome cover as a function of read count

## How to apply

Align reads (e.g., an S. aureus monoculture) against the Web of Life (WoL) reference database with bowtie2, then run calculate_coverages.py from the Zebra_filter repository, supplying the WoL metadata file (databases/WoL/metadata.tsv) to resolve genome lengths; only unique coverage is counted, so multiple reads covering the same genomic location do not increase coverage. Aggregate the coverage statistics and fit the Zebra recurrence that dynamically predicts genome cover as a function of read count, which estimates genome cover and confirms saturation. Compare model-predicted versus observed genome cover for the monoculture's true hit and for low-coverage spurious hits, tabulating predicted and observed values side by side (written to s_aureus_genome_cover_fit.tsv). The rationale is that reads ambiguously overlapping multiple reference genomes create false artifacts whose cover stays low as read count grows, whereas genuine hits track the predicted saturation curve.

## Related tools

- **Zebra** (Provides the recurrence to estimate genome cover, confirm saturation, and threshold the genome cover metric across like samples) — https://github.com/biocore/zebra_filter
- **calculate_coverages.py** (Computes aggregated unique genome cover per reference genome from .sam alignments, using the WoL metadata file for genome lengths) — https://github.com/biocore/zebra_filter
- **bowtie2** (Generates the .sam alignments against the WoL reference database that feed the coverage calculation)
- **Woltka** (Optional downstream integration: regenerate abundance tables from filtered alignments) — https://github.com/qiyunzhu/woltka
- **Python** (Runtime for calculate_coverages.py and fitting/tabulating the recurrence comparison)

## Examples

```
python calculate_coverages.py -i test_data/ -o test_data/sample_output.tsv -d databases/WoL/metadata.tsv
```

## Evaluation signals

- The fitted recurrence's predicted genome cover tracks observed genome cover for the S. aureus monoculture hit
- The predicted-versus-observed comparison cleanly separates S. aureus and true contaminants from false artifacts of reference overlap
- Low-coverage spurious hits (reference-overlap artifacts) show observed cover far below the model prediction at their read count
- Coverage statistics contain valid WoL genome IDs with total_length matching databases/WoL/metadata.tsv and coverage_ratio = covered_length / total_length
- Output table s_aureus_genome_cover_fit.tsv contains paired predicted and observed values for each evaluated hit

## Limitations

- Ambiguous reads overlapping multiple reference genomes create artifacts, and this problem grows with the WoL reference database (10,575 genomes and growing)
- calculate_coverages.py currently only works with the Web of Life database
- It is a threshold on saturated genome cover, not genome cover itself, which indicates a spurious reference hit or distant relative — raw cover values alone are insufficient
- No discussion section content was available, so no additional limitations or reproducibility gaps were extracted from the paper

## Evidence

- [abstract] Dynamic prediction of genome cover evaluated in S. aureus monoculture: "We show how to dynamically predict genome cover by read count and examine our model in Staphylococcus aureus monoculture."
- [abstract] Model cleanly separates true hits from artifacts: "Our modeling cleanly separates both S. aureus and true contaminants from the false artifacts of reference overlap."
- [abstract] Zebra recurrence purpose: "We present Zebra, a method to compute and threshold the genome cover metric across like samples, a recurrence to estimate genome cover and confirm saturation"
- [intro] Unique coverage counting rule: "only unique coverage is counted, multiple reads covering the same location of the genome will not increase the coverage"
- [methods] calculate_coverages.py computes coverage from .sam alignments: "the first, _calculate_coverages.py_, calculates the genome coverage from .sam alignments"
- [readme] WoL metadata required for genome lengths: "The Web of Life metadata file is also required and provided in this repository at _databases/WoL/metadata.tsv_."
- [readme] Coverage ratio definition in output tsv: "coverage_ratio: Portion of hit genome covered. Calculated by covered_length / total_length"
- [abstract] Ambiguity artifacts grow with WoL: "due largely to ambiguity of reads which overlap multiple reference genomes. With the Web of Life (WoL) reference database hosting 10,575 reference genomes and growing, the percentage of ambiguous"

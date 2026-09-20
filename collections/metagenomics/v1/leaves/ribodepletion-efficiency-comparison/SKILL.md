---
name: ribodepletion-efficiency-comparison
description: Use when a pair of metatranscriptome samples (ribodepleted vs. non-ribodepleted,
  e.g. MG-RAST samples 4015 vs. 4012) must be compared to assess whether rRNA depletion
  is taxonomically unbiased, or when residual rRNA reads might distort organism abundance
  estimates.
when_to_use_negative:
- Input is a single ribodepleted sample with no non-ribodepleted control — no per-phylum
  depletion percentages can be computed
- You need species- or strain-level depletion estimates rather than phylum-level aggregation
  (taxonomy_shifter.py can shift levels, but the validated comparison here is at phylum)
- The data are shotgun metagenomes, not metatranscriptomes — rRNA depletion bias does
  not apply because there are no rRNA transcript reads to quantify
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SAMSA
  role: pipeline providing RefSeq_output_reducer.py and taxonomy_shifter.py for aggregating
    and shifting MG-RAST organism annotations to phylum level
  repo: github.com/transcript/SAMSA
- name: MG-RAST
  role: annotation server producing the per-read organism (RefSeq) annotations that
    feed the rRNA-per-phylum tabulation
- name: RefSeq_output_reducer.py
  role: consolidates identical organism matches in summary files while retaining ribosomal
    (rRNA) reads
  repo: github.com/transcript/SAMSA
- name: taxonomy_shifter.py
  role: converts genus-level organism summary files to phylum level using Bacteria_Genus_flattened.tsv
  repo: github.com/transcript/SAMSA
- name: R
  role: RStudio-based analysis (R_organism_script.Rmd-style) to confirm depletion
    patterns and flag under-depleted phyla
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:97f1c26dd8bb2037f9019141a43cc6bacba5c3bd9aff182cce1e494ad468c617
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MG-RAST
  - R
  - RefSeq_output_reducer.py
  - SAMSA
  - taxonomy_shifter.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ribodepletion-efficiency-comparison

## Summary

Quantifies whether ribosomal RNA depletion removes rRNA uniformly across bacterial taxa by tabulating rRNA reads per phylum in ribodepleted versus non-ribodepleted metatranscriptomes and computing per-phylum depletion percentages. Surfaces taxa whose ribosomes resist depletion, which would otherwise skew perceived organism abundances.

## When to use

Use when a pair of metatranscriptome samples (ribodepleted vs. non-ribodepleted, e.g. MG-RAST samples 4015 vs. 4012) must be compared to assess whether rRNA depletion is taxonomically unbiased, or when residual rRNA reads might distort organism abundance estimates.

## When NOT to use

- Input is a single ribodepleted sample with no non-ribodepleted control — no per-phylum depletion percentages can be computed
- You need species- or strain-level depletion estimates rather than phylum-level aggregation (taxonomy_shifter.py can shift levels, but the validated comparison here is at phylum)
- The data are shotgun metagenomes, not metatranscriptomes — rRNA depletion bias does not apply because there are no rRNA transcript reads to quantify

## Inputs

- MG-RAST annotation results files for a ribodepleted sample and a matched non-ribodepleted sample (e.g. stool metatranscriptome samples 4012 and 4015)
- SAMSA organism summary files retaining both rRNA and mRNA read counts (produced by RefSeq_output_reducer.py)
- Bacteria_Genus_flattened.tsv taxonomy flattening database for taxonomy_shifter.py

## Outputs

- Phylum-level rRNA count table for each sample (e.g. table1_rrna_depletion_comparison.csv)
- Per-phylum depletion percentages (ribodepleted relative to non-ribodepleted) and overall rRNA reduction percentage (expected 63–82% for Ribo-Zero Gold)
- Flagged list of phyla showing little or no rRNA reduction (reference: Actinobacteria, Cyanobacteria, Spirochaetes)

## How to apply

Import MG-RAST annotation results for both samples and run SAMSA's RefSeq_output_reducer.py to consolidate identical organism matches into organism summary files, retaining rRNA reads alongside mRNA reads. Convert genus-level summaries to phylum level with taxonomy_shifter.py using the supplied Bacteria_Genus_flattened.tsv reference database. Tabulate rRNA reads per bacterial phylum for each sample and compute per-phylum depletion percentages (ribodepleted relative to non-ribodepleted) plus the overall rRNA reduction; in the reference study the kit (Ribo-Zero Gold) achieved a 63–82% overall reduction. Import the phylum-level tables into R (RStudio, R_organism_script.Rmd-style analysis with DESeq2) to confirm organism-level depletion patterns and flag phyla showing little or no reduction (in the reference: Actinobacteria, Cyanobacteria, Spirochaetes). The rationale is that because depletion does not equally deplete all species, remaining rRNA sequences should be identified per taxon and, if biased, discarded before downstream organism-abundance analysis.

## Related tools

- **SAMSA** (pipeline providing RefSeq_output_reducer.py and taxonomy_shifter.py for aggregating and shifting MG-RAST organism annotations to phylum level) — github.com/transcript/SAMSA
- **MG-RAST** (annotation server producing the per-read organism (RefSeq) annotations that feed the rRNA-per-phylum tabulation)
- **RefSeq_output_reducer.py** (consolidates identical organism matches in summary files while retaining ribosomal (rRNA) reads) — github.com/transcript/SAMSA
- **taxonomy_shifter.py** (converts genus-level organism summary files to phylum level using Bacteria_Genus_flattened.tsv) — github.com/transcript/SAMSA
- **R** (RStudio-based analysis (R_organism_script.Rmd-style) to confirm depletion patterns and flag under-depleted phyla)

## Examples

```
python taxonomy_shifter.py -usage  # then run with organism summary files and Bacteria_Genus_flattened.tsv to shift genus -> phylum
```

## Evaluation signals

- Reproduced overall rRNA reduction falls within the reported 63–82% range
- Output table identifies Actinobacteria, Cyanobacteria, and Spirochaetes as phyla with little or no depletion, matching Table 1 of the paper
- Per-phylum depletion percentages computed as ribodepleted sample (4015) counts relative to non-ribodepleted sample (4012) counts are consistent with the raw per-phylum rRNA counts
- Organism summary files after RefSeq_output_reducer.py retain both rRNA and mRNA reads (no accidental loss of ribosomal category)
- taxonomy_shifter.py output rows map cleanly onto the Bacteria_Genus_flattened.tsv reference database with no unmatched genera

## Limitations

- Only one commercially available ribodepletion kit (Ribo-Zero Gold) was tested; the authors believe all hybridization-probe-based methods will carry some similar bias
- Ribosomes that resist depletion (e.g. Actinobacteria, Cyanobacteria, Spirochaetes) inflate those organisms' apparent abundance if residual rRNA reads are not discarded
- MG-RAST annotation servers require queue waits of days to weeks, delaying sample turnaround
- The comparison data set had insufficient mRNA reads for a direct mRNA/rRNA correlation

## Evidence

- [abstract] Ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded: "ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded"
- [methods] taxonomy_shifter.py converts genus-level summaries to other taxonomic levels: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [methods] RefSeq_output_reducer.py consolidates identical organism matches: "RefSeq_output_reducer.py - consolidates identical organism matches in summary files"
- [discussion] Recommendation to discard remaining ribosomal reads before further analysis: "Therefore, we recommend discarding all remaining ribosomal reads before performing further analysis on these samples."
- [readme] Bacteria_Genus_flattened.tsv is the reference database for taxonomy_shifter.py: "Bacteria_Genus_flattened.tsv - reference database for taxonomy_shifter.py"

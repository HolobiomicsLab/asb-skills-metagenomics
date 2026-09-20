---
name: clustering-result-count-validation
description: Use when a spectral networking/clustering step has produced a spectrum-to-cluster
  assignment and you must verify the number of distinct clusters (e.g. 8,071 families
  from ~10M spectra) before downstream interpretation.
when_to_use_negative:
- The goal is to validate peptide-spectrum match counts (e.g. the 38 RiPPs at 1% FDR)
  rather than the number of spectral families from the networking step
- Input is a raw mzML/MS2 file collection that has not yet been clustered by GNPS
  spectral networks — no family assignments exist to count
- The spectra come from datasets other than the eight analyzed genomic/metagenomic
  GNPS datasets, so the 8,071-family reference count does not apply
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Runs the molecular networking / spectral networks step that computes pairwise
    spectral similarity and clusters the ~10 million tandem spectra into 8,071 spectral
    families; also the platform hosting the exported spectrum-to-family TSV.
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/metaminer/
- name: MetaMiner
  role: The peptidogenomics pipeline whose spectral-networking stage produces the
    family assignments being validated; the enlarged RiPP identifications via spectral
    networks are the biological context for the family count.
  repo: https://github.com/mohimanilab/MetaMiner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:b2a948f66cc195326016d9e93995248a9c56217520cd06f89beef7a76b67fe4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS
  - MetaMiner
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# clustering-result-count-validation

## Summary

A validation skill for spectral-networking outputs: count the distinct clusters (spectral families) produced from a large tandem MS/MS dataset and check the number against a published reference value, e.g. MetaMiner's ~10 million GNPS spectra collapsing into only 8,071 families.

## When to use

Use when a spectral networking/clustering step has produced a spectrum-to-cluster assignment and you must verify the number of distinct clusters (e.g. 8,071 families from ~10M spectra) before downstream interpretation.

## When NOT to use

- The goal is to validate peptide-spectrum match counts (e.g. the 38 RiPPs at 1% FDR) rather than the number of spectral families from the networking step
- Input is a raw mzML/MS2 file collection that has not yet been clustered by GNPS spectral networks — no family assignments exist to count
- The spectra come from datasets other than the eight analyzed genomic/metagenomic GNPS datasets, so the 8,071-family reference count does not apply

## Inputs

- Tandem mass spectra (~10 million MS/MS spectra) retrieved from eight GNPS molecular networking datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, plus the remaining two analyzed datasets)
- Pairwise spectral similarity network computed by GNPS molecular networking / spectral networks
- Spectrum-to-family assignment table in TSV format (spectral_families.tsv) exported from the clustering step

## Outputs

- Number of distinct spectral families (expected: 8,071) derived from the family-assignment table
- Validation pass/fail result confirming the distinct-family count matches the reported reference value

## How to apply

Run the clustering step first — in the MetaMiner context, GNPS molecular networking / spectral networks (Bandeira et al., 2007; Watrous et al., 2012) computes pairwise spectral similarity between the ~10 million tandem MS/MS spectra from the eight GNPS datasets (e.g. MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102) and groups spectra with related fragmentation patterns into families. Export the clustering result as a spectrum-to-family assignment table in TSV format (spectral_families.tsv). Then validate by extracting the set of distinct family identifiers from the TSV and confirming the cardinality equals the reported value (8,071 distinct families). The rationale is that the raw spectrum count massively overstates effective chemical diversity: the 10 million spectra collapse into only 8,071 families, which contextualizes the 38 RiPP identifications reported at 1% FDR. A large discrepancy between your family count and the reference indicates a misconfigured similarity threshold, truncated input, or duplicated spectra in the export.

## Related tools

- **GNPS** (Runs the molecular networking / spectral networks step that computes pairwise spectral similarity and clusters the ~10 million tandem spectra into 8,071 spectral families; also the platform hosting the exported spectrum-to-family TSV.) — https://ccms-ucsd.github.io/GNPSDocumentation/metaminer/
- **MetaMiner** (The peptidogenomics pipeline whose spectral-networking stage produces the family assignments being validated; the enlarged RiPP identifications via spectral networks are the biological context for the family count.) — https://github.com/mohimanilab/MetaMiner

## Examples

```
awk -F'\t' 'NR>1{print $2}' spectral_families.tsv | sort -u | wc -l  # expect 8071
```

## Evaluation signals

- The number of distinct family identifiers in spectral_families.tsv equals 8,071
- Every input tandem spectrum is assigned to exactly one family (no unassigned or duplicated spectrum rows in the TSV)
- The total number of spectra rows in the assignment table is on the order of 10 million, matching the analyzed input
- The family count is orders of magnitude smaller than the spectrum count, confirming the reported effective-chemical-diversity result
- All eight GNPS datasets (MSV000079506, MSV000079622, MSV000078839, MSV000078604, MSV000078552, MSV000080102, and the remaining two) are represented in the clustering input

## Limitations

- The 8,071-family reference count is specific to these eight genomic/metagenomic GNPS datasets and does not generalize to other spectral collections
- Most of the 8,071 families do not correspond to RiPPs: searching families against known structures with Dereplicator+ showed many belong to non-ribosomal peptides, and only 38 RiPPs were identified at 1% FDR
- 99% of spectra in GNPS lack paired genomics/metagenomics data and cannot be searched, so spectral families alone cannot be assigned a biosynthetic origin

## Evidence

- [discussion] spectral networking shows 10M spectra → 8071 families: "our spectral networking analysis shows that these 10 million spectra cluster into only 8071 families."
- [discussion] spectral networks enlarge RiPP identifications: "The set of RiPP identifications is enlarged via spectral networks (Bandeira et al., 2007; Watrous et al., 2012)."
- [discussion] families belong to non-RiPP classes: "searching these families against known chemical structures using Dereplicator+ shows that many of these families belong to non-ribosomal peptides"
- [readme] MetaMiner available as GNPS workflow: [evidence span withheld — anchor status: not_found]

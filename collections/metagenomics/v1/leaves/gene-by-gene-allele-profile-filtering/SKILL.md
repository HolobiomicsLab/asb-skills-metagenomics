---
name: gene-by-gene-allele-profile-filtering
description: Use when you have a chewBBACA AlleleCall results_alleles.tsv profile
  matrix over a strain collection and need to define the core-genome (cgMLST) locus
  set at a presence threshold such as 95%.
when_to_use_negative:
- Input is raw assemblies or reads and no allele calling has been performed yet —
  run chewBBACA CreateSchema and AlleleCall first to obtain the profile matrix.
- Goal is the full wgMLST schema rather than the core genome — the wgMLST locus set
  should not be thresholded at 95% presence.
- Loci are used for strain-level wgMLST surveillance where accessory-genome loci matter;
  thresholding to 95% would discard them.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: chewBBACA
  role: Provides the RemoveGenes, TestGenomeQuality, and ExtractCgMLST modules used
    to filter paralogous loci and select core-genome loci from the allelic profile
    matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: Python 3.4+
  role: Runtime environment for chewBBACA, including the profile-filtering modules
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:00cf64e611451e627c3d2c9f945d4af55d883b3bc49c5d4e4c80ff0e38b2a652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - Python
  - Python 3.4+
  - chewBBACA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-by-gene-allele-profile-filtering

## Summary

Derives a cgMLST schema from a wgMLST allelic profile matrix by removing paralogous loci and retaining only loci with valid allele calls in a chosen fraction (typically 95-99%) of analysed strains, using chewBBACA's RemoveGenes, TestGenomeQuality, and ExtractCgMLST modules.

## When to use

Use when you have a chewBBACA AlleleCall results_alleles.tsv profile matrix over a strain collection and need to define the core-genome (cgMLST) locus set at a presence threshold such as 95%.

## When NOT to use

- Input is raw assemblies or reads and no allele calling has been performed yet — run chewBBACA CreateSchema and AlleleCall first to obtain the profile matrix.
- Goal is the full wgMLST schema rather than the core genome — the wgMLST locus set should not be thresholded at 95% presence.
- Loci are used for strain-level wgMLST surveillance where accessory-genome loci matter; thresholding to 95% would discard them.

## Inputs

- Allelic profile matrix (results_alleles.tsv) from chewBBACA AlleleCall over the analysed assemblies
- RepeatedLoci.txt list of paralogous loci from the AlleleCall results_<datestamp> folder

## Outputs

- Paralog-filtered profile matrix (results_alleles_NoParalogs.tsv)
- cgMLST locus list file (e.g. cgMLSTschema.txt / cgMLST_loci_list.txt) with loci present in the chosen fraction of strains
- TestGenomeQuality HTML report with core-genome size vs. threshold plots

## How to apply

Start from the allelic profile matrix (results_alleles.tsv) produced by chewBBACA AlleleCall and first remove the paralogous loci listed in RepeatedLoci.txt with RemoveGenes, since duplicated loci carry uncertainty in allele assignment (e.g. 20 paralogous loci removed in the S. agalactiae tutorial, leaving 3108 loci). Run TestGenomeQuality on the paralog-free matrix to inspect how different presence thresholds (e.g. -n 13, -t 200, -s 5) affect the number of loci retained in the core genome and core-genome coverage per strain. Choose the threshold — 95% is recommended to tolerate missing calls caused by sequencing coverage and assembly problems; the core definition allows 95-99% presence. Then run ExtractCgMLST with the threshold (e.g. --t 0.95) to write the retained locus identifiers (e.g. 1267 loci for the 32 complete S. agalactiae genomes) to the output schema list (cgMLSTschema.txt), which can be fed back to AlleleCall via --gl to restrict allele calling to core loci.

## Related tools

- **chewBBACA** (Provides the RemoveGenes, TestGenomeQuality, and ExtractCgMLST modules used to filter paralogous loci and select core-genome loci from the allelic profile matrix) — https://github.com/B-UMMI/chewBBACA
- **Python 3.4+** (Runtime environment for chewBBACA, including the profile-filtering modules) — https://github.com/B-UMMI/chewBBACA

## Examples

```
chewBBACA.py ExtractCgMLST -i results32_wgMLST/results_<datestamp>/results_alleles_NoParalogs.tsv -o results32_wgMLST/results_<datestamp>/cgMLST_95 --t 0.95
```

## Evaluation signals

- Every locus in the output cgMLST list has a valid allele call in ≥95% (up to 99%) of the analysed strains.
- Number of removed paralogous loci equals the number of entries in RepeatedLoci.txt (e.g. 20 loci removed, profile drops from 3128 to 3108 loci).
- Locus count in the extracted cgMLST schema matches the threshold-annotated value from the TestGenomeQuality report (e.g. 1267 loci at 95% vs. 1136 loci at 100% presence for the 32 complete genomes).
- The cgMLSTschema.txt file can be passed to AlleleCall --gl without error, confirming the locus identifiers are valid schema members.

## Limitations

- Loci missing from some strains due to sequencing coverage and assembly problems motivate a 95% (rather than 100%) threshold, but this means the cgMLST set is dataset-dependent rather than a fixed schema.
- Paralogous loci in RepeatedLoci.txt carry potential uncertainty in allele assignment and should be removed before core-genome determination.
- The PrepExternalSchema-related caveats (e.g. backing up data before use) apply to schema manipulation workflows generally.

## Evidence

- [readme] The set of loci in the core genome is determined based on a threshold of loci presence in the analysed genomes.: [evidence span withheld — anchor status: external_doc]
- [readme] 95% threshold rationale: [evidence span withheld — anchor status: external_doc]
- [readme] paralog removal rationale: [evidence span withheld — anchor status: external_doc]
- [discussion] ExtractCgMLST efficiency note: "cgMLST profile extraction function (ExtractCgMLST) more efficient (thanks Dillon Barker)."
- [readme] core locus list reuse: [evidence span withheld — anchor status: external_doc]

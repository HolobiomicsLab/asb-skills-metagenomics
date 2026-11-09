---
name: chewbbaca-extractcgmlst-execution
description: Use when you have a chewBBACA AlleleCall results_alleles.tsv matrix (optionally
  paralog-filtered) and need to select the core-genome loci at a chosen presence threshold
  to define a cgMLST schema.
when_to_use_negative:
- The wgMLST schema itself must still be built or evaluated — use CreateSchema/AlleleCall/SchemaEvaluator,
  not ExtractCgMLST
- Paralogous loci have not yet been removed from the allele-calling results — run
  RemoveGenes on RepeatedLoci.txt first, since paralog loci introduce allele-assignment
  uncertainty
- You need to inspect the effect of multiple thresholds interactively — TestGenomeQuality
  (not ExtractCgMLST) is the module for threshold exploration
edam_operation: http://edamontology.org/operation_3695
tools:
- name: chewBBACA
  role: Provides the ExtractCgMLST module that extracts core-genome loci from the
    allele-calling matrix, and TestGenomeQuality to choose the presence threshold
  repo: https://github.com/B-UMMI/chewBBACA
- name: Python 3.4
  role: Runtime requirement for executing chewBBACA modules such as ExtractCgMLST
  repo: https://github.com/B-UMMI/chewBBACA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:e87a2bf738083fae5888c2a6e1bc7666a5e19d934b21f770f54e00e5b03b5697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - Python
  - Python 3.4
  - chewBBACA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# chewbbaca-extractcgmlst-execution

## Summary

Runs the chewBBACA ExtractCgMLST module to derive a cgMLST schema from a wgMLST allelic profile matrix by retaining only loci with valid allele calls in a preset fraction (typically 95-99%) of the analysed strains.

## When to use

Use when you have a chewBBACA AlleleCall results_alleles.tsv matrix (optionally paralog-filtered) and need to select the core-genome loci at a chosen presence threshold to define a cgMLST schema.

## When NOT to use

- The wgMLST schema itself must still be built or evaluated — use CreateSchema/AlleleCall/SchemaEvaluator, not ExtractCgMLST
- Paralogous loci have not yet been removed from the allele-calling results — run RemoveGenes on RepeatedLoci.txt first, since paralog loci introduce allele-assignment uncertainty
- You need to inspect the effect of multiple thresholds interactively — TestGenomeQuality (not ExtractCgMLST) is the module for threshold exploration

## Inputs

- Allelic profile matrix (TSV) from chewBBACA AlleleCall, e.g. results_alleles_NoParalogs.tsv
- Core-genome presence threshold (e.g. 0.95), optionally guided by TestGenomeQuality output
- Paralogous loci list (RepeatedLoci.txt) already removed from the matrix beforehand

## Outputs

- cgMLSTschema.txt: list of locus identifiers constituting the core genome at the chosen threshold (e.g. 1267 loci at 95% of 32 complete S. agalactiae genomes)
- cgMLST schema directory (e.g. cgMLST_95/) usable via the AlleleCall --gl parameter

## How to apply

Take the allelic profile matrix produced by chewBBACA AlleleCall (e.g. results_alleles_NoParalogs.tsv, after removing paralogous loci listed in RepeatedLoci.txt with RemoveGenes) as input to `chewBBACA.py ExtractCgMLST`. Optionally first run TestGenomeQuality (with parameters such as -n 13, -t 200, -s 5 in the tutorial) to see how several presence thresholds affect the number of core loci, then pass the chosen threshold via --t (e.g. 0.95 to account for loci missed due to sequencing coverage and assembly problems). ExtractCgMLST retains every locus with a valid allele call in at least the threshold fraction of the analysed strains and writes the retained locus identifiers to a cgMLSTschema.txt list, which can be fed to AlleleCall via --gl to call only core loci. The 95% threshold is a deliberate relaxation of the strict 100%-presence core definition so that loci absent due to assembly artefacts are not discarded.

## Related tools

- **chewBBACA** (Provides the ExtractCgMLST module that extracts core-genome loci from the allele-calling matrix, and TestGenomeQuality to choose the presence threshold) — https://github.com/B-UMMI/chewBBACA
- **Python 3.4** (Runtime requirement for executing chewBBACA modules such as ExtractCgMLST) — https://github.com/B-UMMI/chewBBACA

## Examples

```
chewBBACA.py ExtractCgMLST -i results32_wgMLST/results_<datestamp>/results_alleles_NoParalogs.tsv -o results32_wgMLST/results_<datestamp>/cgMLST_95 --t 0.95
```

## Evaluation signals

- Every locus in cgMLSTschema.txt has a valid allele call in at least 95% of the analysed strains (tutorial: 1267 of 3108 non-paralog loci at 95% across 32 complete genomes)
- The number of extracted core loci matches the count reported by TestGenomeQuality for the same threshold (1136 loci at 100%, 1267 at 95% in the tutorial)
- Output file cgMLSTschema.txt is generated inside the -o directory (e.g. cgMLST_95/) and contains one locus identifier per line
- The extracted list is accepted as --gl input by a subsequent AlleleCall run without errors
- Locus count in the cgMLST list is strictly smaller than or equal to the wgMLST locus count in the input matrix

## Limitations

- Loci present below the chosen threshold (e.g. <95%) are permanently excluded from the cgMLST schema; only the threshold fraction, not strain-level clinical context, determines inclusion
- The extracted schema is dataset-specific — the core set depends on the analysed strain subset and can change when new genomes are added
- Threshold selection trades schema size against robustness: 100% presence gave 1136 loci vs 1267 at 95% in the tutorial, because low-coverage/assembly problems cause false absences

## Evidence

- [readme] ExtractCgMLST derives core loci list: [evidence span withheld — anchor status: external_doc]
- [readme] Threshold exploration rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Justification for 95% threshold: [evidence span withheld — anchor status: external_doc]
- [abstract] Core-genome computation as chewBBACA functionality: "The chewBBACA suite was designed to assist users in the creation and evaluation of novel whole-genome or core-genome gene-by-gene typing schemas"
- [discussion] Efficiency note on the module: "cgMLST profile extraction function (ExtractCgMLST) more efficient (thanks Dillon Barker)."

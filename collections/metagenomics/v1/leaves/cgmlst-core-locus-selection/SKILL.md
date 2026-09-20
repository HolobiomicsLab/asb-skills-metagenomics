---
name: cgmlst-core-locus-selection
description: Use when you have a chewBBACA allelic profile matrix (results_alleles.tsv)
  over draft or complete assemblies and must reduce the wgMLST schema to loci present
  in ≥95% (or 95-99%) of strains for core-genome typing.
when_to_use_negative:
- Input is raw FASTA assemblies rather than a completed chewBBACA AlleleCall profile
  matrix — schema creation/allele calling must precede core-locus selection
- Paralogous loci from RepeatedLoci.txt have not yet been removed, so uncertain allele
  assignments would inflate the core-locus count
- A pre-defined cg/wgMLST schema already exists (it can be downloaded from Chewie-NS
  or adapted via PrepExternalSchema instead of being re-derived)
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: chewBBACA
  role: Provides TestGenomeQuality (threshold exploration) and ExtractCgMLST (core-locus
    extraction) modules applied to the allele-calling matrix
  repo: https://github.com/B-UMMI/chewBBACA
- name: Python 3.4
  role: Runtime environment required by the chewBBACA suite
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:de89254f2adbe03c5768a2400a2485fd4b92ccd5a8285e17f6bf53ed0fefc260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - Python
  - Python 3.4
  - chewBBACA
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cgmlst-core-locus-selection

## Summary

Defines a cgMLST (core-genome) schema by selecting loci present in a chosen percentage of analysed strains (e.g. 95%) from a chewBBACA wgMLST allele-calling matrix, using TestGenomeQuality to explore thresholds and ExtractCgMLST to write the retained locus list.

## When to use

Use when you have a chewBBACA allelic profile matrix (results_alleles.tsv) over draft or complete assemblies and must reduce the wgMLST schema to loci present in ≥95% (or 95-99%) of strains for core-genome typing.

## When NOT to use

- Input is raw FASTA assemblies rather than a completed chewBBACA AlleleCall profile matrix — schema creation/allele calling must precede core-locus selection
- Paralogous loci from RepeatedLoci.txt have not yet been removed, so uncertain allele assignments would inflate the core-locus count
- A pre-defined cg/wgMLST schema already exists (it can be downloaded from Chewie-NS or adapted via PrepExternalSchema instead of being re-derived)

## Inputs

- chewBBACA allelic profile matrix (results_alleles.tsv or results_alleles_NoParalogs.tsv) from a wgMLST allele call
- RepeatedLoci.txt paralog list (recommended preprocessing to exclude paralogous loci)
- presence-percentage threshold (e.g. 0.95)

## Outputs

- cgMLSTschema.txt listing the loci in the core genome at the chosen threshold
- TestGenomeQuality HTML plot of genome quality versus threshold
- optional reduced allelic profile matrix restricted to core loci (via --gl in AlleleCall)

## How to apply

Start from the allelic profile matrix produced by chewBBACA AlleleCall over the Streptococcus agalactiae assemblies, ideally after removing the paralogous loci listed in RepeatedLoci.txt via RemoveGenes (this reduced the 32-genome set from 3128 to 3108 loci). Run TestGenomeQuality with parameters such as -n 13 -t 200 -s 5 to visualise how different presence thresholds affect the number of core loci, then choose a threshold (the tutorial used 95%) that tolerates loci missed due to sequencing coverage and assembly problems. Execute ExtractCgMLST with --t 0.95 on the paralog-free matrix to extract the retained loci; in the tutorial this yielded 1267 loci at 95% versus 1136 loci present in all 32 complete genomes. The output cgMLSTschema.txt locus list can be fed back into AlleleCall via the --gl parameter to call alleles only for core genes.

## Related tools

- **chewBBACA** (Provides TestGenomeQuality (threshold exploration) and ExtractCgMLST (core-locus extraction) modules applied to the allele-calling matrix) — https://github.com/B-UMMI/chewBBACA
- **Python 3.4** (Runtime environment required by the chewBBACA suite)

## Examples

```
chewBBACA.py ExtractCgMLST -i results32_wgMLST/results_<datestamp>/results_alleles_NoParalogs.tsv -o results32_wgMLST/results_<datestamp>/cgMLST_95 --t 0.95
```

## Evaluation signals

- Every locus in cgMLSTschema.txt has a valid allele call in ≥95% (or the chosen 95-99%) of the analysed strains
- Threshold sensitivity: tutorial baseline shows 1136 loci in 100% of complete genomes and 1267 loci at ≥95% — your counts should bracket this pattern
- The retained-locus count must not exceed the paralog-free wgMLST locus count (3108 loci in the tutorial after removing 20 paralogs)
- Re-running AlleleCall with the core list passed to --gl yields profiles containing only the core loci

## Limitations

- Threshold choice trades strictness for coverage: 95% was chosen to account for loci missed due to sequencing coverage and assembly problems, so a 95% core includes loci absent from some strains
- Paralogous loci (20 in the tutorial dataset) cause potential uncertainty in allele assignment and should be removed before core definition
- TestGenomeQuality automatically opens an HTML plot, which may be unsuitable for non-interactive/cluster environments

## Evidence

- [readme] The set of loci in the core genome is determined based on a threshold of loci presence in the analysed genomes. We can run the TestGenomeQuality module to determine the impact of several threshold values on the number of loci in the core genome.: [evidence span withheld — anchor status: external_doc]
- [readme] Threshold rationale for tolerating assembly artifacts.: [evidence span withheld — anchor status: external_doc]
- [readme] Expected core-locus counts at 100% and 95% thresholds.: [evidence span withheld — anchor status: external_doc]
- [readme] ExtractCgMLST is the module that determines the core-locus set.: [evidence span withheld — anchor status: external_doc]
- [readme] Output file location for the retained core loci.: [evidence span withheld — anchor status: external_doc]
- [discussion] Article confirms the ExtractCgMLST functionality for core-genome loci computation.: "cgMLST profile extraction function (ExtractCgMLST) more efficient (thanks Dillon Barker)."

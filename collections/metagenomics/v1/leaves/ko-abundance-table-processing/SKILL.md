---
name: ko-abundance-table-processing
description: Use when a shotgun metagenomics workflow has produced a length-normalized,
  fractionally counted KO abundance table and you need cross-sample-comparable functional
  profiles for aggregation or downstream statistical analysis.
when_to_use_negative:
- Input is still a gene-level count table that has not yet been mapped to KOs (run
  ortholog counting first).
- You deliberately want simple relative abundances instead of copy-number-per-genome
  normalization (set the method parameter to 'relative').
- Data are not shotgun metagenomic samples with a bacterial gene-content signal (e.g.,
  16S amplicon or single-isolate RNA-seq), where single-copy-gene normalization is
  not applicable.
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaLAFFA
  role: Pipeline whose functional annotation phase performs the KO abundance correction
    step via config/steps/ortholog_abundance_correction.py
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MUSiCC
  role: Default method for correcting ortholog abundances into average copy number
    per genome, with inter-sample normalization and intra-sample correction based
    on universally single-copy bacterial genes
- name: Python
  role: Module-based configuration system that exposes step parameters such as the
    MUSiCC 'method' setting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:1203c508b7c06d2556139e7966566a6651bce148716b67fa166a89039415787c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MUSiCC
  - MetaLAFFA
  - Python
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ko-abundance-table-processing

## Summary

In MetaLAFFA's functional annotation phase, gene-level counts mapped to KEGG orthology groups (KOs) are corrected with MUSiCC, which by default converts ortholog read counts into average copy number per genome per sample rather than leaving raw or relative abundances.

## When to use

Use when a shotgun metagenomics workflow has produced a length-normalized, fractionally counted KO abundance table and you need cross-sample-comparable functional profiles for aggregation or downstream statistical analysis.

## When NOT to use

- Input is still a gene-level count table that has not yet been mapped to KOs (run ortholog counting first).
- You deliberately want simple relative abundances instead of copy-number-per-genome normalization (set the method parameter to 'relative').
- Data are not shotgun metagenomic samples with a bacterial gene-content signal (e.g., 16S amplicon or single-isolate RNA-seq), where single-copy-gene normalization is not applicable.

## Inputs

- KO abundance table (TSV) from the ortholog counting step, with gene-length-normalized fractional counts
- Ortholog abundance correction configuration (config/steps/ortholog_abundance_correction.py)

## Outputs

- MUSiCC-corrected KO abundance table (average KO copy number per genome per sample) as a TSV file

## How to apply

Take the KO abundance table produced by the ortholog counting step, where gene abundances have already been gene-length normalized and fractionally counted across multi-matching reads. Invoke the ortholog abundance correction step implemented in config/steps/ortholog_abundance_correction.py, keeping the default MUSiCC method (the 'method' parameter at its default rather than switching to 'relative'). MUSiCC (Manor and Borenstein 2015) performs inter-sample normalization and intra-sample correction using learned models, leveraging universally single-copy bacterial genes as the normalization basis to account for shifts in community gene content. Write the corrected table — average KO copy number per genome per sample — to disk as a tab-separated values (TSV) file, ready for aggregation into higher-level functional descriptions such as KEGG modules and pathways.

## Related tools

- **MetaLAFFA** (Pipeline whose functional annotation phase performs the KO abundance correction step via config/steps/ortholog_abundance_correction.py) — https://github.com/borenstein-lab/MetaLAFFA
- **MUSiCC** (Default method for correcting ortholog abundances into average copy number per genome, with inter-sample normalization and intra-sample correction based on universally single-copy bacterial genes)
- **Python** (Module-based configuration system that exposes step parameters such as the MUSiCC 'method' setting)

## Examples

```
python config/steps/ortholog_abundance_correction.py --input ortholog_counts.tsv --output musicc_corrected_ko_abundances.tsv  # keep default MUSiCC 'method' parameter
```

## Evaluation signals

- Output TSV file exists at the expected output location for the ortholog abundance correction step.
- Output parses as a tab-delimited table with rows as KOs and columns as samples.
- Values represent corrected average copy numbers per genome, not raw read counts or simple relative abundances.
- The 'method' parameter in config/steps/ortholog_abundance_correction.py is left at its MUSiCC default (not 'relative').
- Downstream aggregation steps (KO-to-module, KO-to-pathway) can consume the corrected table without schema errors.

## Limitations

- MUSiCC's intra-sample correction depends on universally single-copy bacterial genes; samples with unusual gene content or few detectable single-copy markers may be poorly normalized.
- A known runtime issue: MetaLAFFA users have encountered an 'undefined symbol: omp_get_num_procs' error when running MUSiCC, which must be fixed per the FAQ.
- The MUSiCC step can be resource-intensive in the context of the full database-preparation/pipeline environment, which itself can require ~40GB RAM and hundreds of GB of disk.

## Evidence

- [methods] MUSiCC-corrected KO abundances as default: [evidence span withheld — anchor status: not_found]
- [intro] functional annotation phase description: "the functional annotation phase translates these gene abundances into classifications of community functional capacities"
- [methods] fractional counting and length normalization precondition: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [readme] README step listing: "8. Correcting orthology group abundances"
- [readme] README MUSiCC FAQ: "How do I fix the "undefined symbol: omp_get_num_procs" error when MetaLAFFA tries to run MUSiCC?"

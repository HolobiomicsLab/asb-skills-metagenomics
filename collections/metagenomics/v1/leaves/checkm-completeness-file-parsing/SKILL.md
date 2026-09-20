---
name: checkm-completeness-file-parsing
description: Use when running mOTUpan on MAGs/SAGs of varying quality and you have
  (or can produce) a CheckM output file to supply completeness and contamination priors
  per genome.
when_to_use_negative:
- Your input genomes are complete isolates of uniformly high quality — the Bayesian
  completeness prior and CheckM estimates add little and the trait partitioning can
  be computed directly
- You want completeness estimates themselves rather than to feed them into a core-genome
  classification — run CheckM standalone instead
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: mOTUpan
  role: Consumes the parsed completeness/contamination table as the completeness prior
    for its iterative Bayesian core/accessory classification
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: CLI entry point that accepts the CheckM file and writes updated completeness
    estimates with the trait partitioning to output.tsv
  repo: github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Generates the Completeness/Contamination table that is parsed as the completeness
    prior (or a fixed value may be supplied instead)
  repo: ecogenomics.github.io/CheckM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:55a4aa4faac1165adbca330e24e8527989e6261400639c0e874ae57ed2d468ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# checkm-completeness-file-parsing

## Summary

Parses CheckM-style completeness/contamination tables into the completeness prior that mOTUpan's Bayesian core/accessory classifier iteratively re-estimates. Correct parsing of the Bin Id, Completeness, Contamination columns (values 0-100) is what grounds the core-genome likelihood computation in genome quality.

## When to use

Use when running mOTUpan on MAGs/SAGs of varying quality and you have (or can produce) a CheckM output file to supply completeness and contamination priors per genome.

## When NOT to use

- Your input genomes are complete isolates of uniformly high quality — the Bayesian completeness prior and CheckM estimates add little and the trait partitioning can be computed directly
- You want completeness estimates themselves rather than to feed them into a core-genome classification — run CheckM standalone instead

## Inputs

- CheckM output table (e.g. example_files/example_checkm.txt) with columns Bin Id, Completeness, Contamination (values 0-100)
- Fixed completeness value (alternative to a CheckM file)
- Genome-to-trait mapping file (JSON or TAB-separated cog file, e.g. example_files/example_genome2cog.tsv)

## Outputs

- Per-genome completeness prior consumed by the mOTUpan Bayesian core/accessory likelihood loop
- Updated completeness estimates written alongside the core/accessory trait partitioning to output.tsv

## How to apply

Load a CheckM output table (e.g. example_files/example_checkm.txt) requiring the columns Bin Id, Completeness, and Contamination with values on a 0-100 scale; this can also be a fixed value instead of a file. Map each genome's completeness estimate to the genome-to-trait mapping (e.g. example_genome2cog.tsv, a JSON or TAB-separated cog file) so every presence/absence pattern is scored under a quality-weighted prior. mOTUpan then uses these completeness values in its Bayesian estimator (Equations 1-6), re-estimating completeness from the classified core partition iteratively until convergence, capped by --max_iter. When completeness is not provided to mOTUlize.py, all genomes are assumed to be high-quality 'MAGs', which changes mOTU network construction. Note that mOTUlize.py uses contamination, whereas mOTUpan.py uses the completeness prior for the core-genome likelihood loop.

## Related tools

- **mOTUpan** (Consumes the parsed completeness/contamination table as the completeness prior for its iterative Bayesian core/accessory classification) — github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (CLI entry point that accepts the CheckM file and writes updated completeness estimates with the trait partitioning to output.tsv) — github.com/moritzbuck/mOTUlizer
- **CheckM** (Generates the Completeness/Contamination table that is parsed as the completeness prior (or a fixed value may be supplied instead)) — ecogenomics.github.io/CheckM

## Examples

```
mOTUpan.py --faas *.faa --checkm example_files/example_checkm.txt -o output.tsv
```

## Evaluation signals

- The CheckM table parses cleanly with exactly the required columns Bin Id, Completeness, Contamination and all values in the 0-100 range
- Every genome named in the genome2cog mapping has a matching completeness entry; no genome silently defaults to an assumed prior
- The iterative loop converges (or stops at --max_iter) and the final completeness estimates differ from the initial CheckM priors only insofar as the core partitioning changed
- The output.tsv contains both the core/accessory partitioning and the updated per-genome completeness estimates

## Limitations

- MAG/SAG incompleteness is inherently an estimate; the prior is only as good as the CheckM (or fixed) values supplied
- If no --checkm file is given to mOTUlize.py, all genomes are assumed high-quality 'MAGs', which can mislead mOTU network construction
- For mOTUlize.py the parser ignores any lines containing the word 'query' in the similarities file (a parsing quirk to be aware of when combining files)

## Evidence

- [other] completeness/contamination table (e.g. example_files/example_checkm.txt, requiring Bin Id, Completeness, Contamination columns with values 0-100): [evidence span withheld — anchor status: not_found]
- [readme] README on checkm option: "--checkm`: provide a file containing completenesses and contaminations, see `mOTUpan`-section for detail, this time though, contamination is used... also, if not provided all genomes are assumed"
- [readme] README on completeness prior and iterative re-estimation: "as well as a completeness prior of these genomes (for example [`checkm`](https://ecogenomics.github.io/CheckM/) output or a fixed value)"
- [methods] Methods on iterative completeness update: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."

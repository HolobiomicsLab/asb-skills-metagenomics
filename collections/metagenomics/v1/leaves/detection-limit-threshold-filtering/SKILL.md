---
name: detection-limit-threshold-filtering
description: Use when calling fecal source presence/absence in short-read metagenomes
  from SourceApp's genome-equivalent-normalized cell fraction estimates, where spurious
  low-level signal must be filtered before computing apportionment portions.
when_to_use_negative:
- Inputs are long reads or non-FASTQ data — SourceApp's mapping filters were tuned
  for short 2x150-bp Illumina NovaSeq 6000 reads
- The goal is a mass balance of fecal contamination — SourceApp portions estimate
  proportions of prokaryotic cells based on genome equivalents, not mass balance
- Estimating contamination age or detecting growth-capable (nonconservative) populations
  — the tool explicitly cannot do this
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: Implements the attribution rule (LOD 0.01% cell fraction plus -r identity,
    -q query-coverage, and -l limit-threshold filters) and writes attribution/apportionment/fractioning
    CSVs
  repo: https://github.com/bglindner/SourceApp
- name: CoverM
  role: Read-mapping coverage calculator used to obtain per-genome read counts and
    sequence depths fed into the detection-threshold logic
  repo: https://github.com/wwood/CoverM
- name: MicrobeCensus
  role: Estimates genome equivalents (GEQ) used to normalize summed depths into cell
    fractions compared against the LOD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:a875f5ad665368c01a01c9ec75d6ac2dcdb487c0cc685cd89bfbef057eb0063c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# Detection limit threshold filtering

## Summary

Applies SourceApp's detection thresholds — a per-source limit of detection (LOD, 0.01% cell fraction by default) plus read-mapping filters (93% identity, 70% query coverage, sequence breadth limit) — to decide whether a fecal source is present in a shotgun metagenome before apportioning cell fractions among present sources.

## When to use

Use when calling fecal source presence/absence in short-read metagenomes from SourceApp's genome-equivalent-normalized cell fraction estimates, where spurious low-level signal must be filtered before computing apportionment portions.

## When NOT to use

- Inputs are long reads or non-FASTQ data — SourceApp's mapping filters were tuned for short 2x150-bp Illumina NovaSeq 6000 reads
- The goal is a mass balance of fecal contamination — SourceApp portions estimate proportions of prokaryotic cells based on genome equivalents, not mass balance
- Estimating contamination age or detecting growth-capable (nonconservative) populations — the tool explicitly cannot do this

## Inputs

- Gzipped raw shotgun metagenomic reads in FASTQ format from waterway samples
- SourceApp reference genome database with source-association annotations (default database from Zenodo or user-built via sourceapp_build.py)
- MicrobeCensus genome-equivalent (GEQ) estimates per metagenome
- Per-genome sequence depths and mapped read counts

## Outputs

- Source attribution CSV table (presence/absence per source category)
- Apportionment CSV table (Portion per present source)
- Fractioning CSV table (Fraction in c/c units with --use-geq)

## How to apply

Map metagenomic reads to source-associated genome databases with filters of at least 93% read-to-genome alignment identity and at least 70% of the read covered by the alignment, and consider a genome detected only when sequence breadth exceeds the -l/--limit-threshold parameter. Sum per-genome sequence depths across source-specific and cross-reactive genomes per source category, normalize by genome equivalents estimated with MicrobeCensus to obtain cell fraction_j = sum(X_i)/GEQ. Call a source present only when at least one source-specific genome is detected AND the source's estimated cell fraction exceeds the LOD (0.01% by default). Compute apportionment portions only among sources passing this threshold, dividing each present source's summed cell fraction by the sum across all present sources; parameter tuning showed this configuration yields 0.898 specificity and 0.902 sensitivity.

## Related tools

- **SourceApp** (Implements the attribution rule (LOD 0.01% cell fraction plus -r identity, -q query-coverage, and -l limit-threshold filters) and writes attribution/apportionment/fractioning CSVs) — https://github.com/bglindner/SourceApp
- **CoverM** (Read-mapping coverage calculator used to obtain per-genome read counts and sequence depths fed into the detection-threshold logic) — https://github.com/wwood/CoverM
- **MicrobeCensus** (Estimates genome equivalents (GEQ) used to normalize summed depths into cell fractions compared against the LOD)

## Examples

```
sourceapp.py --use-geq --percent-identity 0.93 --query-coverage 0.7 --limit-of-detection 0.0001 -i mesocosm_*.fastq.gz -d sourceapp_db
```

## Evaluation signals

- Specificity near 0.898 and sensitivity near 0.902 when scored against ground-truth cell fractions from spike-in mesocosms (Table S1)
- Attribution calls are binary per source: present only when at least one source-specific genome is detected AND cell fraction > 0.01% (LOD)
- Portion values for present sources must sum to 1 within each metagenome (each portion = source cell fraction / sum of present sources' cell fractions)
- Cell fraction estimates (Fraction, c/c units with --use-geq) should correlate with constructed mesocosm proportions, though residuals remain (best r2 = 0.64 after linear correction)
- Lowering the sequence breadth limit-threshold should reduce false negatives; raising it increases false-negative rate per the parameter documentation

## Limitations

- Cow sources are underestimated/false negatives across samples due to under sampling of cow fecal microbiomes and poor understanding of cow fecal prokaryotic content at the genomic level
- Persistence of species-level marker genomes in the environment is largely unknown, so detection thresholds may not reflect viable contamination
- Apportionment portions are not a mass balance and may vary from mass-based expectations; substantial residuals remain after linear correction (r2 = 0.64)

## Evidence

- [methods] the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default): "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] Optimal parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [readme] SourceApp README LOD/flags: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [methods] Sequence breadth threshold documentation: "Sequence breadth needed to consider a genome detected. Increasing this value will increase false negative rate."
- [results] Benchmarking against gold standard: "no work exists which has benchmarked metagenomic FST methodologies against gold standard data sets such as the spike-in mesocosms constructed herein"

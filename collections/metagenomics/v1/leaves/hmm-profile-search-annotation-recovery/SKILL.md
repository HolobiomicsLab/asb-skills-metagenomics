---
name: hmm-profile-search-annotation-recovery
description: Use when MAG or draft-genome eggNOG/KAAS/KofamKoala KO annotations leave
  KEGG Modules incomplete and unannotated genes (including partially truncated sequences)
  need evidence-based KO recovery via profile HMMs.
when_to_use_negative:
- The goal is only KEGG Module completeness evaluation without annotation expansion
  — use --skip_hmm instead; --no_genome mode stops after completeness evaluation
- A genome-scale metabolic model must be gapfilled rather than annotated — this is
  the HMM search stage only; run KEMET's --gsmm_mode (denovo/existing) with CarveMe
  v1.4.1 (--fbc2 -u) models separately
- Input KOs are already exhaustively annotated or the genome is complete with high-confidence
  KOs, so HMM recovery adds no missing-ortholog signal
tools:
- name: KEMET
  role: Runs the profile HMM search for missing KOs (--hmm_mode onebm) with the legit-hit
    quality threshold (default 0.43) and produces MODULE recap tables
  repo: https://github.com/Matteopaluh/KEMET
- name: HMMER
  role: Underlying profile HMM search engine used by KEMET; soft-errors can be silenced
    with -q
- name: eggNOG-Mapper
  role: Produces the eggnog-format KO annotations (.emapper.annotations) that serve
    as KEMET's input annotation set
- name: Prodigal
  role: Gene calling (v2.6.3, default options) upstream of annotation and HMM recovery
- name: IGGdb
  role: Source of reference complete genomes from PATRIC used as ground-truth comparators
    for concordance testing
  repo: https://github.com/snayfach/IGGdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:06c83f95619f3075bd693c78e16755f629d409134ef58a6209554c19465fdc91
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - HMMER
  - IGGdb
  - KEMET
  - Prodigal
  - eggNOG-Mapper
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hmm-profile-search-annotation-recovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover missing KEGG ortholog (KO) annotations in MAGs or draft genomes by running KEMET's profile HMM searches (HMMER/MAFFT under the hood) against curated KO profiles, and benchmark recovery quality by comparing MAG hits against the corresponding complete reference isolate genomes.

## When to use

Use when MAG or draft-genome eggNOG/KAAS/KofamKoala KO annotations leave KEGG Modules incomplete and unannotated genes (including partially truncated sequences) need evidence-based KO recovery via profile HMMs.

## When NOT to use

- The goal is only KEGG Module completeness evaluation without annotation expansion — use --skip_hmm instead; --no_genome mode stops after completeness evaluation
- A genome-scale metabolic model must be gapfilled rather than annotated — this is the HMM search stage only; run KEMET's --gsmm_mode (denovo/existing) with CarveMe v1.4.1 (--fbc2 -u) models separately
- Input KOs are already exhaustively annotated or the genome is complete with high-confidence KOs, so HMM recovery adds no missing-ortholog signal

## Inputs

- MAG or genome FASTA file (e.g. genomes/bin1.fasta)
- KEGG KO annotation file in eggNOG format (.emapper.annotations; KAAS .ko or kofamkoala also accepted)
- Matching complete reference isolate genome FASTA plus its KO annotation file (from NCBI or PATRIC, e.g. via IGGdb)
- KEMET-curated KO HMM profiles from the KEGG GENES database (updatable via the KEGG API)

## Outputs

- Set of recovered (previously missing) KOs per genome from the HMM search
- KEGG MODULES recap tables of metabolic potential
- Per-pair KO hit concordance classification (common / MAG-only / complete-genome-only)
- Aggregated concordance metrics (e.g. hmm_concordance_metrics.csv: 84.76% shared, 7.62% genome-only, 7.62% MAG-only; precision and sensitivity ≈91.75%)

## How to apply

Annotate predicted genes (Prodigal v2.6.3 gene calling, eggNOG-mapper v2 default parameters) to obtain .emapper.annotations, then run kemet.py on each MAG FASTA with -a eggnog --hmm_mode onebm --skip_gsmm so KEMET searches for missing KOs with profile HMMs; keep the default legit-hit quality threshold of 0.43 (--threshold_value) unless a stricter filter is justified, and use -q to silence MAFFT and HMMER soft errors. Repeat the identical KEMET run on the matching complete reference genome (recovered from NCBI or PATRIC, e.g. via the IGGdb pointer) to obtain a ground-truth hit set. For each genome pair, compare KO hit sets and classify each hit as common, MAG-only, or complete-genome-only, then aggregate across pairs to compute concordance fractions and derived precision/sensitivity. In the KEMET validation this workflow yielded 84.76% shared hits, 7.62% complete-genome-only, and 7.62% MAG-only (precision and sensitivity both ≈91.75%); mock KO-removal tests recovered 20/36 (55%) for MAGs and 8/12 (67%) for complete genomes, including genes truncated to 30% or 70% of original length.

## Related tools

- **KEMET** (Runs the profile HMM search for missing KOs (--hmm_mode onebm) with the legit-hit quality threshold (default 0.43) and produces MODULE recap tables) — https://github.com/Matteopaluh/KEMET
- **HMMER** (Underlying profile HMM search engine used by KEMET; soft-errors can be silenced with -q)
- **eggNOG-Mapper** (Produces the eggnog-format KO annotations (.emapper.annotations) that serve as KEMET's input annotation set)
- **Prodigal** (Gene calling (v2.6.3, default options) upstream of annotation and HMM recovery)
- **IGGdb** (Source of reference complete genomes from PATRIC used as ground-truth comparators for concordance testing) — https://github.com/snayfach/IGGdb

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --skip_gsmm -q --log
```

## Evaluation signals

- Hit-concordance fractions match the reported 84.76% common, 7.62% complete-genome-only, 7.62% MAG-only, giving precision and sensitivity of 84.76/(84.76+7.62) ≈ 91.75%
- Mock KO-removal validation recovers the correct gene and annotation at roughly the reported rates: 20/36 (55%) in MAGs and 8/12 (67%) in complete genomes
- Recovered hits pass the KEMET legit-hit quality threshold (default 0.43); hits below threshold are excluded
- Recovery succeeds even for gene sequences truncated to 30% or 70% of their original length, as tested in the paper
- Every KEMET run uses identical parameters for MAG and reference genome (same -a eggnog, --hmm_mode onebm, --skip_gsmm) so hit-set comparison is like-for-like

## Limitations

- Recovery is imperfect: 7.62% false positives (MAG-only) and 7.62% false negatives (genome-only) were observed, so recovered KOs should not be treated as certain
- MAG-based GSMMs are prone to reconstruction errors due to the gapped nature of metagenomic assemblies, limiting downstream model fidelity
- HMM analysis requires MAFFT and HMMER, whose soft errors may need silencing with -q; MAG quality matters (high-quality MAGs should be ≥90% complete, ≤5% contamination)

## Evidence

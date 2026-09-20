---
name: viral-orf-prediction-codon-usage
description: Use when binning viral scaffolds (e.g., ≥2000 bp candidates from VIBRANT/VirSorter)
  and you need gene/protein calls to compute codon usage as a sequence-composition
  feature for scaffold-pair similarity.
when_to_use_negative:
- You already have gene calls or a proteins file from an external predictor and plan
  to pass them via vRhyme's -p or -g flags instead
- The input is a whole mixed metagenome intended for microbial binning — vRhyme is
  'not meant to bin microbes' and performance on full metagenomes is unevaluated
- Scaffolds are shorter than the 2000 bp minimum length for binning and would only
  add noise to the feature table
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Predicts open reading frames on viral scaffolds using '-m -p meta' to obtain
    genes/proteins for codon usage features
  repo: https://github.com/hyattpd/Prodigal
- name: vRhyme
  role: Consumes the ORF predictions to compute codon usage frequency vectors and
    pairwise cosine distances for supervised machine learning binning
  repo: https://github.com/AnantharamanLab/vRhyme
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:cd5693f2b19815fc28e365b5a8462c76b86ce799df445ca385dcee29a112ab42
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/hyattpd/Prodigal
  tools:
  - Numpy
  - Pandas
  - Prodigal
  - Python
  - vRhyme
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# viral-orf-prediction-codon-usage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts protein-coding open reading frames on viral scaffolds with Prodigal in metagenomic mode and derives per-scaffold codon usage frequency vectors that feed vRhyme's supervised machine learning nucleotide-feature distances.

## When to use

Use when binning viral scaffolds (e.g., ≥2000 bp candidates from VIBRANT/VirSorter) and you need gene/protein calls to compute codon usage as a sequence-composition feature for scaffold-pair similarity.

## When NOT to use

- You already have gene calls or a proteins file from an external predictor and plan to pass them via vRhyme's -p or -g flags instead
- The input is a whole mixed metagenome intended for microbial binning — vRhyme is 'not meant to bin microbes' and performance on full metagenomes is unevaluated
- Scaffolds are shorter than the 2000 bp minimum length for binning and would only add noise to the feature table

## Inputs

- Viral scaffold FASTA file (predicted viral sequences, scaffolds ≥2000 bp minimum binning length)
- Prodigal executable in PATH (optional dependency of vRhyme, required when skipping -p/-g flags)

## Outputs

- Predicted genes file (GFF3-style gene calls) per scaffold
- Predicted proteins FASTA (.faa) used for codon usage feature extraction
- Per-scaffold codon usage frequency vectors compared as cosine distances in the seven-feature pairwise distance table vRhyme_machine_distances.tsv

## How to apply

Run Prodigal on the filtered viral scaffold FASTA with the '-m -p meta' flags: '-p meta' selects the anonymous metagenome mode appropriate for viral scaffolds that lack training genomes, and '-m' retains partial genes so short viral scaffolds still contribute codon counts. Use the predicted genes/proteins to compute per-scaffold codon usage frequency vectors as one of the nucleotide composition features alongside tetranucleotide frequencies, GC content, CpG content, and GC skew. Compare codon usage vectors between scaffold pairs by cosine distance (one of the seven pairwise feature distances, M_i,j) that populate vRhyme_machine_distances.tsv for the machine learning models. Codon usage is sequence-derived, so it complements coverage-variance evidence to separate scaffolds that 'sound the same' but belong to different genomes.

## Related tools

- **Prodigal** (Predicts open reading frames on viral scaffolds using '-m -p meta' to obtain genes/proteins for codon usage features) — https://github.com/hyattpd/Prodigal
- **vRhyme** (Consumes the ORF predictions to compute codon usage frequency vectors and pairwise cosine distances for supervised machine learning binning) — https://github.com/AnantharamanLab/vRhyme

## Examples

```
prodigal -i viral_scaffolds.fna -o genes.gff -a proteins.faa -p meta -m
```

## Evaluation signals

- vRhyme's test_vRhyme.py reports Prodigal as 'Success' when ORF prediction is used (required if -p/-g flags are skipped)
- Predicted protein FASTA is non-empty and contains translations for scaffolds ≥2000 bp, including partial genes retained by '-m'
- The resulting vRhyme_machine_distances.tsv contains raw pairwise distance values for every scaffold pair, including the codon usage (CU) column
- Codon usage distances are cosine distances bounded in [0, 2] and comparable in scale to the other vector-based metrics (RTF, TUD, ZOM)

## Limitations

- Prodigal is an optional vRhyme dependency — it is only required to be 'Success' if you skip vRhyme's -p or -g flags; otherwise external gene/protein inputs are expected
- Codon usage alone cannot separate scaffolds of different genomes with similar composition; vRhyme layers coverage variance comparisons on top of nucleotide features
- Metagenomic mode ('-p meta') is needed because viral scaffolds lack a training genome, which may be less accurate than single-genome mode on near-complete genomes

## Evidence

- [methods] ORF prediction with Prodigal using '-m -p meta' to obtain genes/proteins for codon usage features: [evidence span withheld — anchor status: not_found]
- [methods] Pairwise cosine distances for frequency-vector metrics including codon usage: [evidence span withheld — anchor status: not_found]
- [readme] Prodigal metagenomic mode invocation from README: [evidence span withheld — anchor status: external_doc]

---
name: plasmid-prediction-benchmark-evaluation
description: Use when evaluating or reproducing plasmid prediction performance on
  assembled bacterial contigs, e.g. recovering plasmid recall/precision on the 42
  plasmid-rich genomes of Arredondo-Alonso et al. and comparing against cBar and Recycler
  baselines.
when_to_use_negative:
- Input is raw paired-end reads rather than assembled contigs — Recycler- or PlasmidSpades-style
  approaches requiring an assembly graph fit that case instead
- 'Contigs are shorter than 1000 bp: ''PlasFlow, similarly to other kmer-based methods,
  does not perform well on short sequences'''
- The goal is identifying plasmid replicons by database similarity (PlasmidFinder)
  rather than signature-based whole-sequence classification
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PlasFlow
  role: Neural-network classifier that labels contigs as plasmid/chromosome by phylum
    and bins them into FASTA outputs; invoked with --threshold 0.7
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script used before classification to retain only contigs longer
    than 1000 bp
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: SOM-based plasmid-prediction baseline used for recall/precision comparison
- name: Recycler
  role: Assembly-graph circular-contig recovery tool used as a comparison baseline
  repo: https://github.com/Shamir-Lab/Recycler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:59ec4f6e9fc17e9268cdba9ab7c8750c19cd21167a1a55595852ad5071132af6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/smaegol/PlasFlow
  repo_url: https://github.com/smaegol/PlasFlow
  tools:
  - PlasFlow
  - Recycler
  - cBar
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# plasmid-prediction-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for comparing plasmid-sequence prediction tools (PlasFlow, cBar, Recycler) on SPAdes-assembled bacterial genomes, computing plasmid recall and precision against GenBank reference plasmids using k-mer-signature neural-network classification.

## When to use

Use when evaluating or reproducing plasmid prediction performance on assembled bacterial contigs, e.g. recovering plasmid recall/precision on the 42 plasmid-rich genomes of Arredondo-Alonso et al. and comparing against cBar and Recycler baselines.

## When NOT to use

- Input is raw paired-end reads rather than assembled contigs — Recycler- or PlasmidSpades-style approaches requiring an assembly graph fit that case instead
- Contigs are shorter than 1000 bp: 'PlasFlow, similarly to other kmer-based methods, does not perform well on short sequences'
- The goal is identifying plasmid replicons by database similarity (PlasmidFinder) rather than signature-based whole-sequence classification

## Inputs

- SPAdes 3.9.1 assembly FASTA files (scaffolds/contigs) of the 42 plasmid-rich bacterial genomes (Supplemental Table S2)
- GenBank reference plasmid sequences for each of the 42 genomes (ground truth)
- Filtered FASTA of contigs longer than 1000 bp (via filter_sequences_by_length.pl)

## Outputs

- PlasFlow predictions TSV per genome (contig_id, contig_name, contig_length, id, label, per-class probabilities)
- Binned FASTA files: plasmids, chromosomes, and unclassified sequences
- Plasmid recall and precision metrics (e.g. PlasFlow 85.98% recall / 72.17% precision; cBar 76.82% / 60.51%; Recycler 12.0% / 30.00%)

## How to apply

Obtain SPAdes 3.9.1 assemblies and filter contigs with filter_sequences_by_length.pl using a 1000 bp threshold, since k-mer-based prediction is unreliable on short sequences. Run PlasFlow.py on each filtered FASTA with --threshold 0.7 so low-probability classifications are excluded, classifying contigs into chromosome/plasmid classes by phylum and binning results into plasmids, chromosomes and unclassified FASTA files. Collect the tab-separated predictions (contig_id, contig_name, contig_length, id, label, per-class probabilities). Compare predicted plasmid contigs to GenBank reference plasmids to compute recall and precision; PlasFlow targets are 85.98% recall / 72.17% precision on the 42 genomes, versus cBar (76.82% / 60.51%) and Recycler (12.0% / 30.00%).

## Related tools

- **PlasFlow** (Neural-network classifier that labels contigs as plasmid/chromosome by phylum and bins them into FASTA outputs; invoked with --threshold 0.7) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl/Bioperl script used before classification to retain only contigs longer than 1000 bp) — https://github.com/smaegol/PlasFlow
- **cBar** (SOM-based plasmid-prediction baseline used for recall/precision comparison)
- **Recycler** (Assembly-graph circular-contig recovery tool used as a comparison baseline) — https://github.com/Shamir-Lab/Recycler

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000; PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Reproduced plasmid recall on the 42-genome benchmark is ~85.98% for PlasFlow (precision ~72.17%)
- Baseline comparison must rank PlasFlow above cBar (76.82% recall) and Recycler (12.0% recall)
- Every prediction file is a tab-separated table containing contig_id, contig_name, contig_length, id, label, and per-class probabilities
- Output directory contains three FASTA bins: plasmids, chromosomes, and unclassified
- No contig shorter than 1000 bp appears in the input to PlasFlow.py (post-filter check)

## Limitations

- PlasFlow does not perform well on short sequences (k-mer coverage insufficient), so filtering at 1000 bp is required and short contigs remain unclassified
- The model requires regular updating with newly published genomic data to keep covering taxonomic diversity
- Probability thresholding excludes uncertain predictions, so some true plasmids are discarded as unclassified
- Recycler and PlasmidSpades require an assembly graph and paired reads, so the comparison is not on identical input representations

## Evidence

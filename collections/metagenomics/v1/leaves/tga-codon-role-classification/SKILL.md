---
name: tga-codon-role-classification
description: Use when annotating human microbiome metagenomic contigs of unknown provenance
  where the genetic code is unknown, and correct interpretation of TGA (stop vs. sense/tryptophan)
  is needed before or during gene prediction.
when_to_use_negative:
- The input sequences come from organisms with a known standard genetic code already
  recorded (e.g. complete GenBank genomes) — use GeneMarkS-2 with the known code instead
  of classifying TGA de novo.
- The goal is protein-level functional annotation or taxonomic binning rather than
  gene prediction in contigs with ambiguous genetic code.
- Prerequisites (gmhmmp2 key, blastn in PATH, refmod model library, pan-genome BLAST
  database) cannot be installed, since the TGA-role determination depends on the full
  GeneMark-HM model-selection machinery.
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0622
tools:
- name: GeneMark-HM
  role: Pipeline whose TGA-role determination step runs the dual MetaGeneMark classifier
    on each anonymous contig to decide if TGA is stop or sense
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Provides the dual (stop-TGA vs. sense-TGA) GC-content-specific models used
    to classify the TGA role in anonymous contigs
- name: blastn
  role: Similarity search (blastn/MegaBLAST) of predicted genes against the pan-genome
    reference database to verify codon-role assignments
- name: makeblastdb
  role: Builds the nucleotide BLAST database from the ~20 Gb pan.fasta pan-genome
    file prior to model selection
- name: compp
  role: Comparison script that scores output.gtf against reference genemark.gtf to
    quantify TGA-role identification accuracy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:08a75a1761966c0aca453c151ab04b6c85cedc8c7381f642829e556d8ec4090d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GeneProbe/GeneMark-HM
  repo_url: https://github.com/GeneProbe/GeneMark-HM
  tools:
  - GeneMark-HM
  - MetaGeneMark
  - blastn
  - compp
  - makeblastdb
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tga-codon-role-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify whether TGA acts as a stop codon or a sense codon in anonymous metagenomic contigs using GeneMark-HM's dual MetaGeneMark classifier, which runs each contig with models for the two possible TGA roles and selects the more likely one. This matters because in some bacterial genomes TGA codes for tryptophan, and misassigning its role corrupts downstream gene prediction.

## When to use

Use when annotating human microbiome metagenomic contigs of unknown provenance where the genetic code is unknown, and correct interpretation of TGA (stop vs. sense/tryptophan) is needed before or during gene prediction.

## When NOT to use

- The input sequences come from organisms with a known standard genetic code already recorded (e.g. complete GenBank genomes) — use GeneMarkS-2 with the known code instead of classifying TGA de novo.
- The goal is protein-level functional annotation or taxonomic binning rather than gene prediction in contigs with ambiguous genetic code.
- Prerequisites (gmhmmp2 key, blastn in PATH, refmod model library, pan-genome BLAST database) cannot be installed, since the TGA-role determination depends on the full GeneMark-HM model-selection machinery.

## Inputs

- Anonymous metagenomic contigs in FASTA format (e.g. sequence.fasta)
- refmod.tar library of pre-computed GeneMark model files
- pan.fasta pan-genome sequence database (~20 Gb, human microbiome species-level pan-genomes)
- Pipeline license key placed at ~/.gmhmmp2_key
- Optional reference annotation (genemark.gtf) for accuracy comparison

## Outputs

- output.gtf with predicted genes annotated per contig, reflecting the determined TGA role
- Per-contig genetic-code / TGA-role assignments (stop-codon vs. sense-codon, e.g. genetic code 4)
- Verbose STDOUT log reporting performance of individual pipeline steps
- compp comparison statistics quantifying identification accuracy against reference annotation

## How to apply

Install the GeneMark-HM pipeline key at ~/.gmhmmp2_key and verify prerequisites (Perl 5.10 with Parallel::ForkManager, blastn in PATH); unpack the refmod.tar model library into bin and build a nucleotide BLAST database from the ~20 Gb pan.fasta pan-genome with makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan. Run the gm_hm.pl driver on the test contigs (--seq ../sequence.fasta --cores 4 --out output.gtf --clean) so the pipeline's TGA-role determination step runs the dual MetaGeneMark classifier — one model treating TGA as stop, one as sense — on each contig and selects the better-scoring role. Rationale: genes initially predicted in each contig are used for fast similarity search against the pan-genome gene database to pick near-optimal species-specific models, which also constrains the genetic-code variant; contigs not matched to a pan-genome fall back to GC-content-specific models that still distinguish TGA roles. Verify assignments by similarity search of predicted genes against the reference database with blastn/MegaBLAST, and compare output.gtf against a reference genemark.gtf with the compp script. Accuracy is judged against held-out fragments with known codes: the paper reports 99.9% correct TGA-role identification, no errors on contigs with more than 100 genes, and 18 of 4930 rGenomes using genetic code 4 (sense TGA).

## Related tools

- **GeneMark-HM** (Pipeline whose TGA-role determination step runs the dual MetaGeneMark classifier on each anonymous contig to decide if TGA is stop or sense) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Provides the dual (stop-TGA vs. sense-TGA) GC-content-specific models used to classify the TGA role in anonymous contigs)
- **blastn** (Similarity search (blastn/MegaBLAST) of predicted genes against the pan-genome reference database to verify codon-role assignments)
- **makeblastdb** (Builds the nucleotide BLAST database from the ~20 Gb pan.fasta pan-genome file prior to model selection)
- **compp** (Comparison script that scores output.gtf against reference genemark.gtf to quantify TGA-role identification accuracy)

## Examples

```
nohup bin/gm_hm.pl --seq ../sequence.fasta --cores 4 --out output.gtf --clean > logfile &
```

## Evaluation signals

- On held-out 10,000 nt fragments with known genetic codes, the classifier reaches ~99.9% correct TGA-role identification; deviations much below this indicate misconfiguration.
- The count of rGenomes inferred to use genetic code 4 (sense TGA) should match the reported 18 of 4930.
- GeneMark-HM should make no TGA-role errors on contigs containing more than 100 genes.
- Each contig in output.gtf should be annotated consistently with a single TGA role (no mixed stop/sense assignments within one contig).
- compp comparison of output.gtf vs. reference genemark.gtf should show few or zero discrepancies attributable to TGA interpretation.

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome; unmatched contigs rely on cruder GC-content-specific models.
- Role determination is less reliable on short contigs with few genes — errors were only absent for contigs with more than 100 genes.
- Oral and skin microbiome contigs have lower pan-genome assignment coverage (89.0% and 79.4% vs. 99.4% for gut), which can weaken TGA-role calls in those environments.
- The pan-genome database download is ~20 Gb and pipeline setup requires a license key plus Perl Parallel::ForkManager and NCBI blastn in PATH.

## Evidence

- [readme] README run command for the pipeline: [evidence span withheld — anchor status: external_doc]

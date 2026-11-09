---
name: translation-initiation-signal-classification
description: Use when benchmarking gene start prediction tools (e.g., GeneMark-HM
  vs MetaProdigal) on complete genomes and you need to stratify results by translation
  initiation signal type — canonical RBS, non-canonical RBS, or leaderless.
when_to_use_negative:
- Input is metagenomic contigs rather than complete genomes — the classes are defined
  by GeneMarkS-2 self-training, which requires sufficiently long sequences (complete
  genomes)
- You only need overall (unstratified) gene start accuracy — the class partition adds
  value only when initiation-signal-specific behavior matters
- No reference annotation with GeneMarkS-2-identified initiation signals is available,
  since class assignment depends on signals upstream of each annotated start
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: GeneMarkS-2
  role: Self-training gene finder for complete genomes whose annotated starts define
    the upstream translation initiation signals used to assign genomes to canonical
    RBS / non-canonical RBS / leaderless classes
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark-HM
  role: Gene prediction pipeline whose predicted gene starts (output.gtf) are benchmarked
    per initiation-signal class against GeneMarkS-2 reference annotations
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaProdigal
  role: Runner-up gene prediction tool whose per-class start error counts provide
    the baseline for computing GeneMark-HM's error-rate decrease
- name: compp
  role: Comparison utility used to count gene start prediction errors of both tools
    against GeneMarkS-2 reference annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:f3178de72c2bfc358b4387dc58e80dbe679ab71eaae251b4350ae290606b4595
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
  - GeneMark.hmm-2
  - GeneMarkS-2
  - MetaGeneMark
  - MetaProdigal
  - compp
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# translation-initiation-signal-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition complete prokaryotic genomes into classes defined by the translation initiation signal upstream of each annotated start codon — canonical RBS, non-canonical RBS, or leaderless transcription — to enable stratified benchmarking of gene start prediction accuracy (as done in the GeneMark-HM paper's Table 4 analysis of 625 test genomes grouped by GeneMarkS-2).

## When to use

Use when benchmarking gene start prediction tools (e.g., GeneMark-HM vs MetaProdigal) on complete genomes and you need to stratify results by translation initiation signal type — canonical RBS, non-canonical RBS, or leaderless.

## When NOT to use

- Input is metagenomic contigs rather than complete genomes — the classes are defined by GeneMarkS-2 self-training, which requires sufficiently long sequences (complete genomes)
- You only need overall (unstratified) gene start accuracy — the class partition adds value only when initiation-signal-specific behavior matters
- No reference annotation with GeneMarkS-2-identified initiation signals is available, since class assignment depends on signals upstream of each annotated start

## Inputs

- 625 complete test genome sequences (FASTA) with GeneMarkS-2 reference gene annotations including start coordinates
- Gene start predictions in GTF format from GeneMark-HM (output.gtf per genome)
- Gene start predictions in GTF format from MetaProdigal
- compp comparison utility for start-coordinate agreement counting

## Outputs

- Per-genome translation initiation signal class assignment (canonical RBS, non-canonical RBS, leaderless)
- Per-class gene start error counts for each tool, written as table4_gene_start_errors.csv mirroring the layout of Table 4
- Per-class error-rate decrease of GeneMark-HM relative to MetaProdigal

## How to apply

Take the 625 test genome sequences with reference annotations from GeneMarkS-2, which self-trains on complete genomes and identifies the translation initiation signal found upstream of each annotated start. Assign each genome to one of three classes: canonical RBS (Shine-Dalgarno motif upstream of the start), non-canonical RBS, or frequent leaderless transcription initiation (start codon at/near the transcript 5' end). Run each gene prediction tool (GeneMark-HM via bin/gm_hm.pl with --cores 4, and MetaProdigal) on all genomes, then compare predicted start coordinates against the GeneMarkS-2 reference with the compp comparison utility. Aggregate per-class gene start error counts (table4_gene_start_errors.csv) and compute the error-rate decrease of GeneMark-HM relative to the runner-up MetaProdigal within each class; the paper found this decrease was smallest for canonical RBS genomes and largest for the leaderless (group C) genomes.

## Related tools

- **GeneMarkS-2** (Self-training gene finder for complete genomes whose annotated starts define the upstream translation initiation signals used to assign genomes to canonical RBS / non-canonical RBS / leaderless classes) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark-HM** (Gene prediction pipeline whose predicted gene starts (output.gtf) are benchmarked per initiation-signal class against GeneMarkS-2 reference annotations) — https://github.com/GeneProbe/GeneMark-HM
- **MetaProdigal** (Runner-up gene prediction tool whose per-class start error counts provide the baseline for computing GeneMark-HM's error-rate decrease)
- **compp** (Comparison utility used to count gene start prediction errors of both tools against GeneMarkS-2 reference annotations)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq genome.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Reproduced per-class error counts must match the error counts reported in Table 4 for GeneMark-HM vs MetaProdigal
- The GeneMark-HM error-rate decrease relative to MetaProdigal must be largest in the leaderless translation initiation group (group C) and smallest for genomes with canonical RBS
- Every one of the 625 test genomes must be assigned to exactly one of the three classes, with no unclassified genomes
- GTF outputs from both tools must parse cleanly and cover the same set of genomes before compp comparison

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome; misassignment would degrade the start predictions being stratified
- Class definitions rely on GeneMarkS-2 self-training on complete genomes; the signal detection is not validated independently
- Results depend on external tables (Table 4) and per-group counts; the manuscript section alone does not contain the numeric thresholds or raw counts

## Evidence

- [full_text] The difference in gene start prediction errors between GeneMark-HM and the runner-up MetaProdigal was smallest for genomes with canonical RBS: [evidence span withheld — anchor status: not_found]
- [readme] nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &: [evidence span withheld — anchor status: external_doc]

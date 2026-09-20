---
name: gene-start-annotation-benchmarking
description: Use when evaluating gene start prediction accuracy of metagenomic gene
  finders on complete genomes and you need error counts stratified by the translation
  initiation signal (canonical RBS, non-canonical RBS, leaderless) identified upstream
  of reference starts.
when_to_use_negative:
- Input is raw metagenomic contig data rather than complete test genomes with reference
  annotations — use the GeneMark-HM annotation pipeline skill instead of this benchmarking
  skill
- The goal is 3' gene end (stop codon) accuracy or TGA reassignment benchmarking (Tables
  2-3) rather than gene start error comparison (Table 4)
- Reference annotations were produced by a self-training tool other than GeneMarkS-2,
  so the paper's error-rate comparisons would not be reproducible
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: GeneMark-HM
  role: Pipeline under benchmark; predicts gene starts using GeneMark.hmm-2 models
    selected via pan-genome assignment
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMarkS-2
  role: Self-training algorithm that produces the reference gene start annotations
    on the complete test genomes and classifies their initiation signals
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaProdigal
  role: Runner-up gene finder benchmarked against GeneMark-HM on gene start prediction
    errors
- name: MetaGeneMark
  role: GC-content-specific gene prediction tool referenced as prior state-of-the-art
    baseline
- name: NCBI blastn
  role: Similarity search of initially predicted genes against the pan-genome database
    to select GeneMark.hmm-2 models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:b4aa3597aafd2dcea5c4fd7a7654073f6ac69b941c4f5f8518a3cb43cc4da9ae
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
  - NCBI blastn
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-start-annotation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that compares predicted gene start coordinates from metagenomic gene finders (GeneMark-HM vs MetaProdigal) against GeneMarkS-2 reference annotations, stratifying genomes by translation initiation signal class (canonical RBS, non-canonical RBS, leaderless) to quantify where model selection improves start prediction.

## When to use

Use when evaluating gene start prediction accuracy of metagenomic gene finders on complete genomes and you need error counts stratified by the translation initiation signal (canonical RBS, non-canonical RBS, leaderless) identified upstream of reference starts.

## When NOT to use

- Input is raw metagenomic contig data rather than complete test genomes with reference annotations — use the GeneMark-HM annotation pipeline skill instead of this benchmarking skill
- The goal is 3' gene end (stop codon) accuracy or TGA reassignment benchmarking (Tables 2-3) rather than gene start error comparison (Table 4)
- Reference annotations were produced by a self-training tool other than GeneMarkS-2, so the paper's error-rate comparisons would not be reproducible

## Inputs

- 625 test genome sequences (FASTA) with GeneMarkS-2 self-training annotations of gene starts
- Grouping of genomes by upstream translation initiation signal: canonical RBS, non-canonical RBS, frequent leaderless transcription
- ~20 Gb pan-genome sequences database for the human microbiome (Amazon S3)
- refmod database of pan-genome specific gene finding model files

## Outputs

- output.gtf per genome from GeneMark-HM (gm_hm.pl)
- GTF gene start predictions per genome from MetaProdigal
- Per-group gene start error counts from the compp comparison utility
- table4_gene_start_errors.csv mirroring the layout of Table 4

## How to apply

Partition the 625 test genomes into three groups by the translation initiation signal found upstream of each GeneMarkS-2 annotated start: canonical RBS, non-canonical RBS, and leaderless. Run the GeneMark-HM pipeline (bin/gm_hm.pl with --cores 4, --seq, --aa, --nt, --out output.gtf) on each genome; its predicted genes serve as blastn queries against the ~20 Gb pan-genome database (built with makeblastdb -parse_seqids -dbtype nucl) to select GeneMark.hmm-2 models. Run MetaProdigal on the same genomes to obtain its GTF gene start predictions. Compare both tools' predicted start coordinates against the GeneMarkS-2 reference annotations using the compp comparison utility, aggregate per-group error counts, and write table4_gene_start_errors.csv mirroring the layout of Table 4. The rationale is that GeneMark-HM's automatic selection of a translation start site sequence model after pan-genome assignment should reduce start errors most where initiation signal modeling matters most — the leaderless group.

## Related tools

- **GeneMark-HM** (Pipeline under benchmark; predicts gene starts using GeneMark.hmm-2 models selected via pan-genome assignment) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMarkS-2** (Self-training algorithm that produces the reference gene start annotations on the complete test genomes and classifies their initiation signals) — https://github.com/GeneProbe/GeneMark-HM
- **MetaProdigal** (Runner-up gene finder benchmarked against GeneMark-HM on gene start prediction errors)
- **MetaGeneMark** (GC-content-specific gene prediction tool referenced as prior state-of-the-art baseline)
- **NCBI blastn** (Similarity search of initially predicted genes against the pan-genome database to select GeneMark.hmm-2 models)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq genome.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Reproduced per-group error counts match the error counts reported in Table 4 for GeneMark-HM vs MetaProdigal
- The GeneMark-HM error-rate decrease relative to MetaProdigal is smallest in the canonical RBS group and largest in the leaderless translation initiation group (group C)
- Each of the 625 test genomes is assigned to exactly one of the three initiation signal classes and every genome yields an output.gtf from both tools
- compp comparison output covers all annotated starts per genome with no missing coordinates before aggregation into table4_gene_start_errors.csv

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome; misassignment degrades start prediction
- No version numbers are stated for GeneMark-HM, GeneMark.hmm-2, GeneMarkS-2, MetaGeneMark, or MetaProdigal, complicating exact reproduction
- Results depend on external comparison tables (e.g., Table 4) not contained in the repository text itself

## Evidence

- [results] The difference in gene start prediction errors between GeneMark-HM and the runner-up MetaProdigal was smallest for genomes with canonical RBS and largest for genomes with frequent leaderless translation initiation.: [evidence span withheld — anchor status: not_found]
- [readme] nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &: [evidence span withheld — anchor status: external_doc]

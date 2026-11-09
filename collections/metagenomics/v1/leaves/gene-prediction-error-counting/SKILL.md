---
name: gene-prediction-error-counting
description: Use when benchmarking a new gene prediction tool's start-codon accuracy
  against reference GeneMarkS-2 annotations and you need per-group error counts split
  by regulatory signal class (canonical RBS, non-canonical RBS, leaderless).
when_to_use_negative:
- Your goal is de novo gene prediction on a novel metagenome rather than accuracy
  benchmarking against reference annotations
- Your test genomes lack GeneMarkS-2 reference start annotations, so no error counting
  against a trusted standard is possible
- Your comparison targets gene 3' end accuracy or TGA codon role determination (Tables
  2-3 comparisons) rather than start-position error stratification
edam_operation: http://edamontology.org/operation_0362
tools:
- name: GeneMark-HM
  role: Primary predictor whose gene start calls are being evaluated; run via bin/gm_hm.pl
    to generate output.gtf per test genome
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaProdigal
  role: Runner-up gene predictor used as the baseline whose start errors are compared
    against GeneMark-HM per signal class
- name: GeneMarkS-2
  role: Self-training algorithm producing the reference start annotations on complete
    test genomes used as ground truth for error counting
- name: GeneMark.hmm-2
  role: Applied inside the pipeline with pan-genome-selected models after blastn matching
    of initial gene predictions
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI blastn
  role: Queries initially predicted genes against the pan-genome database to select
    GeneMark.hmm-2 models before prediction
- name: compp
  role: Comparison utility that scores predicted gene start coordinates against GeneMarkS-2
    reference annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:2d96dada2bddcade1259b68b83cca326e9169b370aac09e52785e6a87935f68b
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
  - compp
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-prediction-error-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies gene start prediction errors of GeneMark-HM versus runner-up MetaProdigal against GeneMarkS-2 reference annotations, stratified over 625 test genomes grouped by translation initiation signal class (canonical RBS, non-canonical RBS, frequent leaderless transcription), reproducing the paper's Table 4 comparison.

## When to use

Use when benchmarking a new gene prediction tool's start-codon accuracy against reference GeneMarkS-2 annotations and you need per-group error counts split by regulatory signal class (canonical RBS, non-canonical RBS, leaderless).

## When NOT to use

- Your goal is de novo gene prediction on a novel metagenome rather than accuracy benchmarking against reference annotations
- Your test genomes lack GeneMarkS-2 reference start annotations, so no error counting against a trusted standard is possible
- Your comparison targets gene 3' end accuracy or TGA codon role determination (Tables 2-3 comparisons) rather than start-position error stratification

## Inputs

- 625 test genome sequences in FASTA format
- GeneMarkS-2 reference annotations (GTF/GFF) with start coordinates and upstream initiation signal classes
- ~20 Gb pan-genome sequence database (S3-hosted pan files)
- Species-specific gene finding model files (refmod database)
- MetaProdigal GTF predictions for the same 625 genomes

## Outputs

- output.gtf with GeneMark-HM predicted gene starts per genome
- table4_gene_start_errors.csv with per-group gene start error counts for GeneMark-HM vs MetaProdigal
- Validated statement of which signal class shows the largest GeneMark-HM advantage

## How to apply

Partition the 625 test genomes into three classes based on the translation initiation signal found upstream of each GeneMarkS-2 annotated start: canonical RBS, non-canonical RBS, and frequent leaderless transcription. Run the GeneMark-HM pipeline (bin/gm_hm.pl with --cores 4), letting initially predicted genes serve as blastn queries against the ~20 Gb pan-genome database (makeblastdb -parse_seqids -dbtype nucl) to select GeneMark.hmm-2 models, producing output.gtf per genome. Run MetaProdigal on the same genomes to obtain its GTF gene start predictions. Compare both tools' predicted start coordinates against GeneMarkS-2 reference annotations using the compp comparison utility. Aggregate per-group gene start error counts into table4_gene_start_errors.csv mirroring Table 4, and check that the GeneMark-HM error-rate decrease relative to MetaProdigal is smallest in the canonical RBS group and largest in the leaderless translation initiation group.

## Related tools

- **GeneMark-HM** (Primary predictor whose gene start calls are being evaluated; run via bin/gm_hm.pl to generate output.gtf per test genome) — https://github.com/GeneProbe/GeneMark-HM
- **MetaProdigal** (Runner-up gene predictor used as the baseline whose start errors are compared against GeneMark-HM per signal class)
- **GeneMarkS-2** (Self-training algorithm producing the reference start annotations on complete test genomes used as ground truth for error counting)
- **GeneMark.hmm-2** (Applied inside the pipeline with pan-genome-selected models after blastn matching of initial gene predictions) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI blastn** (Queries initially predicted genes against the pan-genome database to select GeneMark.hmm-2 models before prediction)
- **compp** (Comparison utility that scores predicted gene start coordinates against GeneMarkS-2 reference annotations)

## Examples

```
mkdir work_dir && cd work_dir && nohup path_to/bin/gm_hm.pl --seq genome.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Reproduced error counts in table4_gene_start_errors.csv must match the counts reported in Table 4 for GeneMark-HM vs MetaProdigal
- The GeneMark-HM error-rate decrease relative to MetaProdigal must be smallest in the canonical RBS group
- The same decrease must be largest in the frequent leaderless translation initiation group (group C)
- Every one of the 625 genomes must fall into exactly one of the three signal classes (canonical RBS, non-canonical RBS, leaderless) with no unclassified genomes
- output.gtf must be produced for all test genomes with pan-genome-assigned models; unmatched contigs should be handled by the MetaGeneMark/GeneMarkS-2 length-based fallback

## Limitations

- GeneMark-HM accuracy depends on correct assignment of metagenomic contigs to reference pan-genomes; misassignment degrades start predictions and thus error counts
- Results depend on external tables (e.g. Table 4) not contained in the article text; exact numeric counts must be recovered from the paper or GeneMark-HM-exp repository
- No version numbers are stated for GeneMark-HM, GeneMark.hmm-2, GeneMarkS-2, MetaGeneMark, or MetaProdigal, so exact reproducibility of counts may vary across software versions
- Download of the ~20 Gb pan-genome database can take considerable time, and no accession location beyond Amazon S3 is given

## Evidence

- [readme] Pipeline is run per genome with gm_hm.pl, --cores 4, producing output.gtf: [evidence span withheld — anchor status: external_doc]

---
name: gene-prediction-open-reading-frame-calling
description: Use when assembled contigs or long-read sequences must be annotated with
  predicted ORFs before exclusion-based viral classification, e.g., after SPAdes/metaSPAdes/MEGAHIT
  assembly or when loading contigs via virMine's -A option.
when_to_use_negative:
- Input is still raw short-read FASTQ — gene prediction is applied after read QC and
  assembly, not before
- The goal is direct homology-only identification of known viruses without gene-content
  scoring — virMine's exclusion technique explicitly removes this sole dependency
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: virMine 2.0
  role: Executes the gene prediction step on assembled or supplied contigs and hands
    ORFs to BLAST-based scoring
  repo: https://github.com/putonti/virmine
- name: Python 3.9
  role: Runtime for the virMine.py pipeline entry point that orchestrates ORF calling
  repo: https://github.com/putonti/virmine
- name: BLAST
  role: Searches predicted ORF proteins against viral (-v) and nonviral (-nv) databases;
    selectable as blastp or blastx via -b
- name: Docker
  role: Container runtime that resolves all gene-prediction and BLAST dependencies
    via the Dockerfile
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:f0c3d07a0c1b1aed53886d3108ca13296c239824368ff211485efc91bf650582
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BLAST
  - Docker
  - Python
  - Python 3.9
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Gene prediction / open reading frame calling

## Summary

Predict open reading frames on each assembled contig (or supplied long-read/contig assembly) as the annotation step of virMine 2.0, producing the ORF protein set that BLAST-based viral scoring depends on.

## When to use

Use when assembled contigs or long-read sequences must be annotated with predicted ORFs before exclusion-based viral classification, e.g., after SPAdes/metaSPAdes/MEGAHIT assembly or when loading contigs via virMine's -A option.

## When NOT to use

- Input is still raw short-read FASTQ — gene prediction is applied after read QC and assembly, not before
- The goal is direct homology-only identification of known viruses without gene-content scoring — virMine's exclusion technique explicitly removes this sole dependency

## Inputs

- Assembled contigs FASTA (e.g., final_contigs.fasta from SPAdes/metaSPAdes/MEGAHIT assembly)
- User-supplied long-read or assembled-sequence FASTA passed via virMine's -A option
- Viral protein database FASTA (-v, e.g., inputFiles/viral_aa.fasta)
- Nonviral protein database FASTA (-nv, e.g., inputFiles/nonviral_aa.fasta)

## Outputs

- All predicted ORFs file (open reading frames for every contig)
- Viral ORFs only file (subset used for downstream scoring)
- BLAST output files (blastp or blastx hits of ORFs against viral and nonviral databases)

## How to apply

Run gene prediction on every contig in the assembly — either contigs assembled by virMine (SPAdes, metaSPAdes, or MEGAHIT) or those supplied via the -A flag (e.g., inputFiles/assembled_contigs.fasta). Select BLAST flavor with -b ('blastp' or 'blastx'; blastp is faster) so the predicted ORF proteins can subsequently be searched against the user-supplied viral (-v) and nonviral (-nv) databases. The predicted ORFs are the substrate for the exclusion technique: genes are classified as cellular, viral, or unknown based on BLAST hits, and contigs are scored on their gene content's origin. Retain the predicted ORF output files alongside the BLAST outputs so scoring is reproducible. Validate on the article's test data (E. coli K12 NC_000913.3 + Escherichia phage Lambda J02459.1 artificial reads), where 3 contigs in final_contigs.fasta should yield 1 viral contig.

## Related tools

- **virMine 2.0** (Executes the gene prediction step on assembled or supplied contigs and hands ORFs to BLAST-based scoring) — https://github.com/putonti/virmine
- **Python 3.9** (Runtime for the virMine.py pipeline entry point that orchestrates ORF calling) — https://github.com/putonti/virmine
- **BLAST** (Searches predicted ORF proteins against viral (-v) and nonviral (-nv) databases; selectable as blastp or blastx via -b)
- **Docker** (Container runtime that resolves all gene-prediction and BLAST dependencies via the Dockerfile) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -b blastp -o output
```

## Evaluation signals

- ORF output files exist for every contig in final_contigs.fasta and are retained alongside BLAST outputs
- A viral-ORFs-only file is produced as a strict subset of the all-predicted-ORFs file
- BLAST output files contain hits of predicted ORFs against both -v and -nv databases, confirming the exclusion technique consumed the ORFs
- On the article's test data (E. coli K12 + Escherichia phage Lambda reads, all3 assembly), 3 contigs result and 1 is classified as viral in viral_contigs.fasta
- Running with -b blastp vs -b blastx yields consistent contig-level viral classification (blastp is the faster option)

## Limitations

- If any section of the program errors or will not run, check that enough memory is allocated to Docker resources
- Contig quality entering gene prediction depends on the chosen assembler (spades, metaspades, megahit) and optional filters (-m, -M, -c, -cov); short contigs or low-coverage contigs can be removed before ORF calling

## Evidence

- [abstract] Next, virMine preforms gene prediction.: "Next, virMine preforms gene prediction."
- [intro] assembled short-reads or supplied long-reads/contigs are then annotated: "These assembled short-reads or supplied long-reads/contigs are then annotated, and individual contigs are scored based upon their likelihood of being viral in origin."
- [readme] Other output files include all predicted ORFs: "Other output files include all predicted ORFs, the viral ORFs only, and all BLAST output files."
- [methods] test data yields 3 contigs, 1 viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"

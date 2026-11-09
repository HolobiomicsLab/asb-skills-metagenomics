---
name: nucleotide-similarity-search
description: Use when annotating metagenomic contigs that are too short for self-training
  gene finding, and sequenced genomes of close relatives (species-level pan-genomes)
  are available to supply protein-coding region parameters via nucleotide similarity
  assignment.
when_to_use_negative:
- Contigs are long enough for reliable self-training with GeneMarkS-2, or belong to
  taxa with no available reference genomes (no pan-genome assignment possible)
- Input is raw reads rather than assembled contigs; the pipeline expects metagenome
  assemblies
- The goal is de novo model estimation on complete genomes, which GeneMarkS-2 self-training
  handles directly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: GeneMark-HM
  role: Pipeline (gm_hm.pl) that wraps the nucleotide similarity search for pan-genome
    assignment and subsequent model-based gene prediction
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: makeblastdb
  role: Indexes the ~20 Gb pan-genome nucleotide database (pan.fasta) with -parse_seqids
    -dbtype nucl for the similarity search
- name: blastn
  role: Provides the fast nucleotide similarity search of initially predicted gene
    queries against the pan-genome database
- name: GeneMark.hmm-2
  role: Predicts genes in each contig using the model selected via pan-genome similarity
    assignment
- name: GeneMarkS-2
  role: Generates native self-training reference annotations (GTF) on complete rGenomes
    to validate pan-genome-selected model accuracy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:41071d09991707aa9a75923c3ddb2b6bec9e3a84343ac6a210b34de0845cfc7f
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
  - blastn
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nucleotide-similarity-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A similarity-search step at the core of the GeneMark-HM pipeline: genes initially predicted in each metagenomic contig are used as queries against a pan-genome nucleotide database (indexed with NCBI makeblastdb) to assign the contig to a species-level pan-genome and select a near-optimal gene prediction model. It is also used to validate that pan-genome-selected models reproduce native GeneMarkS-2 annotations across rGenomes of 1849 SGBs.

## When to use

Use when annotating metagenomic contigs that are too short for self-training gene finding, and sequenced genomes of close relatives (species-level pan-genomes) are available to supply protein-coding region parameters via nucleotide similarity assignment.

## When NOT to use

- Contigs are long enough for reliable self-training with GeneMarkS-2, or belong to taxa with no available reference genomes (no pan-genome assignment possible)
- Input is raw reads rather than assembled contigs; the pipeline expects metagenome assemblies
- The goal is de novo model estimation on complete genomes, which GeneMarkS-2 self-training handles directly

## Inputs

- Metagenome assembly contigs in FASTA format
- ~20 Gb pan-genome sequence file (pan.fasta) from the human microbiome pan-genome database (Amazon S3)
- refmod.tar database of pan-genome-specific gene finding model files
- rGenome SGB sequence files (multi-rGenome FASTA) for validation against native GeneMarkS-2 annotations

## Outputs

- Contig-to-pan-genome assignments selecting species-specific parameters for GeneMark.hmm-2
- Gene predictions in GTF format (--out output.gtf), plus protein (--aa) and nucleotide (--nt) FASTA outputs
- Per-rGenome nucleotide-level sensitivity (Sn) and specificity (Sp) values when validating SGB representative models

## How to apply

Index the ~20 Gb human-microbiome pan-genome sequence file (pan.fasta) with NCBI makeblastdb using -parse_seqids -dbtype nucl. In the GeneMark-HM pipeline (gm_hm.pl), genes initially predicted in each contig serve as queries for a fast nucleotide similarity search through the pan-genome database; the best matches trigger assignment of the contig to a pan-genome and automatic selection of a suitable translation-start-site model for GeneMark.hmm-2. Contigs with no reliable pan-genome assignment fall back to GeneMarkS-2 (longer contigs) or MetaGeneMark (shorter contigs). As validation, run gm_hm.pl with the SGB representative model on rGenome sequences, run GeneMarkS-2 in self-training mode on the same sequences for reference GTFs, and compare the two GTF annotations nucleotide-by-nucleotide (e.g. with the compp script) to compute sensitivity (Sn) and specificity (Sp) per rGenome.

## Related tools

- **GeneMark-HM** (Pipeline (gm_hm.pl) that wraps the nucleotide similarity search for pan-genome assignment and subsequent model-based gene prediction) — https://github.com/GeneProbe/GeneMark-HM
- **makeblastdb** (Indexes the ~20 Gb pan-genome nucleotide database (pan.fasta) with -parse_seqids -dbtype nucl for the similarity search)
- **blastn** (Provides the fast nucleotide similarity search of initially predicted gene queries against the pan-genome database)
- **GeneMark.hmm-2** (Predicts genes in each contig using the model selected via pan-genome similarity assignment)
- **GeneMarkS-2** (Generates native self-training reference annotations (GTF) on complete rGenomes to validate pan-genome-selected model accuracy)

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl; nohup bin/gm_hm.pl --seq sequence.fasta --cores 4 --out output.gtf --clean > logfile &
```

## Evaluation signals

- Nucleotide-level (Sn + Sp)/2 accuracy between GeneMark.hmm-2 predictions with SGB representative models and native GeneMarkS-2 annotations exceeds 98% for most of the 1849 SGB rGenomes
- Pan-genome assignment rate reaches up to 99.4% of contigs for human gut metagenomes (89.0% oral, 79.4% skin as lower bounds)
- Pipeline runs in verbose mode without errors and produces output.gtf plus aa/nt FASTA files; --clean removes all temporary files
- Contigs failing pan-genome assignment are routed to the fallback path (MetaGeneMark for shorter, GeneMarkS-2 for longer contigs) rather than dropped

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome
- Lower accuracy is seen only in highly fragmented rGenomes when validating SGB representative models
- Oral and skin metagenomes have substantially lower pan-genome assignment coverage (89.0% and 79.4%), possibly due to environmental contamination
- The pan-genome sequence file is ~20 Gb; download and database indexing can take significant time and resources

## Evidence

- [methods] Workflow index pan-genome with makeblastdb: [evidence span withheld — anchor status: not_found]
- [results] Accuracy of SGB representative models across 1849 rGenomes: [evidence span withheld — anchor status: not_found]
- [readme] README example invocation of the pipeline: [evidence span withheld — anchor status: external_doc]

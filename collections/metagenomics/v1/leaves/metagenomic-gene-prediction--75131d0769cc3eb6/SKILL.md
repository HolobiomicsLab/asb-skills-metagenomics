---
name: metagenomic-gene-prediction
description: Use when input is a metagenome assembly FASTA (human microbiome contigs)
  needing gene prediction in GTF format, and near-complete-genome accuracy is required
  for short contigs without genomic context.
when_to_use_negative:
- Input is a single complete prokaryotic genome that can be annotated directly with
  the self-training GeneMarkS-2 algorithm — the pan-genome assignment step of GeneMark-HM
  adds no value
- Metagenomes from environments other than the human microbiome (e.g. environmental
  or non-human-host samples), where the human-microbiome pan-genome database has no
  matching taxa and most contigs will fall back to generic models
- Rigorous benchmarking requiring native self-trained per-genome models rather than
  representative pan-genome-derived models, since accuracy depends on correct pan-genome
  assignment
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: GeneMark-HM
  role: Main pipeline orchestrating model selection and gene prediction on metagenome
    contigs
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMarkS-2
  role: Self-training gene predictor used to build pan-genome models and to annotate
    longer unmatched contigs
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark.hmm-2
  role: Runs gene prediction on pan-genome-assigned contigs with externally supplied
    species-specific models
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: GC-content-specific fallback gene predictor for shorter contigs not assigned
    to pan-genomes
- name: NCBI blastn
  role: Similarity search of initially predicted contig genes against the pan-genome
    database to select the model
- name: makeblastdb
  role: Indexes the ~20 Gb pan-genome FASTA (-parse_seqids -dbtype nucl) for blastn
    queries
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:94c21749ab816e7f42168ff0f38f90dfb35c189957231316b0281a6e1d48e236
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
  - NCBI blastn
  - Parallel::ForkManager
  - makeblastdb
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metagenomic-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict protein-coding genes in assembled human microbiome metagenomes with the GeneMark-HM pipeline, which selects species-pan-genome-specific GeneMark.hmm-2 models via blastn against a ~20 Gb pan-genome database and falls back to GeneMarkS-2 (long contigs) or MetaGeneMark (short contigs).

## When to use

Use when input is a metagenome assembly FASTA (human microbiome contigs) needing gene prediction in GTF format, and near-complete-genome accuracy is required for short contigs without genomic context.

## When NOT to use

- Input is a single complete prokaryotic genome that can be annotated directly with the self-training GeneMarkS-2 algorithm — the pan-genome assignment step of GeneMark-HM adds no value
- Metagenomes from environments other than the human microbiome (e.g. environmental or non-human-host samples), where the human-microbiome pan-genome database has no matching taxa and most contigs will fall back to generic models
- Rigorous benchmarking requiring native self-trained per-genome models rather than representative pan-genome-derived models, since accuracy depends on correct pan-genome assignment

## Inputs

- Assembled metagenome contigs in multi-FASTA format (e.g. human gut, oral, or skin assemblies such as GCA_900415335, GCA_003604395)
- Pan-genome FASTA database (~20 Gb) formatted with makeblastdb -parse_seqids -dbtype nucl
- refmod.tar library of pre-computed gene-finding model files
- GeneMark-HM key file (~/.gmhmmp2_key)

## Outputs

- Gene predictions in GTF format (output.gtf)
- Predicted protein sequences (--aa aa.fasta)
- Predicted nucleotide sequences (--nt nt.fasta)
- Pipeline log with per-step performance info in verbose mode

## How to apply

Install the GeneMark-HM pipeline (bin/gm_hm.pl), place the gmhmmp2 key at ~/.gmhmmp2_key, ensure Perl 5.10+ with Parallel::ForkManager and NCBI blastn/makeblastdb in PATH, and download the refmod model-file library plus the ~20 Gb pan-genome FASTA indexed with 'makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan'. For each metagenome assembly run gm_hm.pl, which first predicts genes in each contig with initial models and uses those genes as blastn queries against the pan-genome database to select a species-specific model for GeneMark.hmm-2. Contigs reliably assigned to a pan-genome are annotated with the matched model, including automatic selection of a translation-start-site model; unmatched contigs are processed by GeneMarkS-2 (longer) or MetaGeneMark (shorter) based on contig length, and the role of TGA is determined per contig. Run in a dedicated working directory with --cores set to available resources and optionally --clean to remove temporary files. Validate predictions against PGAP reference annotations using the compp utility, counting missed genes and translation-start errors.

## Related tools

- **GeneMark-HM** (Main pipeline orchestrating model selection and gene prediction on metagenome contigs) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMarkS-2** (Self-training gene predictor used to build pan-genome models and to annotate longer unmatched contigs) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark.hmm-2** (Runs gene prediction on pan-genome-assigned contigs with externally supplied species-specific models) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (GC-content-specific fallback gene predictor for shorter contigs not assigned to pan-genomes)
- **NCBI blastn** (Similarity search of initially predicted contig genes against the pan-genome database to select the model)
- **makeblastdb** (Indexes the ~20 Gb pan-genome FASTA (-parse_seqids -dbtype nucl) for blastn queries)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --clean > logfile &
```

## Evaluation signals

- On simulated metagenomes built from 625 genomes, compp-derived missed-gene counts and gene-start error counts for GeneMark-HM must be lower than FragGeneScan, MetaGeneAnnotator, MetaProdigal, and MetaGeneMark, reproducing Table 3
- Pan-genome assignment fractions must match Table 6: up to 99.4% for human gut, 89.0% for oral, and 79.4% for skin metagenomes
- Nucleotide-level (Sn+Sp)/2 accuracy of GeneMark.hmm-2 with SGB representative models vs native GeneMarkS-2 models should exceed 98% for most of the 1849 SGB rGenomes (Figure 5)
- GTF output must be produced for every input contig, with each contig's predictions traceable to one of the three branches: pan-genome model, GeneMarkS-2 (long), or MetaGeneMark (short)
- Gene start prediction errors should be lowest versus MetaProdigal in the leaderless translation-initiation group (group C) per Table 4

## Limitations

- Accuracy depends on correct assignment of a contig to a reference pan-genome; mis-assignment degrades predictions
- Oral and skin metagenomes have substantially lower pan-genome coverage (89.0% and 79.4%) possibly because these microbiomes are mixed with environmental microbes
- Lower representative-model accuracy is seen in highly fragmented rGenomes
- The ~20 Gb pan-genome download is large and blastn scanning accounts for ~80% of a ~12 min run on a 45 Mb input (vs MetaGeneMark ~10 s and MetaProdigal ~3.5 min)

## Evidence

- [readme] nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose: [evidence span withheld — anchor status: external_doc]
- [readme] Option "--clean" can be specified to remove all the temporary files after code execution.: [evidence span withheld — anchor status: external_doc]

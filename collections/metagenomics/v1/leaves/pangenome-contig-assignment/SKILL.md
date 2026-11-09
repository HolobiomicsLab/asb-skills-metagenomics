---
name: pangenome-contig-assignment
description: Use when predicting protein-coding genes in assembled human microbiome
  metagenome contigs and you need species-level parameter selection via pan-genome
  assignment rather than generic metagenomic models.
when_to_use_negative:
- The metagenome is not from the human microbiome — the pan-genome database was built
  from human gut/oral/skin taxa, so contigs from environmental or non-human sources
  will largely miss assignment and fall back to crude GC-based models
- The contigs are complete or nearly complete genomes — GeneMarkS-2 alone can self-train
  directly and does not need the pan-genome assignment step
- A species-specific model for the organism of interest already exists and can be
  supplied externally
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: GeneMark-HM
  role: Pipeline (gm_hm.pl) orchestrating initial prediction, blastn-based pan-genome
    assignment, model selection, and fallback gene prediction
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Produces the initial gene predictions used as blastn queries, and serves as
    the fallback for shorter unmatched contigs
- name: GeneMarkS-2
  role: Self-trains the species-specific pan-genome models used in the refmod library;
    fallback algorithm for longer unmatched contigs
- name: GeneMark.hmm-2
  role: Runs final gene prediction with the pan-genome-derived model after successful
    contig assignment
- name: NCBI blastn
  role: Similarity search of initially predicted genes against the pan-genome BLAST
    database
- name: Parallel::ForkManager
  role: Perl module enabling parallel processing of contigs across the cores specified
    with --cores
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:f69616d8e056df0ad88e9e8e20ea7db1f64e098989d6205928a2ba87e2228912
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# pangenome-contig-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns metagenomic contigs from the human microbiome to species-level pan-genomes by BLASTing initially predicted genes against a pan-gene database, then selects the corresponding pre-computed GeneMark.hmm-2 model (fallback: GeneMarkS-2 for long contigs, MetaGeneMark for short ones).

## When to use

Use when predicting protein-coding genes in assembled human microbiome metagenome contigs and you need species-level parameter selection via pan-genome assignment rather than generic metagenomic models.

## When NOT to use

- The metagenome is not from the human microbiome — the pan-genome database was built from human gut/oral/skin taxa, so contigs from environmental or non-human sources will largely miss assignment and fall back to crude GC-based models
- The contigs are complete or nearly complete genomes — GeneMarkS-2 alone can self-train directly and does not need the pan-genome assignment step
- A species-specific model for the organism of interest already exists and can be supplied externally

## Inputs

- Metagenome assembly FASTA file of human microbiome contigs (e.g. GCA_900415335, GCA_003604395, GCA_003640265, GCA_013297495)
- Pan-genome sequence database (~20 Gb pan.fasta converted to a BLAST nucleotide database)
- refmod database of pre-computed species-specific model files (refmod.tar from geneprobe.net)
- NCBI blastn executable on PATH (or set via $blastn in gm_hm.pl)

## Outputs

- output.gtf with final gene predictions from GeneMark.hmm-2 (pan-genome path) or GeneMarkS-2/MetaGeneMark fallback branches
- Per-contig branch assignments (pan-genome model vs GeneMarkS-2 vs MetaGeneMark) aggregatable into branch-fraction tables (e.g. table6_model_branch_fractions.tsv)
- Optional protein (--aa aa.fasta) and nucleotide (--nt nt.fasta) gene sequence files

## How to apply

Run the GeneMark-HM entry script gm_hm.pl on the metagenome assembly FASTA, which first produces initial predictions with MetaGeneMark/GeneMarkS-2-class models and uses them as blastn queries against the ~20 Gb pan-genome database (built with makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan). Retain hits passing the filter: E-value < 1e-5, at least 90 nt of aligned query sequence, and average nucleotide identity (ANI) above 95%. Assign each contig to the pan-genome holding the majority of its matched genes, which triggers selection of the corresponding pre-computed model from the refmod model library (derived from GeneMarkS-2 self-training on complete genomes of >5000 human-microbiome taxa) and an automatic selection of a translation-start-site model. Contigs with no reliable assignment fall back to GeneMarkS-2 (longer contigs) or MetaGeneMark (shorter contigs), with a 4-core Parallel::ForkManager parallelization over contigs. Rationale: species-specific parameters usually outperform general metagenomic parameters, and nearly optimal models can be leveraged from sequenced close relatives even for contigs too short for self-training.

## Related tools

- **GeneMark-HM** (Pipeline (gm_hm.pl) orchestrating initial prediction, blastn-based pan-genome assignment, model selection, and fallback gene prediction) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Produces the initial gene predictions used as blastn queries, and serves as the fallback for shorter unmatched contigs)
- **GeneMarkS-2** (Self-trains the species-specific pan-genome models used in the refmod library; fallback algorithm for longer unmatched contigs)
- **GeneMark.hmm-2** (Runs final gene prediction with the pan-genome-derived model after successful contig assignment)
- **NCBI blastn** (Similarity search of initially predicted genes against the pan-genome BLAST database)
- **Parallel::ForkManager** (Perl module enabling parallel processing of contigs across the cores specified with --cores)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Reproduced pan-genome assignment fractions match Table 6: up to 99.4% for human gut, 89.0% for oral, and 79.4% for skin metagenomes
- All retained blastn hits satisfy the filter: E-value < 1e-5, aligned query length > 90 nt, ANI > 95%
- Each contig is assigned to exactly one pan-genome via majority vote of its matched genes; contigs failing assignment are routed to GeneMarkS-2 (longer) or MetaGeneMark (shorter) by contig length
- Output GTF compared against a reference (e.g. compp -a ../genemark.gtf -b output.gtf) shows the reported low false-positive, high-sensitivity behavior
- Run in --verbose mode completes without 'blastn not found' errors (blastn on PATH) and Parallel::ForkManager installed, with --clean removing all temporary files

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a contig to a reference pan-genome; wrong assignment propagates model errors
- Oral and skin metagenomes show substantially lower assignment coverage (89.0% / 79.4%) partly because these microbiomes might be mixed with microbes from the environment
- The length threshold distinguishing 'shorter' (MetaGeneMark) vs 'longer' (GeneMarkS-2) unmatched contigs is not stated in the article text (deferred to Figure 1)

## Evidence

- [readme] Pipeline checks if a contig can be assigned to one of the reference taxa and in case of reliable assignment uses corresponding taxa specific parameters for gene finding in a contig.: [evidence span withheld — anchor status: external_doc]

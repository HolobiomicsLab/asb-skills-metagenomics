---
name: metagenomic-annotation-statistics-aggregation
description: Use when you have GeneMark-HM output on human microbiome metagenome assemblies
  and need to quantify how many contigs were assigned to species-level pan-genomes
  versus handled by the GeneMarkS-2 or MetaGeneMark fallback branches.
when_to_use_negative:
- Input sequences are complete prokaryotic genomes or non-human-microbiome metagenomes;
  the pan-genome database and model selection were built specifically for human microbiome
  contigs
- You need absolute gene-prediction accuracy evaluation against truth labels; this
  skill only quantifies model-branch routing, not prediction accuracy (which requires
  the paper's simulated metagenome benchmarks)
- The pan-genome blastn database has not been built and the branch identity per contig
  cannot be recovered from the run
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: GeneMark-HM
  role: Pipeline entry script gm_hm.pl that performs model selection and produces
    the branch assignments being aggregated
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: NCBI blastn
  role: Similarity search of initial gene predictions against the pan-genome database
    that determines contig-to-pan-genome assignment
- name: GeneMark.hmm-2
  role: Produces final gene predictions for contigs assigned to a pan-genome with
    pan-genome-derived models
- name: GeneMarkS-2
  role: Self-training gene prediction for longer unmatched contigs (one fallback branch)
- name: MetaGeneMark
  role: GC-content-specific gene prediction for shorter unmatched contigs (other fallback
    branch)
- name: Parallel::ForkManager
  role: Perl module enabling parallel processing of contigs across cores in gm_hm.pl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:31be85b070b68af54167c5c99528b545a05a14c59a4c0e962a29b6ec611d5339
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenomic-annotation-statistics-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-contig gene-prediction branch assignments from a GeneMark-HM run (pan-genome-matched GeneMark.hmm-2, GeneMarkS-2 for longer unmatched contigs, MetaGeneMark for shorter unmatched contigs) into fractions comparable to the paper's Table 6, which reports 99.4% (gut), 89.0% (oral) and 79.4% (skin) pan-genome assignment rates.

## When to use

Use when you have GeneMark-HM output on human microbiome metagenome assemblies and need to quantify how many contigs were assigned to species-level pan-genomes versus handled by the GeneMarkS-2 or MetaGeneMark fallback branches.

## When NOT to use

- Input sequences are complete prokaryotic genomes or non-human-microbiome metagenomes; the pan-genome database and model selection were built specifically for human microbiome contigs
- You need absolute gene-prediction accuracy evaluation against truth labels; this skill only quantifies model-branch routing, not prediction accuracy (which requires the paper's simulated metagenome benchmarks)
- The pan-genome blastn database has not been built and the branch identity per contig cannot be recovered from the run

## Inputs

- Metagenome assembly FASTA files (e.g. GCA_900415335, GCA_003604395, GCA_003640265, GCA_013297495)
- GeneMark-HM refmod.tar database of model files from geneprobe.net
- ~20 Gb pan-genome sequence FASTA converted to an NCBI blastn database
- Per-contig GeneMark-HM run logs/output indicating which model-selection branch fired

## Outputs

- Per-metagenome counts of contigs assigned to each branch (pan-genome GeneMark.hmm-2, GeneMarkS-2, MetaGeneMark)
- Fractions of contigs assigned via the pan-genome path (table6_model_branch_fractions.tsv)
- Optionally the GeneMark-HM annotation outputs: output.gtf, aa.fasta, nt.fasta

## How to apply

Run gm_hm.pl on each metagenome assembly (e.g. GCA_900415335, GCA_003604395, GCA_003640265, GCA_013297495) after installing the refmod model-file database from geneprobe.net and building a blastn database of the ~20 Gb pan-genome sequence file with 'makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan'. Ensure NCBI blastn is on PATH (or hardcoded in gm_hm.pl) and Parallel::ForkManager is installed so contigs are processed in parallel with --cores 4. The pipeline first predicts genes with MetaGeneMark/GeneMarkS-2-class models and uses them as blastn queries against the pan-genome database; for each contig, record which branch produced the final predictions: pan-genome-derived GeneMark.hmm-2 model, GeneMarkS-2 (longer unmatched contigs), or MetaGeneMark (shorter unmatched contigs). Aggregate contig counts per branch per metagenome and compute the fraction routed through the pan-genome path; write results to a table such as table6_model_branch_fractions.tsv.

## Related tools

- **GeneMark-HM** (Pipeline entry script gm_hm.pl that performs model selection and produces the branch assignments being aggregated) — https://github.com/GeneProbe/GeneMark-HM
- **NCBI blastn** (Similarity search of initial gene predictions against the pan-genome database that determines contig-to-pan-genome assignment)
- **GeneMark.hmm-2** (Produces final gene predictions for contigs assigned to a pan-genome with pan-genome-derived models)
- **GeneMarkS-2** (Self-training gene prediction for longer unmatched contigs (one fallback branch))
- **MetaGeneMark** (GC-content-specific gene prediction for shorter unmatched contigs (other fallback branch))
- **Parallel::ForkManager** (Perl module enabling parallel processing of contigs across cores in gm_hm.pl)

## Examples

```
nohup bin/gm_hm.pl --seq assembly.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Reproduced pan-genome assignment fraction must be consistent with the paper's Table 6 values: up to 99.4% for human gut, 89.0% for oral, and 79.4% for skin metagenomes
- Per-contig branch labels must be exhaustive: every contig is assigned to exactly one of pan-genome GeneMark.hmm-2, GeneMarkS-2, or MetaGeneMark
- Sum of the three per-branch contig counts must equal the total contig count in the input assembly
- All reported fractions must fall in [0,1] and the pan-genome fraction must dominate (be the largest branch) for every metagenome, per the paper's finding that most predictions go through the pan-genome path
- Verbose-mode STDOUT from gm_hm.pl should report per-step performance statistics that match the aggregated branch counts

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a contig to a reference pan-genome; mis-assignment propagates into the aggregated statistics
- Oral and skin metagenomes show substantially lower pan-genome assignment coverage (down to 79.4%) possibly because these microbiomes are mixed with environmental microbes
- The length threshold distinguishing 'shorter' (MetaGeneMark) vs 'longer' (GeneMarkS-2) unmatched contigs is only defined in the paper's Figure 1, not in the article text; no software version numbers are stated for any of the gene finders

## Evidence

- [readme] To generate predictions on metagenome assembly: nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 ...: [evidence span withheld — anchor status: external_doc]
- [readme] In cases when no taxa assignment was made one of the two algorithms GeneMarkS-2 or MetaGeneMark-2 are used to generate the gene prediction.: [evidence span withheld — anchor status: external_doc]

---
name: perl-pipeline-configuration-and-execution
description: Use when predicting protein-coding genes in assembled metagenome contigs
  from shotgun sequencing of the human microbiome, especially short contigs where
  contig-specific parameter estimation is impossible and taxa-specific models are
  needed.
when_to_use_negative:
- Input is a single complete prokaryotic genome where GeneMarkS-2 self-training is
  appropriate rather than metagenomic model assignment
- Input is raw metagenomic reads rather than assembled contigs
- Metagenome is not from the human microbiome and taxon coverage of the human-microbiome
  pan-genome database (>5,000 taxa) is likely too low for reliable pan-genome assignment
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GeneMark-HM
  role: Pipeline being configured and executed via gm_hm.pl to produce GTF gene predictions
    for metagenome contigs
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: makeblastdb
  role: Builds the BLAST nucleotide database (pan) from the ~20 Gb pan-genome FASTA
    used for contig-to-pan-genome assignment
- name: blastn
  role: Fast similarity search backend required in PATH for the pan-genome assignment
    step
- name: GeneMarkS-2
  role: Self-training fallback gene finder used by the pipeline for longer contigs
    not assigned to pan-genomes
- name: MetaGeneMark
  role: GC-content-specific fallback gene finder used by the pipeline for shorter
    unassigned contigs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:0f505085e42c3b8a230a9dd0e0f4c2c80b84b79594fbd5163d85cc0f327b9fea
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
  - blastn
  - makeblastdb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# perl-pipeline-configuration-and-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run the GeneMark-HM Perl pipeline (gm_hm.pl) for protein-coding gene prediction in assembled human-microbiome metagenomes, which assigns contigs to species-level pan-genomes and applies taxa-specific GeneMark models instead of generic metagenomic parameters.

## When to use

Use when predicting protein-coding genes in assembled metagenome contigs from shotgun sequencing of the human microbiome, especially short contigs where contig-specific parameter estimation is impossible and taxa-specific models are needed.

## When NOT to use

- Input is a single complete prokaryotic genome where GeneMarkS-2 self-training is appropriate rather than metagenomic model assignment
- Input is raw metagenomic reads rather than assembled contigs
- Metagenome is not from the human microbiome and taxon coverage of the human-microbiome pan-genome database (>5,000 taxa) is likely too low for reliable pan-genome assignment

## Inputs

- Assembled metagenome contig FASTA file (e.g. simulated or real human microbiome shotgun assembly)
- GeneMark-HM license key file (~/.gmhmmp2_key)
- refmod database of pre-computed model files (downloaded from Amazon S3)
- Pan-genome sequences FASTA (~20 Gb) used to build a BLAST nucleotide database

## Outputs

- Gene predictions in GTF format (--out output.gtf)
- Predicted protein sequences FASTA (--aa aa.fasta)
- Predicted gene nucleotide sequences FASTA (--nt nt.fasta)
- Pipeline log with per-step performance details from --verbose mode

## How to apply

First complete environment setup: install the GeneMark-HM key file to ~/.gmhmmp2_key, verify Perl 5.10 with the Parallel::ForkManager module, and make the NCBI blastn executable available in PATH (or hard-code its path in gm_hm.pl). Clone the repository (git clone https://github.com/GeneProbe/GeneMark-HM.git), download the refmod model-file library and the ~20 Gb pan-genome FASTA from Amazon S3, and build the BLAST nucleotide database with makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan. Run the pipeline in a dedicated working directory per metagenome, invoking path_to/bin/gm_hm.pl --seq input.fasta --cores N --out output.gtf (add --aa and --nt to also emit protein and nucleotide FASTA of predicted genes), preferably in --verbose mode so per-step performance is reported on STDOUT; use --clean to remove temporary files afterward. The rationale is that the pipeline internally does fast similarity search of initially predicted genes against the pan-gene database to assign contigs to taxa-specific GeneMark.hmm-2 models, falling back to MetaGeneMark-2 (short contigs) or GeneMarkS-2 (longer contigs) when no assignment is made. Judge success by obtaining a complete GTF output with no pipeline-stage errors, and by comparing predicted GTFs against a reference annotation (e.g. PGAP) with the bundled compp utility.

## Related tools

- **GeneMark-HM** (Pipeline being configured and executed via gm_hm.pl to produce GTF gene predictions for metagenome contigs) — https://github.com/GeneProbe/GeneMark-HM
- **makeblastdb** (Builds the BLAST nucleotide database (pan) from the ~20 Gb pan-genome FASTA used for contig-to-pan-genome assignment)
- **blastn** (Fast similarity search backend required in PATH for the pan-genome assignment step)
- **GeneMarkS-2** (Self-training fallback gene finder used by the pipeline for longer contigs not assigned to pan-genomes)
- **MetaGeneMark** (GC-content-specific fallback gene finder used by the pipeline for shorter unassigned contigs)

## Examples

```
nohup path_to/bin/gm_hm.pl --seq sequence.fasta --cores 4 --aa aa.fasta --nt nt.fasta --out output.gtf --verbose > logfile &
```

## Evaluation signals

- Pipeline completes with exit 0 and produces output.gtf (plus --aa and --nt FASTA if requested) with no leftover temporary files when --clean is set
- --verbose STDOUT reports performance information for each pipeline step without errors in the pan-genome assignment stage
- makeblastdb succeeds in building the 'pan' nucleotide database from pan.fasta with -parse_seqids
- Gene predictions on the D1-D4 simulated metagenomes reproduce Table 3 missed-gene and gene-start error counts via compp comparison against the PGAP reference annotation (1,640,483 genes)

## Limitations

- Accuracy depends on correct assignment of contigs to reference pan-genomes; unassigned contigs fall back to less accurate generic models
- Pan-genome database covers more than five thousand human-gut taxa; coverage is lower for oral (89.0%) and skin (79.4%) metagenomes, possibly mixed with environmental microbes
- Software is licensed for academic and non-commercial use only, and requires a key file (~/.gmhmmp2_key), Perl 5.10 with Parallel::ForkManager, and blastn in PATH
- Pan-genome sequences file is ~20 Gb and download can take significant time

## Evidence

- [readme] README command invocation: [evidence span withheld — anchor status: external_doc]
- [readme] dedicated working directory recommendation: [evidence span withheld — anchor status: external_doc]

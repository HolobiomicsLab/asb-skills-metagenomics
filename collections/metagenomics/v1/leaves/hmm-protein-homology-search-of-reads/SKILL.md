---
name: hmm-protein-homology-search-of-reads
description: Use when raw, unassembled Illumina FASTQ reads must be screened for remote
  protein homologs (e.g. cas genes) and individual matching reads are needed as seeds,
  rather than assembled contigs.
when_to_use_negative:
- The target sequences are nucleotide repeats (e.g. CRISPR array repeats) with no
  protein homology — use DNA mode (--nucl) with VSEARCH identity matching against
  repeat FASTA queries instead
- The user already possesses the exact seed sequences — use user-defined mode (--seed
  -define file.fasta), which bypasses protein and DNA searches entirely
- Input consists of assembled contigs/genomes rather than short reads — complete assembly-and-annotation
  pipelines may be more appropriate since read-level six-frame translation is designed
  for short unassembled reads
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CasCollect
  role: Python wrapper pipeline whose default protein mode orchestrates read translation
    and HMMER search to collect cas gene seed reads
  repo: https://github.com/sandialabs/CasCollect
- name: FragGeneScanPlus
  role: Translates each raw sequencing read into the most likely of its six open reading
    frames, scaled for high-throughput datasets via multithreading
  repo: hallamlab/FragGeneScanPlus
- name: HMMER
  role: Searches read translations against the 120 Cas protein family profile HMMs
    (or user-provided HMM directory) to identify matching seed reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:5fa20afb643ab616b0374f0da5759f0e692a4aef7972335d543df63bb982ed4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# HMM protein homology search of reads

## Summary

Translates raw short-read sequencing data into candidate proteins and searches them with HMMER against Cas protein profile HMMs to identify seed reads, as used in CasCollect's default protein mode for targeted CRISPR/Cas assembly.

## When to use

Use when raw, unassembled Illumina FASTQ reads must be screened for remote protein homologs (e.g. cas genes) and individual matching reads are needed as seeds, rather than assembled contigs.

## When NOT to use

- The target sequences are nucleotide repeats (e.g. CRISPR array repeats) with no protein homology — use DNA mode (--nucl) with VSEARCH identity matching against repeat FASTA queries instead
- The user already possesses the exact seed sequences — use user-defined mode (--seed -define file.fasta), which bypasses protein and DNA searches entirely
- Input consists of assembled contigs/genomes rather than short reads — complete assembly-and-annotation pipelines may be more appropriate since read-level six-frame translation is designed for short unassembled reads

## Inputs

- Raw unassembled paired-end Illumina FASTQ reads (e.g. bacterial isolate or metagenomic reads)
- Profile HMM collection: default 120 Cas protein family profile HMMs shipped with CasCollect, or a user-supplied HMM directory (-hmm)

## Outputs

- Seed reads whose translated ORFs match a profile HMM, pooled as seed_reads.fastq ready for downstream vsearch-based read subset expansion

## How to apply

In CasCollect's default protein mode (enabled unless --noprot), each raw sequencing read is translated by FragGeneScanPlus into the most likely of its six open reading frames, leveraging FragGeneScan's hidden Markov model that combines codon usage bias, sequencing error models and start/stop codon patterns. The resulting six-frame translations are then searched with HMMER against the default set of 120 Cas protein family profile HMMs (shipped with CasCollect, sourced from CRISPRCasFinder) or against a user-provided HMM directory via the -hmm option for reconfigurable, non-Cas gene targets. Reads whose translated ORFs hit a profile HMM above threshold are collected and added to the seed-read pool, which is combined with DNA-mode (--nucl VSEARCH) and user-defined (--seed -define) seeds and written to seed_reads.fastq. Profile HMM search is chosen over pairwise similarity because cas gene families are diverse and rapidly evolving, so position-specific models capture remote homology that exact or near-exact nucleotide matching would miss. Before running, verify all dependencies (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) are in the user's path, using the built-in check or Check.py, which can auto-download missing software.

## Related tools

- **CasCollect** (Python wrapper pipeline whose default protein mode orchestrates read translation and HMMER search to collect cas gene seed reads) — https://github.com/sandialabs/CasCollect
- **FragGeneScanPlus** (Translates each raw sequencing read into the most likely of its six open reading frames, scaled for high-throughput datasets via multithreading) — hallamlab/FragGeneScanPlus
- **HMMER** (Searches read translations against the 120 Cas protein family profile HMMs (or user-provided HMM directory) to identify matching seed reads)

## Examples

```
python Check.py
```

## Evaluation signals

- seed_reads.fastq is produced, is a valid FASTQ/FASTA of read sequences, and is non-empty only when the HMMER search returned matches
- Module executes without error for each enabled flag combination (e.g. default protein mode vs. --noprot vs. -hmm user directory)
- All required third-party programs (HMMER, FragGeneScan, etc.) are detected in the user's path before the run; Check.py reports no missing software
- Known cas genes from benchmark isolate genomes (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39) are recovered, mirroring the article's result that all previously identified cas genes were found
- Removing matched seed reads leaves no redundant HMMER hits when the same read set is re-screened, confirming matches are captured once and passed downstream

## Limitations

- The paper does not benchmark the seed-generation modes in isolation, so no mode-specific sensitivity/precision thresholds are established
- Protein mode depends on six-frame ORF prediction of short reads (90–226 bp in the benchmarks); very short or error-prone reads may yield poor translations
- Detection is limited by the profile HMM set — without a user-supplied -hmm directory, only the 120 shipped Cas protein families can be found
- No software version numbers are stated for CasCollect or its third-party dependencies in the paper

## Evidence

- [results] Default 120 Cas protein HMMs as HMMER query: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] CasCollect identifies reads matching HMM profiles: "CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [readme] FragGeneScan HMM-based ORF prediction on short reads: [evidence span withheld — anchor status: external_doc]
- [readme] CasCollect purpose from README: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."

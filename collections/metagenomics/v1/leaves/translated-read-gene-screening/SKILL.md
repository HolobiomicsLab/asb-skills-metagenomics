---
name: translated-read-gene-screening
description: Use when you have raw short DNA sequencing reads (FASTQ) and need to
  find reads encoding a known protein family without assembling the dataset first.
when_to_use_negative:
- The target genes are nucleotide features such as CRISPR repeat arrays rather than
  proteins — use CasCollect DNA mode (--nucl with -query) which VSEARCH-matches reads
  against repeat FASTA files instead
- You already have a user-defined seed FASTA to assemble around — use --seed -define
  file.fasta, which bypasses protein and DNA searches entirely
- Input is assembled contigs or genomes rather than raw reads — standard HMMER annotation
  of ORF-called assemblies is more appropriate than read-level translation
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: CasCollect
  role: Python wrapper pipeline whose protein mode orchestrates read translation and
    HMM search to generate cas gene seed reads
  repo: https://github.com/sandialabs/CasCollect
- name: FragGeneScanPlus
  role: Translates each raw sequencing read into its most likely open reading frame
    using a hidden Markov model combining codon usage, sequencing error and start/stop
    codon patterns
  repo: https://github.com/hallamlab/FragGeneScanPlus
- name: HMMER
  role: Searches the translated reads against Cas protein family profile HMMs to identify
    matching seed reads
- name: VSEARCH
  role: Used in the adjacent DNA mode for identity matching of reads to nucleotide
    repeat queries; complements the protein-mode HMM screening in the combined seed
    pool
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:a9e2668a04278e45401016afafaab520cacf4ea461282ffd617e4a87a704634c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  - VSEARCH
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# translated-read-gene-screening

## Summary

Screen raw unassembled Illumina reads for protein-coding genes of interest by predicting open reading frames directly on reads (FragGeneScan/FragGeneScan-Plus) and searching the translations with HMMER against profile HMMs — here the default 120 Cas protein family profile HMMs used by CasCollect's protein mode to collect cas gene seed reads.

## When to use

Use when you have raw short DNA sequencing reads (FASTQ) and need to find reads encoding a known protein family without assembling the dataset first.

## When NOT to use

- The target genes are nucleotide features such as CRISPR repeat arrays rather than proteins — use CasCollect DNA mode (--nucl with -query) which VSEARCH-matches reads against repeat FASTA files instead
- You already have a user-defined seed FASTA to assemble around — use --seed -define file.fasta, which bypasses protein and DNA searches entirely
- Input is assembled contigs or genomes rather than raw reads — standard HMMER annotation of ORF-called assemblies is more appropriate than read-level translation

## Inputs

- Raw unassembled short-read Illumina sequencing data in FASTQ format (paired-end or single-end)
- Profile HMM collection for the target protein family (default: 120 Cas protein family profile HMMs bundled with CasCollect; alternative: user-supplied HMM directory via -hmm)

## Outputs

- Seed reads (FASTQ/FASTA) whose translations match target protein family HMMs, written to seed_reads.fastq for downstream read-subset expansion
- Translated read ORF predictions produced by FragGeneScanPlus (intermediate protein sequences searched by HMMER)

## How to apply

Run CasCollect in its default protein mode (omit --noprot) so each FASTQ read is translated by FragGeneScanPlus into its most likely open reading frame among the six frames. Search the resulting translated reads with HMMER against the default set of 120 Cas protein family profile HMMs shipped with CasCollect, or supply a user HMM directory with -hmm (usable with --noprot for custom targets alone). Reads whose translations hit a profile HMM are collected as cas gene seed reads. Seeds from protein mode can be combined with DNA-mode (--nucl -query) VSEARCH repeat matches and user-defined seeds (--seed -define) into a single seed_reads.fastq that primes the downstream vsearch read-subset expansion. This read-level, translated search avoids complete genome assembly and is the reason CasCollect reduces cas gene identification time by almost two orders of magnitude versus full assembly pipelines.

## Related tools

- **CasCollect** (Python wrapper pipeline whose protein mode orchestrates read translation and HMM search to generate cas gene seed reads) — https://github.com/sandialabs/CasCollect
- **FragGeneScanPlus** (Translates each raw sequencing read into its most likely open reading frame using a hidden Markov model combining codon usage, sequencing error and start/stop codon patterns) — https://github.com/hallamlab/FragGeneScanPlus
- **HMMER** (Searches the translated reads against Cas protein family profile HMMs to identify matching seed reads)
- **VSEARCH** (Used in the adjacent DNA mode for identity matching of reads to nucleotide repeat queries; complements the protein-mode HMM screening in the combined seed pool) — https://github.com/torognes/vsearch

## Examples

```
python CasCollect.py -1 reads_R1.fq.gz -2 reads_R2.fq.gz -o out_dir
```

## Evaluation signals

- CasCollect completes without error for the enabled flag combination (protein mode default, or --noprot with -hmm) and its pre-run software check reports FragGeneScan, HMMER and VSEARCH all present in the user's path
- seed_reads.fastq exists, is a valid FASTQ/FASTA of read sequences, and is non-empty only when the HMM search returned matches
- On a benchmark isolate with known cas content (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39), all previously known cas genes are recovered, matching the complete-assembly annotation result
- In the simulated metagenomic benchmark (bacterial reads pooled with spiked-in mouse reads), no cas seed reads derive from the contaminant/organism background
- Seed reads are consistent with the chosen profile HMM set: swapping in a user -hmm directory should change the recovered seed composition accordingly

## Limitations

- The paper does not benchmark seed-generation modes in isolation; sensitivity/precision of protein-mode screening alone is not quantified
- Detection is limited by the profile HMM collection used — the default 120 Cas protein family HMMs constrain which protein families can be seeded, and the default human-genome contaminant reference applies only if --clean is run
- Read-level ORF prediction is most reliable for short incomplete gene fragments typical of 90–226 bp reads; very short or low-quality reads may yield no usable translation
- No software version numbers are stated for CasCollect or its third-party dependencies, hindering exact reproducibility

## Evidence

- [results] protein-mode HMM screening: "CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [results] default HMM set: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [intro] seed-read priming: "identifying reads that match cas genes and/or CRISPR arrays and using these ‘seed’ reads to prime the targeted assembly"
- [abstract] speedup rationale: "Benchmarking our targeted assembly pipeline demonstrates significantly improved timing by almost two orders of magnitude"
- [other] FragGeneScanPlus role: [evidence span withheld — anchor status: external_doc]
- [abstract] user reconfigurability: "reconfigurable for user provided Hidden Markov Models and/or reference nucleotide sequences"
- [readme] software path requirement: "Before running, **CasCollect** will check for the necessary software in the users path and denote any missing software."

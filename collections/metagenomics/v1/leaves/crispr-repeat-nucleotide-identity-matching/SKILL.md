---
name: crispr-repeat-nucleotide-identity-matching
description: Use when raw unassembled FASTQ reads must be searched for CRISPR array
  repeat sequences by nucleotide identity to seed targeted assembly, especially where
  repeat-only arrays without cas genes would be missed by protein HMM screening.
when_to_use_negative:
- Input is already an assembled contig or genome — the skill is designed for short
  unassembled sequencing reads and loses purpose on assemblies.
- The target is protein-coding cas genes rather than CRISPR array repeats — protein
  mode (FragGeneScanPlus translation + HMMER against the 120 Cas profile HMMs) is
  the appropriate search.
- The user already has a defined seed FASTA and wants to bypass all searching — use
  the --seed -define mode to supply seeds directly.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: CasCollect
  role: Pipeline wrapper whose DNA mode (--nucl) coordinates repeat-based seed read
    collection and writes seed_reads.fastq
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Performs the identity matching of raw reads against the user-defined CRISPR
    repeat nucleotide query set
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:e5f6d9dfd9649d7e0d7614463492bb7841db3070c2578a77ef2372ba02b5d06b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# crispr-repeat-nucleotide-identity-matching

## Summary

CasCollect's DNA-mode seed generation: identify raw sequencing reads containing CRISPR array repeats by VSEARCH identity matching against a user-defined FASTA of repeat sequences, adding the matching reads to the seed pool. This complements protein-mode HMMER screening of cas genes so that repeat-only arrays lacking cas genes are still captured.

## When to use

Use when raw unassembled FASTQ reads must be searched for CRISPR array repeat sequences by nucleotide identity to seed targeted assembly, especially where repeat-only arrays without cas genes would be missed by protein HMM screening.

## When NOT to use

- Input is already an assembled contig or genome — the skill is designed for short unassembled sequencing reads and loses purpose on assemblies.
- The target is protein-coding cas genes rather than CRISPR array repeats — protein mode (FragGeneScanPlus translation + HMMER against the 120 Cas profile HMMs) is the appropriate search.
- The user already has a defined seed FASTA and wants to bypass all searching — use the --seed -define mode to supply seeds directly.

## Inputs

- Raw Illumina short-read sequencing data (paired-end FASTQ)
- User-defined nucleotide query FASTA of CRISPR repeat sequences (supplied via -query)
- Optional: CasCollect's bundled 105 CRISPR repeat sequences as individual FASTA files

## Outputs

- DNA-mode seed reads (reads identity-matched to repeat queries) merged into the combined seed_reads.fastq

## How to apply

Run CasCollect with the --nucl flag together with -query pointing at a nucleotide query FASTA of CRISPR repeat sequences; CasCollect then uses VSEARCH to identity-match the raw reads against this query set and adds the matching reads to the seed pool. CasCollect ships 105 CRISPR repeat sequences (from CRISPRCasFinder) as individual FASTA files, so a user can supply a curated subset or their own repeat file. The percent identity for vsearch matching is adjustable via -match (the pipeline's default identity threshold is a moderately high 95%, set for the read-subset expansion step). Matching reads are merged with any protein-mode (FragGeneScanPlus + HMMER) seed reads into a single seed_reads.fastq that primes the downstream vsearch read-subset expansion. Because the modes are optional and composable, verify that --nucl is paired with its required -query file and that combined seeds exist only where searches returned matches.

## Related tools

- **CasCollect** (Pipeline wrapper whose DNA mode (--nucl) coordinates repeat-based seed read collection and writes seed_reads.fastq) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Performs the identity matching of raw reads against the user-defined CRISPR repeat nucleotide query set) — https://github.com/torognes/vsearch

## Examples

```
./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt
```

## Evaluation signals

- seed_reads.fastq is a valid, non-empty FASTQ whose reads correspond to matches returned by the VSEARCH identity search against the repeat query file.
- The module exits without error only when the documented constraint is satisfied: --nucl is invoked together with a -query FASTA file.
- Matched reads identified in a previous search cycle are removed from the database set so redundant searches are eliminated in subsequent expansion cycles.
- For isolate datasets with known CRISPR arrays (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39), repeat-containing seed reads should be recovered such that downstream assembly/CRISPRCasFinder detects the known arrays.
- Identity threshold used is consistent with the -match setting; the pipeline default is a moderately high 95% identity.

## Limitations

- Nucleotide identity matching only recovers reads closely matching supplied repeat sequences; divergent or novel repeat arrays absent from the query FASTA will be missed.
- The paper does not benchmark seed-generation modes in isolation, so per-mode sensitivity is not independently validated.
- CasCollect is restricted to short Illumina sequencing reads; longer-read platforms are out of scope.

## Evidence

- [results] DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats: "DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats"
- [results] 105 bundled repeats: "We have included 105 CRISPR repeat sequences (13) as individual Fasta files with CasCollect"
- [results] default identity threshold: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [methods] -match CLI parameter: "-match number       percent match for vsearch"
- [intro] seed read concept: "identifying reads that match cas genes and/or CRISPR arrays and using these ‘seed’ reads to prime the targeted assembly"

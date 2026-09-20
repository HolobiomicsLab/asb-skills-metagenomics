---
name: vsearch-read-subset-expansion
description: Use when running CasCollect's read subset expansion step to grow seed
  reads (matching cas gene HMM profiles or CRISPR array repeats) into a larger read
  subset prior to targeted SPAdes assembly.
when_to_use_negative:
- Input is not raw unassembled sequencing reads (e.g. a finished genome or existing
  assembly) — the expansion step is designed to avoid complete assembly of raw reads
- 'DNA repeat mode when the goal is cas gene detection in protein mode, or vice versa:
  VSEARCH identity matching is used for user-defined CRISPR array repeats in DNA mode,
  while protein mode uses HMMER profiles for seeds'
- Data are metagenomic and metagenome-scale diversity is expected to overwhelm the
  95% identity threshold with strain variants, unless metaSPAdes is substituted downstream
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3673
tools:
- name: VSEARCH
  role: Performs identity matching of reads to the seed subset during read subset
    expansion; in DNA mode it matches reads to user-defined CRISPR array repeats
  repo: https://github.com/torognes/vsearch
- name: CasCollect
  role: Python wrapper pipeline that orchestrates the seed generation, VSEARCH expansion
    cycles, matched-read database pruning, and subsequent SPAdes assembly
  repo: https://github.com/sandialabs/CasCollect
- name: SPAdes
  role: Assembles the expanded read subset produced by the VSEARCH expansion step
- name: HMMER
  role: Provides the 120 Cas protein family profile HMMs whose matches define seed
    reads in protein mode before expansion begins
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:d454e25f9e8ce085f8631aedc0de8664f40a96c2a39cf2da23e2567564a25d34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - CasCollect
  - HMMER
  - SPAdes
  - VSEARCH
  - bbduk
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# vsearch-read-subset-expansion

## Summary

Iterative read-subset expansion from seed reads using VSEARCH in CasCollect: matched reads are cycled against the read database at a 95% identity threshold to grow the targeted assembly input around CRISPR/Cas loci, then assembled with SPAdes.

## When to use

Use when running CasCollect's read subset expansion step to grow seed reads (matching cas gene HMM profiles or CRISPR array repeats) into a larger read subset prior to targeted SPAdes assembly.

## When NOT to use

- Input is not raw unassembled sequencing reads (e.g. a finished genome or existing assembly) — the expansion step is designed to avoid complete assembly of raw reads
- DNA repeat mode when the goal is cas gene detection in protein mode, or vice versa: VSEARCH identity matching is used for user-defined CRISPR array repeats in DNA mode, while protein mode uses HMMER profiles for seeds
- Data are metagenomic and metagenome-scale diversity is expected to overwhelm the 95% identity threshold with strain variants, unless metaSPAdes is substituted downstream

## Inputs

- CasCollect seed reads matching cas gene HMM profiles (protein mode) or CRISPR array repeat sequences (DNA mode)
- Full paired-end Illumina FASTQ read dataset (e.g. E. coli KLY, P. aeruginosa VA-134, or S. pyogenes M39 genomic reads)
- CasCollect command-line parameters (-match, -cycle, -cpu, -mem)

## Outputs

- Expanded read subset for the CRISPR/Cas locus
- SPAdes-targeted assembly contigs from the expanded subset (contigs ≥ 500 bp tallied as annotatable)

## How to apply

After seed generation, use VSEARCH (in DNA mode) to iteratively search the full read database for reads matching the current seed set at the default 95% identity threshold (adjustable with -match). The default number of expansion cycles is five; the task recipe used -cycle 2 to reduce VSEARCH seed expansion from the default five to two cycles. Between cycles, matched reads are removed from the database to eliminate redundant searches, and newly matched reads are added to the seed subset. The expanded subset is then passed to SPAdes for targeted assembly of only the CRISPR/Cas regions. Invoke CasCollect.py with 1 CPU and 20 Gb RAM defaults (-cpu/-mem) unless otherwise specified.

## Related tools

- **VSEARCH** (Performs identity matching of reads to the seed subset during read subset expansion; in DNA mode it matches reads to user-defined CRISPR array repeats) — https://github.com/torognes/vsearch
- **CasCollect** (Python wrapper pipeline that orchestrates the seed generation, VSEARCH expansion cycles, matched-read database pruning, and subsequent SPAdes assembly) — https://github.com/sandialabs/CasCollect
- **SPAdes** (Assembles the expanded read subset produced by the VSEARCH expansion step)
- **HMMER** (Provides the 120 Cas protein family profile HMMs whose matches define seed reads in protein mode before expansion begins)

## Examples

```
python CasCollect.py reads_R1.fastq reads_R2.fastq -fwd cas_seeds.fna -rev -out outdir --trim -cycle 2 -cpu 1 -mem 20
```

## Evaluation signals

- Expansion completes with VSEARCH reporting matches at the configured -match threshold (default 95%) over the configured number of -cycle rounds
- Matched reads are removed from the database set between cycles — verify no reads are searched redundantly across cycles
- Resulting SPAdes assembly of the expanded subset yields contigs that contain all previously known cas genes for the isolate (both pipelines identified all known cas genes in the three isolate genomes)
- Assembled contigs ≥ 500 bp are annotatable and contain the expected cas operons and CRISPR arrays; runtime should be consistent with targeted assembly rather than complete genome assembly
- Changing -cycle from 5 to 2 should reduce seed-expansion search time without losing cas gene/CRISPR array recovery

## Limitations

- 95% default identity match may miss divergent homologs; the threshold is only 'moderately high' and must be tuned with -match for divergent systems
- Fewer expansion cycles (e.g. -cycle 2 instead of the default 5) trades sensitivity of locus capture for reduced search time
- No software version numbers stated for VSEARCH or other CasCollect dependencies, complicating exact reproducibility of expansion results

## Evidence

- [results] default read subset expansion identity match of 95%: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] default five read subset expansion cycles: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] matched reads removed between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [results] VSEARCH DNA mode identity matching: "DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats"
- [results] expansion output feeds SPAdes assembly: "The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps"
- [readme] CasCollect preflight checks dependencies in path: "Before running, **CasCollect** will check for the necessary software in the users path and denote any missing software."

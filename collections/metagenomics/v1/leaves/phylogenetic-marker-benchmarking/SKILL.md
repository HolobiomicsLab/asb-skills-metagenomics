---
name: phylogenetic-marker-benchmarking
description: Use when assessing whether DIAMOND's tantan masking suppresses alignments
  during Patchwork marker retrieval — especially for read-based query data — by A/B
  comparing --masking 0 against the default masking configuration.
when_to_use_negative:
- The goal is marker retrieval itself rather than comparing DIAMOND masking settings
  — a single default Patchwork run suffices
- Query and reference are highly divergent (e.g., cross-species near-USCOs like the
  H. robusta references), where percent-identity drops may confound the masking comparison
- Input is already a concatenated alignment or feature table rather than WGS contigs/reads
  to be searched
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Patchwork
  role: Marker retrieval and concatenation tool under benchmark; wraps DIAMOND BLASTX
    and exposes DIAMOND arguments such as --masking
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Sequence aligner performing the BLASTX similarity search whose tantan masking
    setting (--masking 0) is the benchmark variable
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:7d48e6d42b40649f6f8ac41e60f9514f51a7a90e1387eb1283acf4b9bba50d60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# phylogenetic-marker-benchmarking

## Summary

Benchmark a DIAMOND-based marker retrieval pipeline (Patchwork) by running paired configurations that differ only in the DIAMOND tantan masking setting (--masking 0 vs. default), then compare percent identity and query coverage of recovered markers against same-species USCO references.

## When to use

Use when assessing whether DIAMOND's tantan masking suppresses alignments during Patchwork marker retrieval — especially for read-based query data — by A/B comparing --masking 0 against the default masking configuration.

## When NOT to use

- The goal is marker retrieval itself rather than comparing DIAMOND masking settings — a single default Patchwork run suffices
- Query and reference are highly divergent (e.g., cross-species near-USCOs like the H. robusta references), where percent-identity drops may confound the masking comparison
- Input is already a concatenated alignment or feature table rather than WGS contigs/reads to be searched

## Inputs

- SPAdes de novo assembly of D. gyrociliatus Illumina short reads (K-mer size 55) as query contigs (DNA FASTA)
- 815 D. gyrociliatus USCO reference sequences from GenBank accession GCA_904063045.1 (amino acid FASTA)

## Outputs

- Side-by-side table of percent identity (exact matches over all aligned positions) and query coverage (total aligned positions) for the masking-enabled and masking-disabled Patchwork runs
- Recovered and concatenated marker sequences per reference USCO for each configuration

## How to apply

Load the SPAdes de novo assembly of D. gyrociliatus Illumina short reads (K-mer size 55) as query and the 815 same-species USCOs from GenBank accession GCA_904063045.1 as reference. Run Patchwork (Julia implementation using DIAMOND BLASTX) with default BLOSUM62 scoring (gap open 11, gap extension 1), --ultra-sensitive --frameshift 15, keeping tantan masking enabled; then repeat the identical run with the sole change of --masking 0. For each run, evaluate recovered markers by computing exact matches over all aligned positions (percent identity) and total aligned positions (query coverage), and tabulate the two runs side by side. The masking-disabled run should show higher query coverage with only a slight decrease in percent identity, an effect most pronounced when raw reads rather than assemblies are used as query input.

## Related tools

- **Patchwork** (Marker retrieval and concatenation tool under benchmark; wraps DIAMOND BLASTX and exposes DIAMOND arguments such as --masking) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Sequence aligner performing the BLASTX similarity search whose tantan masking setting (--masking 0) is the benchmark variable) — https://github.com/bbuchfink/diamond

## Examples

```
patchwork --query assembly.fasta --reference uscos.fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- The masking-disabled (--masking 0) run yields higher total aligned positions (query coverage) than the masking-enabled run on the same query/reference pair
- Percent identity decreases only slightly when masking is disabled, mirroring the reported observation
- The improvement in query coverage is disproportionately large for read-based (unassembled) query input compared to assembly input
- With a high-quality same-species assembly, the masking-enabled baseline retrieves all 815 markers at ~95.9% average identical aligned positions and ~92.2% mean query coverage, providing a sanity-reference point

## Limitations

- Disabling tantan masking trades percent identity for coverage — identity decreases slightly, so downstream trimming/quality checks are still needed
- Benchmark validity depends on the only changed parameter being --masking 0; other settings (e.g., --ultra-sensitive --frameshift 15, BLOSUM62 gap open 11/extend 1) must be held constant
- Findings were established on D. gyrociliatus USCO data; reference should be limited to near-USCOs to avoid chimeric sequences

## Evidence

- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [methods] 815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1): "815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1)"
- [results] On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%: "On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [readme] Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data.: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."

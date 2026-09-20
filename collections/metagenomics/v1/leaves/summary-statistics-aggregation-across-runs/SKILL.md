---
name: summary-statistics-aggregation-across-runs
description: Use when pooling per-contig results (seed length, final length, extension
  gain) from several ContigExtender runs into a single combined statistics table such
  as summary_statistics.csv.
when_to_use_negative:
- Input is a single-run result with no cross-run pooling needed — compute per-run
  statistics directly instead
- Contigs lack parsable pairing between extended output and seed contigs (e.g., headers
  were renamed), making per-contig gain computation impossible
- The goal is accuracy assessment against references (e.g., BLAST to NCBI viral reference
  nucleotide sequences) rather than length/quality aggregation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: ContigExtender
  role: Produces the extended contigs whose lengths and quality are aggregated across
    the NIBSC, Animal, and Human runs
  repo: https://github.com/dengzac/contig-extender
- name: metaSPAdes
  role: Generates the initial seed contigs that serve as the baseline for computing
    seed length and extension gain
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:35e53793b17f0be62df1feba63db5237dfb127c4a1f6788780f43eab6e48d9c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# summary-statistics-aggregation-across-runs

## Summary

Aggregates per-contig length and quality measurements from multiple ContigExtender runs (e.g., NIBSC, Animal, Human metagenomic datasets) into combined summary statistics, reporting means, medians, and chimera-free counts to quantify aggregate extension improvement.

## When to use

Use when pooling per-contig results (seed length, final length, extension gain) from several ContigExtender runs into a single combined statistics table such as summary_statistics.csv.

## When NOT to use

- Input is a single-run result with no cross-run pooling needed — compute per-run statistics directly instead
- Contigs lack parsable pairing between extended output and seed contigs (e.g., headers were renamed), making per-contig gain computation impossible
- The goal is accuracy assessment against references (e.g., BLAST to NCBI viral reference nucleotide sequences) rather than length/quality aggregation

## Inputs

- Extended contigs in FASTA format (output/contigs.fasta per ContigExtender run)
- metaSPAdes seed contigs in FASTA format (the 'fasta file with contigs to extend') for each dataset (NIBSC, Animal, Human)

## Outputs

- summary_statistics.csv with pooled means (seed length, gain, final length), medians (gain, seed length, output length), and chimera-free counts

## How to apply

Load each run's extended contigs (output/contigs.fasta) together with the corresponding metaSPAdes seed contigs (the fasta file with contigs to extend). Parse FASTA headers to pair each extended contig with its seed, then compute seed length, final length, and extension gain (final length minus seed length) in kb for each contig. Inspect each extension to classify whether it is chimeric, tallying chimera-free contigs out of the total. Pool per-contig measurements across runs and compute means (seed length, gain, final length) and medians (gain, seed length, output length), then write the aggregated statistics to summary_statistics.csv. As a validation anchor, the combined 49-contig result should reproduce: 45/49 chimera-free, average seed length 2.8 kb, ~2 kb average gain, 4.8 kb average final length, with medians of 1.5 kb gain, 2.4 kb seed, and 3.9 kb output.

## Related tools

- **ContigExtender** (Produces the extended contigs whose lengths and quality are aggregated across the NIBSC, Animal, and Human runs) — https://github.com/dengzac/contig-extender
- **metaSPAdes** (Generates the initial seed contigs that serve as the baseline for computing seed length and extension gain)

## Examples

```
python aggregate_stats.py --extended output/contigs.fasta --seeds seeds.fasta --out summary_statistics.csv
```

## Evaluation signals

- Combined 49-contig result reproduces 45 of 49 contigs free of chimeric extensions
- Average seed length across the 45 high-quality contigs is 2.8 kb with ~2 kb average gain
- Average final contig length equals 4.8 kb for the 45 high-quality contigs
- Medians match 1.5 kb gain, 2.4 kb seed length, and 3.9 kb output length
- Per-contig gain equals final length minus seed length when spot-checked against FASTA sequence lengths

## Limitations

- Chimeric extensions must be excluded from the pooled means; including them skews length statistics (4 of 49 contigs were chimeric in the combined result)
- Contigs that are not extendable gain no length, so aggregate improvement is conditional on extendable contigs
- Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded upstream, so downstream statistics inherit any upstream filtering effects

## Evidence

- [discussion] For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb: "For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb"
- [readme] The output contigs for each input will be found in the output/contigs.fasta`` sorted by length and in the order they appeared in the input.: "The output contigs for each input will be found in the output/contigs.fasta`` sorted by length and in the order they appeared in the input."
- [intro] Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended.: "Since metaSPAdes has been adopted by many labs, we will use it as our benchmark tool to generate the initial seed contigs to be extended."
- [readme] Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded.: "Paired alignments with incorrect orientation or fragment lengths longer than 500bp are excluded."

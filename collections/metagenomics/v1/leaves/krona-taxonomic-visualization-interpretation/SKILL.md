---
name: krona-taxonomic-visualization-interpretation
description: 'Use when Mapler has split HiFi metagenome reads into mapped/unmapped
  FASTQ fractions (kraken2: true) and you must compare their taxonomic composition,
  e.g. quantify phylum-level bacteria or find taxa unique to unassembled reads.'
when_to_use_negative:
- Taxonomic assignment of assembly bins is the goal — Mapler assigns bins with GTDB-Tk,
  not via read-fraction Krona plots.
- Bin quality (completeness/contamination) from marker gene content is the question
  — use the checkm-plot.pdf/checkm_report.txt outputs instead.
- Reads were never split into mapped/unmapped fractions (kraken2 not enabled in the
  configfile), so no per-fraction comparison can be drawn.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kraken 2
  role: Assigns taxonomy to each mapped/unmapped read fraction; its kraken2.tsv per-fraction
    output is the basis of the Krona visualization being interpreted
  repo: https://github.com/DerrickWood/kraken2
- name: Krona
  role: Renders the interactive krona.html taxonomic rings per fraction used to quantify
    phylum-level bacteria and spot taxa (e.g., Legionella) unique to one fraction
- name: Mapler
  role: 'Splits reads into mapped/unmapped fractions via read-contig alignment, runs
    the Kraken 2 + Krona step when kraken2: true, and places krona.html under outputs/<sample_name>/<assembler>/kraken2/'
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:89355a7fbe9bb8f0d8c1e91efd1476a17a426948c1ad4cc7455f541a36d4fb66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken2
  tools:
  - Bash
  - Conda
  - FastQC
  - KAT
  - Kraken 2
  - Kraken2
  - Krona
  - Mapler
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# krona-taxonomic-visualization-interpretation

## Summary

Turns Kraken 2 assignments of Mapler's mapped vs unmapped HiFi read fractions into interactive Krona HTML rings so that taxonomic gaps between assembled and unassembled reads (e.g., 76% vs 56% phylum-level bacteria; Legionella only in unmapped Deadwood reads) can be quantified and interpreted.

## When to use

Use when Mapler has split HiFi metagenome reads into mapped/unmapped FASTQ fractions (kraken2: true) and you must compare their taxonomic composition, e.g. quantify phylum-level bacteria or find taxa unique to unassembled reads.

## When NOT to use

- Taxonomic assignment of assembly bins is the goal — Mapler assigns bins with GTDB-Tk, not via read-fraction Krona plots.
- Bin quality (completeness/contamination) from marker gene content is the question — use the checkm-plot.pdf/checkm_report.txt outputs instead.
- Reads were never split into mapped/unmapped fractions (kraken2 not enabled in the configfile), so no per-fraction comparison can be drawn.

## Inputs

- Mapler read fractions as FASTQ (<fraction_name>_reads.fastq, e.g. mapped/unmapped) under outputs/<sample_name>/<assembler>/
- Kraken 2 database (kraken2db path in the Mapler configfile, e.g. from benlangmead.github.io/aws-indexes/k2)
- Per-fraction Kraken 2 classification table kraken2.tsv feeding the Krona rendering

## Outputs

- Interactive krona.html plus krona.html.files per read fraction under outputs/<sample_name>/<assembler>/kraken2/<fraction_name>/
- Quantified per-fraction taxonomic summary (e.g., % bacteria assignable at phylum level: 56% unmapped vs 76% mapped; Legionella restricted to unmapped reads) for the fraction-comparison table

## How to apply

In the Mapler configfile set kraken2: true and point kraken2db at a Kraken 2 database chosen for the targeted taxa and available disk space; the pipeline classifies each read fraction (<fraction_name>_reads.fastq: mapped, unmapped) and writes kraken2.tsv plus krona.html under outputs/<sample_name>/<assembler>/kraken2/<fraction_name>/. Read the interactive rings rank by rank (phylum to species) to quantify how much of each fraction is assignable: in the Deadwood HiFi benchmark, ~76% of assembled (mapped) reads vs ~56% of unassembled (unmapped) reads were bacteria assignable at phylum level, the gap marking taxonomically novel or low-abundance lineages among unassembled reads. Scan the unmapped plot for taxa absent from the mapped plot — Legionella species appeared only in the unmapped fraction — as evidence that assembly missed whole populations; per the article, low-abundance reads are typically over-represented in the unmapped portion. Cross-check ring proportions against kraken2.tsv counts and record them in the fraction-comparison summary alongside read_contig_mapping.txt and report.txt, accepting the interpretation when per-fraction phylum-level percentages and presence/absence calls match reported values within stochastic tolerance.

## Related tools

- **Kraken 2** (Assigns taxonomy to each mapped/unmapped read fraction; its kraken2.tsv per-fraction output is the basis of the Krona visualization being interpreted) — https://github.com/DerrickWood/kraken2
- **Krona** (Renders the interactive krona.html taxonomic rings per fraction used to quantify phylum-level bacteria and spot taxa (e.g., Legionella) unique to one fraction)
- **Mapler** (Splits reads into mapped/unmapped fractions via read-contig alignment, runs the Kraken 2 + Krona step when kraken2: true, and places krona.html under outputs/<sample_name>/<assembler>/kraken2/) — https://github.com/Nimauric/Mapler

## Examples

```
./local_pipeline.sh config/config_test.yaml > mylog.txt
```

## Evaluation signals

- krona.html and krona.html.files exist for every configured fraction under outputs/<sample_name>/<assembler>/kraken2/<fraction_name>/
- On the Deadwood HiFi dataset (doi:10.57745/2SA8AB), phylum-level bacteria assignment reads ~56% in the unmapped Krona plot vs ~76% in the mapped one
- Legionella leaves appear only in the unmapped fraction's Krona plot, never in the mapped fraction
- Ring-level counts are consistent with the underlying kraken2.tsv classifications for each fraction
- Recorded per-fraction percentages match the reported values within expected stochastic/deviation tolerance when compiled into the fraction-comparison summary table

## Limitations

- Interpretation is bounded by the Kraken 2 database chosen ('to be chosen depending on targeted taxa and available disk space'); divergent or novel taxa may be left unclassified or coarsely assigned at high ranks.
- Krona percentages are descriptive read-level assignments: taxa seen only in unmapped reads may reflect low abundance rather than true absence, consistent with unmapped Deadwood reads being dominated by abundance-1 k-mers.
- Cross-fraction interpretation is only meaningful when Mapler produced both mapped and unmapped FASTQs; a single fraction yields no baseline for comparison.

## Evidence

- [methods] Taxonomic assignment of reads : done with kraken2: "Taxonomic assignment of reads : done with kraken2"
- [methods] Typically, low abundance reads are more libely to be more abundant in the unmapped portions: "Typically, low abundance reads are more libely to be more abundant in the unmapped portions"
- [readme] separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment: "separately analyzes mapped and unmapped reads via their k-mer frequency, read quality, and taxonomic assignment"
- [readme] several Kraken 2 databases are available [here](https://benlangmead.github.io/aws-indexes/k2), to be chosen depending on targeted taxa and available disk space.: "several Kraken 2 databases are available [here](https://benlangmead.github.io/aws-indexes/k2), to be chosen depending on targeted taxa and available disk space."
- [other] Run Kraken 2 on each fraction and render interactive Krona plots, quantifying the proportion of bacteria assignable at phylum level in unassembled (56%) vs assembled (76%) reads: [evidence span withheld — anchor status: not_found]

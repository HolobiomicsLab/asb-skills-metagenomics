---
name: metagenomic-read-taxonomic-classification
description: Use when you have FASTQ read fractions (mapped/unmapped from read-contig
  alignment) from a HiFi metagenome assembly and need to know whether low-representation
  taxa are missing from the assembly.
when_to_use_negative:
- The input is genomic bins needing taxonomic assignment — Mapler handles that separately
  with GTDB-Tk, not Kraken 2.
- Reads are short reads or non-HiFi data outside Mapler's HiFi long-read assembly-evaluation
  scope.
- You only need a single taxonomic label per sample with no mapped/unmapped comparison
  — standalone Kraken 2 suffices without the Mapler fraction split.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kraken2
  role: Taxonomic assignment of reads, run per mapped/unmapped read fraction
  repo: https://github.com/DerrickWood/kraken2
- name: Krona
  role: Renders interactive Krona plots from Kraken 2 output per read fraction
- name: Mapler
  role: Snakemake pipeline that produces the mapped/unmapped read fractions passed
    to Kraken 2
  repo: https://github.com/Nimauric/Mapler
- name: GTDB-Tk
  role: Parallel taxonomic assignment of bins (not reads) within the same pipeline
  repo: https://github.com/Ecogenomics/GTDBTk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:cfda74eb84468b450014cf26c3731f484dcfe81f4c4929f6bb6c8b1aa9935946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken2
  tools:
  - Bash
  - Conda
  - FastQC
  - GTDB-Tk
  - KAT
  - Kraken2
  - Krona
  - Mapler
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metagenomic-read-taxonomic-classification

## Summary

Classifies HiFi metagenomic reads taxonomically with Kraken 2, applied separately to mapped and unmapped read fractions produced by the Mapler assembly-quality pipeline, and renders interactive Krona plots to quantify which taxa the assembly fails to capture.

## When to use

Use when you have FASTQ read fractions (mapped/unmapped from read-contig alignment) from a HiFi metagenome assembly and need to know whether low-representation taxa are missing from the assembly.

## When NOT to use

- The input is genomic bins needing taxonomic assignment — Mapler handles that separately with GTDB-Tk, not Kraken 2.
- Reads are short reads or non-HiFi data outside Mapler's HiFi long-read assembly-evaluation scope.
- You only need a single taxonomic label per sample with no mapped/unmapped comparison — standalone Kraken 2 suffices without the Mapler fraction split.

## Inputs

- FASTQ read fractions (e.g. mapped_reads.fastq, unmapped_reads.fastq) produced by Mapler's read-contig mapping step
- Kraken 2 taxonomic database (path set in configfile kraken2db field)
- Mapler configfile with kraken2: true in the controls section

## Outputs

- kraken2.tsv per read fraction under outputs/<sample_name>/<assembler>/kraken2/<fraction_name>/
- krona.html interactive taxonomic visualization per fraction
- fraction-comparison statistics (e.g. % bacteria at phylum level per fraction)

## How to apply

First separate reads into mapped and unmapped fractions by aligning reads on contigs with minimap2 inside the Mapler Snakemake pipeline, which writes them as <fraction_name>_reads.fastq under outputs/<sample_name>/<assembler>/. Enable kraken2 (and optionally fastqc and kat) in the configfile and provide a Kraken 2 database path via the kraken2db field; databases are selected depending on targeted taxa and disk space. Run Kraken 2 on each fraction, then render interactive Krona plots from the kraken2.tsv output to visualize composition. Finally, compare fractions quantitatively — e.g., proportion of bacteria assignable at phylum level (56% in unmapped vs 76% in mapped reads for the Deadwood sample) and taxa detected only in the unmapped fraction (e.g., Legionella) — as evidence of assembly-relevant taxonomic bias.

## Related tools

- **Kraken2** (Taxonomic assignment of reads, run per mapped/unmapped read fraction) — https://github.com/DerrickWood/kraken2
- **Krona** (Renders interactive Krona plots from Kraken 2 output per read fraction)
- **Mapler** (Snakemake pipeline that produces the mapped/unmapped read fractions passed to Kraken 2) — https://github.com/Nimauric/Mapler
- **GTDB-Tk** (Parallel taxonomic assignment of bins (not reads) within the same pipeline) — https://github.com/Ecogenomics/GTDBTk

## Examples

```
./local_pipeline.sh config/deadwood.yaml > mylog.txt
```

## Evaluation signals

- Unmapped fraction shows a distinct taxonomic profile from the mapped fraction, e.g. ~56% bacteria assignable at phylum level vs ~76% in mapped Deadwood reads.
- Taxa restricted to the unmapped fraction (e.g. Legionella species) are detected in krona.html only for unmapped reads.
- Both krona.html files render without error and contain hierarchical taxonomy derived from kraken2.tsv.
- Per-fraction kraken2.tsv exists at outputs/<sample_name>/<assembler>/kraken2/<fraction_name>/ for every fraction listed in the configfile.
- Reproduced proportions match reported values within expected stochastic/deviation tolerance.

## Limitations

- Requires an external Kraken 2 database whose coverage depends on targeted taxa and available disk space; without kraken2db set, the analysis is skipped.
- In the Deadwood case, fewer unmapped reads were classifiable at phylum level (56% vs 76%), so interpretation of low-prevalence taxa must account for reduced classification power.
- Only part of the analysis runs on the bundled test dataset, so a full-taxonomy test requires a full run with real data.

## Evidence

- [methods] Taxonomic assignment of reads : done with kraken2: "Taxonomic assignment of reads : done with kraken2"
- [abstract] Mapler separately analyzes mapped and unmapped reads via k-mer frequency, read quality, and taxonomic assignment: "Mapler analyzes the alignment of reads on contigs. It does this by calculating the ratio of mapped reads and bases, and separately analyzes mapped and unmapped reads via their k-mer frequency, read"
- [readme] Kraken 2 database path must be indicated in the configfile (kraken2db and gtdbtk_database): "To run taxonomic assignment, both Kraken 2 (reads) and GTDB-Tk (bins) require external databases. If used, their path must be indicated in the configfile (kraken2db and gtdbtk_database)."
- [readme] Several Kraken 2 databases are available, chosen depending on targeted taxa and disk space: "several Kraken 2 databases are available [here](https://benlangmead.github.io/aws-indexes/k2), to be chosen depending on targeted taxa and available disk space."

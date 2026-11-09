---
name: per-lineage-viral-read-fraction-quantification
description: Use when a viral taxonomic classification run (e.g., VirMAP on a mock
  community such as the NetoVir control SRR3458562) must be converted into per-lineage
  read fractions and benchmarked against published relative abundance proportions.
when_to_use_negative:
- The goal is absolute viral load quantification rather than relative per-lineage
  fractions of total reads.
- The sample is a real environmental/clinical virome with no published reference proportions,
  leaving no external benchmark for the R² comparison step.
- Input is raw unassembled reads with no VirMAP (or equivalent lineage-level) classification
  yet produced.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3697
tools:
- name: VirMAP
  role: Taxonomically classifies reads into viral lineages whose per-lineage read
    counts are aggregated into fractions
  repo: https://github.com/cmmr/VirMAP
- name: fasterq-dump
  role: Retrieves the paired-end FASTQ reads from SRA to supply the total-read denominator
- name: Statistics::Basic
  role: Computes the coefficient of determination (R²) between VirMAP per-lineage
    proportions and published Conceição-Neto proportions
- name: makeVirmapDb.pl
  role: Builds the Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn and gbBlastx.dmnd
    reference database from Genbank viral and phage divisions used for classification
  repo: https://raw.githubusercontent.com/cmmr/virmap/master/virmapInstall.pl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:731397c4ec7be0d28aff15bdfeaf6265e72252501556256bd61107c867325139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - Statistics::Basic
  - VirMAP
  - fasterq-dump
  - makeVirmapDb.pl
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# per-lineage-viral-read-fraction-quantification

## Summary

Aggregates VirMAP-classified reads by viral lineage to compute the fraction of total reads assigned to each lineage in a mock virome sample, then correlates those proportions against independently published per-lineage values (e.g., Conceição-Neto proportions) to quantify concordance.

## When to use

Use when a viral taxonomic classification run (e.g., VirMAP on a mock community such as the NetoVir control SRR3458562) must be converted into per-lineage read fractions and benchmarked against published relative abundance proportions.

## When NOT to use

- The goal is absolute viral load quantification rather than relative per-lineage fractions of total reads.
- The sample is a real environmental/clinical virome with no published reference proportions, leaving no external benchmark for the R² comparison step.
- Input is raw unassembled reads with no VirMAP (or equivalent lineage-level) classification yet produced.

## Inputs

- VirMAP taxonomic classification output for the sample (per-lineage classified read assignments)
- Paired-end FASTQ reads retrieved from SRA (e.g., SRR3458562_1.fastq, SRR3458562_2.fastq) for total read counts
- Published per-lineage reference proportions (Conceição-Neto proportions for the NetoVir mock community)

## Outputs

- results_table.csv with per-lineage viral read fractions and the R² value versus published proportions

## How to apply

After Virmap.pl finishes classifying a sample (e.g., SRR3458562 processed with --useMegahit and --useBbnorm against a database built from the Genbank viral and phage divisions with makeVirmapDb.pl), aggregate the classified reads per viral lineage from VirMAP's taxonomic output. Count total input reads from the fasterq-dump FASTQ pair to form the denominator, and compute each lineage's fraction of total reads; the mock community should recover all nine constituent viruses as distinct lineages (10 lineages in the NetoVir case including all mock members). Compute the coefficient of determination between the per-lineage fractions and the published Conceição-Neto per-lineage proportions using Perl's Statistics::Basic, then write per-lineage fractions and the R² value to results_table.csv. Validity is judged by the overall classified viral fraction approximating the reported 37.04% (6,707,800 of ~18.1 million reads) and the correlation reaching the reported R² = 0.9972.

## Related tools

- **VirMAP** (Taxonomically classifies reads into viral lineages whose per-lineage read counts are aggregated into fractions) — https://github.com/cmmr/VirMAP
- **fasterq-dump** (Retrieves the paired-end FASTQ reads from SRA to supply the total-read denominator)
- **Statistics::Basic** (Computes the coefficient of determination (R²) between VirMAP per-lineage proportions and published Conceição-Neto proportions)
- **makeVirmapDb.pl** (Builds the Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn and gbBlastx.dmnd reference database from Genbank viral and phage divisions used for classification) — https://raw.githubusercontent.com/cmmr/virmap/master/virmapInstall.pl

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR3458562 && Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --readF /home/$USER/VirmapTest/SRR3458562_1.fastq
```

## Evaluation signals

- Overall classified viral fraction approximates the reported 37.04% (6,707,800 reads) for SRR3458562.
- The number of distinct viral lineages recovered matches the expectation (10 lineages spanning all nine mock-community viruses).
- Coefficient of determination against Conceição-Neto per-lineage proportions reaches R² = 0.9972.
- Sum of per-lineage classified read counts equals the total number of reads VirMAP classified as viral.
- results_table.csv contains one fraction per lineage plus the R² value with denominators traceable to the raw FASTQ read counts.

## Limitations

- Fraction quantification depends on the Genbank gbvrl/gbphage database snapshot; the article does not state a database version or download date, so fractions are not reproducible across snapshots.
- VirMAP's sensitivity/specificity tradeoff parameters (Supplementary Data 14) are modifiable, so lineage fractions can shift with parameter choices.
- Correlation (R²) measures concordance of relative proportions, not absolute accuracy of viral load per lineage.

## Evidence

- [discussion] VirMAP classified 6,707,800 reads (37.04%) of the NetoVir control sample as being of viral origin, spanning 10 distinct viral lineages that included all nine viruses of the mock community.: [evidence span withheld — anchor status: not_found]
- [discussion] Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome.: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome"
- [methods] `Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [methods] fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293: "`fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293`"

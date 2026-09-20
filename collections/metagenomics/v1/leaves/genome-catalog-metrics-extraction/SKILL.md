---
name: genome-catalog-metrics-extraction
description: Use when a metagenomic binning study reports genome yields (e.g. complete
  vs draft counts) and you need to reproduce or extract the per-category, per-phylum
  breakdown from abstract text, tables, or supplementary tables.
when_to_use_negative:
- Input is raw metagenomic sequence data requiring assembly and binning rather than
  already-reported yield counts
- The goal is de novo genome binning (e.g. with abawaca using differential coverage
  and DNA signature) rather than extracting reported catalog metrics
- The study does not report both complete and draft genome categories or per-phylum
  distributions, so no breakdown table can be constructed
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: abawaca
  role: binning program that produced the genome bins underlying the 8 complete and
    789 draft genomes being counted
  repo: https://github.com/CK7/abawaca
- name: fix_assembly_errors (re_assemble_errors.py / ra2.py)
  role: scaffold curation scripts used to manually curate the 8 complete genomes counted
    in the yield table
  repo: https://github.com/christophertbrown/fix_assembly_errors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:7cf2fd565496d46da9e7d779957050dc0f451305129cbc615930d20ef49c574b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CK7/abawaca
  tools:
  - abawaca
  - fix_assembly_errors (re_assemble_errors.py / ra2.py)
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genome-catalog-metrics-extraction

## Summary

Extract and tabulate genome reconstruction yield metrics from a metagenomic study — the number of complete (manually curated) versus draft-quality genomes, their phylum distribution, and characteristic genome size notes — into a structured CSV for reuse and validation.

## When to use

Use when a metagenomic binning study reports genome yields (e.g. complete vs draft counts) and you need to reproduce or extract the per-category, per-phylum breakdown from abstract text, tables, or supplementary tables.

## When NOT to use

- Input is raw metagenomic sequence data requiring assembly and binning rather than already-reported yield counts
- The goal is de novo genome binning (e.g. with abawaca using differential coverage and DNA signature) rather than extracting reported catalog metrics
- The study does not report both complete and draft genome categories or per-phylum distributions, so no breakdown table can be constructed

## Inputs

- Article abstract and Table 1 (reported genome yields: 8 complete, 789 draft, >35 phyla)
- Supplementary Table 3 (phylum assignments of the 8 manually curated complete genomes)
- Article abstract statements on genome size (small CPR genomes, <1 Mb for complete genomes)

## Outputs

- genome_yields_phylum_distribution.csv with columns: genome category (complete/draft), phylum, count, genome size note

## How to apply

Parse the article abstract and Table 1 to extract reported genome yields — in this study, 8 complete and 789 draft genomes reconstructed from bacteria representing >35 phyla of the candidate phyla radiation (CPR). Extract from Table 1 and Supplementary Table 3 the phylum assignments of the manually curated complete genomes: 3 Microgenomates, 2 Parcubacteria, 1 Kazan, 1 Berkelbacteria, and 1 Saccharibacteria. Record the reported small genome size characteristic (<1 Mb) for the complete genomes, consistent with the finding that all CPR genomes are small. Assemble these values into a CSV with columns for genome category (complete/draft), phylum, count, and genome size note, saved as genome_yields_phylum_distribution.csv. Validate that the CSV contains the yield rows with counts 8 (complete) and 789 (draft), and that the phylum-distribution rows (3+2+1+1+1) sum to the 8 complete genomes.

## Related tools

- **abawaca** (binning program that produced the genome bins underlying the 8 complete and 789 draft genomes being counted) — https://github.com/CK7/abawaca
- **fix_assembly_errors (re_assemble_errors.py / ra2.py)** (scaffold curation scripts used to manually curate the 8 complete genomes counted in the yield table) — https://github.com/christophertbrown/fix_assembly_errors

## Examples

```
python parse_tables.py --abstract abstract.txt --table1 table1.csv --supp3 supp_table3.csv --out genome_yields_phylum_distribution.csv
```

## Evaluation signals

- CSV contains a complete-genome row with count 8 and a draft-genome row with count 789
- Phylum-distribution rows (3 Microgenomates + 2 Parcubacteria + 1 Kazan + 1 Berkelbacteria + 1 Saccharibacteria) sum to 8 complete genomes
- Genome size note column records the <1 Mb characteristic for complete CPR genomes
- Every count row carries provenance to its source (abstract/Table 1 or Supplementary Table 3)

## Limitations

- Counts are only as reliable as the underlying binning quality criteria (draft bins required ≥50% SCGs with fewer than 1.125 copies), so bins failing curation thresholds are excluded from the tally
- Phylum assignments for complete genomes depend on curated Supplementary Table 3 values; discrepancies between main text and supplementary tables must be resolved before reporting
- The draft/complete distinction reflects manual curation effort, not an objective quality standard, and may not be comparable across studies

## Evidence

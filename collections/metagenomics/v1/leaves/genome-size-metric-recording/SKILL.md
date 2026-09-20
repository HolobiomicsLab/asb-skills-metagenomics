---
name: genome-size-metric-recording
description: Use when reproducing or reporting genome yield tables (complete vs draft
  bin counts) and per-phylum distributions of curated genomes from metagenomic binning
  results.
when_to_use_negative:
- Input is already a completed genome-count or phylum-distribution table with validated
  totals
- The task requires re-binning scaffolds or recomputing SCG copy numbers rather than
  recording reported yield metrics
- Genome counts come from a study with different completion criteria (e.g. no scaffold
  curation step) so the 8/789 framing does not apply
tools:
- name: fix_assembly_errors
  role: Script (re_assemble_errors.py / ra2.py) used to curate scaffolds of the manually
    curated complete genomes whose counts are recorded
  repo: https://github.com/christophertbrown/fix_assembly_errors
- name: abawaca
  role: Binning algorithm that produced the genome bins underlying the draft/complete
    yield counts
  repo: https://github.com/CK7/abawaca
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:54be1c66b4800462ef444f4515829a0ed84631386f3868dbd797b3a2f0611fc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - abawaca
  - fix_assembly_errors
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# genome-size-metric-recording

## Summary

Extracts and tabulates genome reconstruction yields (manually curated complete vs draft bins) and their phylum-level distribution from a metagenomic binning study, e.g. the CPR study's 8 complete and 789 draft genomes from >35 phyla, into a structured CSV for downstream validation.

## When to use

Use when reproducing or reporting genome yield tables (complete vs draft bin counts) and per-phylum distributions of curated genomes from metagenomic binning results.

## When NOT to use

- Input is already a completed genome-count or phylum-distribution table with validated totals
- The task requires re-binning scaffolds or recomputing SCG copy numbers rather than recording reported yield metrics
- Genome counts come from a study with different completion criteria (e.g. no scaffold curation step) so the 8/789 framing does not apply

## Inputs

- Article abstract and Table 1 text reporting genome yields (8 complete, 789 draft, >35 phyla)
- Supplementary Table 3 phylum assignments for the manually curated complete genomes
- Binned genome list (draft bins meeting the ≥50% SCG, <1.125 copy-number criterion; curated complete genomes)

## Outputs

- genome_yields_phylum_distribution.csv with columns: genome category (complete/draft), phylum, count, genome size note

## How to apply

Parse the abstract and Table 1 to pull the reported yields — here 8 complete and 789 draft genomes from bacteria representing >35 phyla — then cross-reference Table 1 and Supplementary Table 3 for the phylum assignments of the manually curated complete genomes: 3 Microgenomates, 2 Parcubacteria, 1 Kazan, 1 Berkelbacteria, 1 Saccharibacteria. Record the reported small genome size characteristic (<1 Mb) alongside the complete-genome rows. Assemble the values into a CSV with columns for genome category (complete/draft), phylum, count, and genome size note, saved as genome_yields_phylum_distribution.csv. Validate that the complete-genome phylum rows sum to 8 (3+2+1+1+1) and the draft row reads 789; binning counts ultimately derive from bins meeting the draft criterion of ≥50% single-copy genes with fewer than 1.125 copies, and complete genomes were scaffold-curated (e.g. with re_assemble_errors.py/ra2.py) before being counted.

## Related tools

- **fix_assembly_errors** (Script (re_assemble_errors.py / ra2.py) used to curate scaffolds of the manually curated complete genomes whose counts are recorded) — https://github.com/christophertbrown/fix_assembly_errors
- **abawaca** (Binning algorithm that produced the genome bins underlying the draft/complete yield counts) — https://github.com/CK7/abawaca

## Examples

```
python -c "import csv; w=csv.writer(open('genome_yields_phylum_distribution.csv','w')); w.writerow(['category','phylum','count','genome_size_note']); w.writerow(['complete','Microgenomates',3,'<1 Mb'])"
```

## Evaluation signals

- CSV contains a complete-genome row with count 8 and a draft row with count 789
- Phylum-distribution rows for complete genomes sum to 8 (3 Microgenomates + 2 Parcubacteria + 1 Kazan + 1 Berkelbacteria + 1 Saccharibacteria)
- Complete-genome rows carry the <1 Mb genome size note
- Header matches the specified schema: genome category (complete/draft), phylum, count, genome size note
- Phyla count reference (>35 phyla) is recorded as context for the draft yield

## Limitations

- The <1 Mb size note applies to the CPR complete genomes; it cannot be generalized to the 789 draft bins without checking Supplementary Table 3
- Phylum distribution covers only the 8 manually curated complete genomes, not the full draft set
- Draft bin counts depend on the study's binning criteria (≥50% SCGs with fewer than 1.125 copies); reusing the recorded number in a differently binned dataset is invalid

## Evidence

- [readme] Complete-genome scaffold curation: [evidence span withheld — anchor status: not_found]

---
name: cross-study-classification-agreement-correlation
description: Use when a mock-community or benchmark sample has both a newly computed
  per-lineage classification profile and published reference proportions from another
  study, and you need a single agreement statistic (e.g., R²) to validate the new
  method's quantitative accuracy.
when_to_use_negative:
- The sample is not a characterized mock community with published independent per-lineage
  proportions — there is no reference vector to correlate against, so R² cannot be
  computed meaningfully
- 'You need absolute abundance accuracy rather than cross-study agreement: R² captures
  rank/proportion agreement between studies and can be high even when total classified
  fraction differs substantially between pipelines'
- The new method's per-lineage profile has a different lineage granularity than the
  published proportions (e.g., species-level vs. the paper's 10 distinct viral lineages)
  without a documented mapping between the two taxonomic levels
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: VirMAP
  role: Produces the per-lineage viral read classification fractions being validated
    against published proportions
  repo: https://github.com/cmmr/VirMAP
- name: Perl
  role: Language in which VirMAP and the Statistics::Basic-based R² computation are
    implemented
  repo: https://github.com/cmmr/VirMAP
- name: fasterq-dump
  role: Retrieves the paired-end FASTQ reads for SRR3458562 from the SRA
- name: DIAMOND
  role: Provides translated protein alignments (virDmnd.dmnd, gbBlastx.dmnd) underlying
    VirMAP's per-lineage classification
  repo: https://github.com/bbuchfink/diamond
- name: BBMap
  role: Provides nucleotide alignment component (virBbmap) and bbnorm digital normalization
    used during classification
- name: MEGAHIT
  role: De-novo assembler invoked via --useMegahit in the classification run whose
    output is correlated
  repo: https://github.com/voutcn/megahit
- name: Statistics::Basic
  role: Perl module used to compute the coefficient of determination between VirMAP
    and published per-lineage proportions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:aea32a63125c30be12bc72642cd2d5ccd26bd7dbd3208e0a2622eab168de04db
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-study-classification-agreement-correlation

## Summary

Quantify agreement between a tool's per-lineage taxonomic classification fractions on a shared mock-community sample and previously published proportions from an independent study, using the coefficient of determination (R²) computed with Statistics::Basic. Applied in VirMAP validation by correlating per-lineage viral read fractions on the NetoVir mock virome (SRR3458562) against Conceição-Neto's published proportions.

## When to use

Use when a mock-community or benchmark sample has both a newly computed per-lineage classification profile and published reference proportions from another study, and you need a single agreement statistic (e.g., R²) to validate the new method's quantitative accuracy.

## When NOT to use

- The sample is not a characterized mock community with published independent per-lineage proportions — there is no reference vector to correlate against, so R² cannot be computed meaningfully
- You need absolute abundance accuracy rather than cross-study agreement: R² captures rank/proportion agreement between studies and can be high even when total classified fraction differs substantially between pipelines
- The new method's per-lineage profile has a different lineage granularity than the published proportions (e.g., species-level vs. the paper's 10 distinct viral lineages) without a documented mapping between the two taxonomic levels

## Inputs

- Paired-end FASTQ reads for the benchmark mock-community sample (e.g., SRR3458562 from SRA, BioProject PRJNA319556)
- VirMAP classification output: per-lineage read counts / fractions from Virmap.pl run with --taxaJson, --virDmnd, --virBbmap, --gbBlastn, --gbBlastx
- Published per-lineage viral read proportions from the reference study (Conceição-Neto NetoVir mock community proportions)

## Outputs

- Per-lineage viral read fraction table (results_table.csv) for the 10 lineages spanning the nine mock-community viruses
- Coefficient of determination (R²) between computed and published per-lineage proportions
- Overall classified viral fraction (fraction of total reads classified as viral)

## How to apply

First run the classifier (here Virmap.pl with --useMegahit --useBbnorm against a database built from the Genbank viral and phage divisions gbvrl/gbphage) on the shared benchmark sample, then aggregate classified reads per viral lineage and compute each lineage's fraction of total reads classified as viral. Retrieve the published per-lineage proportions from the independent study (Conceição-Neto proportions for the NetoVir mock community, BioProject PRJNA319556, sample SRR3458562). Compute the coefficient of determination between the two per-lineage proportion vectors using the Perl Statistics::Basic module, and report both the overall classified viral fraction (expected ≈37.04% of 6,707,800 reads across 10 lineages) and the R² (reported value 0.9972) alongside the per-lineage fractions in a results table (results_table.csv). The R² is the validation metric: it captures how faithfully the new pipeline reproduces the independent study's lineage-level abundance ordering, not just total viral detection.

## Related tools

- **VirMAP** (Produces the per-lineage viral read classification fractions being validated against published proportions) — https://github.com/cmmr/VirMAP
- **Perl** (Language in which VirMAP and the Statistics::Basic-based R² computation are implemented) — https://github.com/cmmr/VirMAP
- **fasterq-dump** (Retrieves the paired-end FASTQ reads for SRR3458562 from the SRA)
- **DIAMOND** (Provides translated protein alignments (virDmnd.dmnd, gbBlastx.dmnd) underlying VirMAP's per-lineage classification) — https://github.com/bbuchfink/diamond
- **BBMap** (Provides nucleotide alignment component (virBbmap) and bbnorm digital normalization used during classification)
- **MEGAHIT** (De-novo assembler invoked via --useMegahit in the classification run whose output is correlated) — https://github.com/voutcn/megahit
- **Statistics::Basic** (Perl module used to compute the coefficient of determination between VirMAP and published per-lineage proportions)

## Examples

```
perl -MStatistics::Basic -e 'my $r2 = Statistics::Basic::correlation(\@virmap_fractions, \@netovir_proportions)->square; print "R2=$r2\n";'
```

## Evaluation signals

- Overall classified viral fraction approximates the reported 37.04% (6,707,800 of total reads) for SRR3458562
- Number of viral lineages with classified reads equals the reported 10, covering all nine viruses of the mock community
- Coefficient of determination against Conceição-Neto per-lineage proportions reaches the reported R² = 0.9972 (values well below ~0.99 indicate a pipeline or aggregation discrepancy)
- results_table.csv contains one fraction per lineage plus the R² value, with fractions summing consistently against the overall viral fraction
- Per-lineage fractions are computed over the same denominator (total reads) in both the computed and published profiles before correlating

## Limitations

- The correlation validates agreement with one published study's proportions (Conceição-Neto) on one sample; it does not by itself establish absolute abundance accuracy or transferability to clinical/environmental samples with high background noise
- VirMAP classification depends on database coverage: putative viral genomes and protein pseudo-scaffolds come from clustered alignments to the Genbank viral and phage divisions (gbvrl and gbphage), and the paper notes the minimum SSD and RAM requirements expand as Genbank expands, with no Genbank snapshot/version specified
- The R² is computed over only 10 lineages, so it is sensitive to small per-lineage proportion differences and should be reported together with the per-lineage fractions

## Evidence

- [results] VirMAP classified 6,707,800 reads (37.04%) of the NetoVir control sample as being of viral origin, spanning 10 distinct viral lineages: [evidence span withheld — anchor status: not_found]
- [results] correlation against Conceição-Neto proportions R² = 0.9972: [evidence span withheld — anchor status: not_found]
- [intro] clustered nucleotide and translated alignments to the Genbank viral and phage divisions: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] NetoVir mock community BioProject PRJNA319556: "NetoVir mock community, BioProject: PRJNA319556"
- [methods] Virmap.pl invocation pattern with useMegahit and useBbnorm: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [discussion] VirMAP is open-source and implemented in Perl: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
